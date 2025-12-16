---
title: "Unified Inbox Technical Implementation"
description: "Database schema, message aggregation service, real-time sync, and API endpoints"
level: "3"
status: "AVAILABLE"
roadmap_timeline: "Q1 2026"
priority: "Critical"
keywords: [database-schema, api, websocket, real-time-sync, implementation]
---

# Unified Inbox Technical Implementation

Technical implementation details for the Unified Inbox including database schema, services, and API endpoints.

## Database Schema

### Inbox Threads (Conversations)

```sql
CREATE TABLE inbox_threads (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  workspace_id UUID REFERENCES workspaces(id),

  -- Context
  campaign_id UUID REFERENCES campaigns(id),
  contact_id UUID REFERENCES contacts(id),
  account_id UUID REFERENCES email_accounts(id), -- The sending account

  -- State
  subject VARCHAR(255),
  snippet TEXT,
  status VARCHAR(50) DEFAULT 'open', -- open, archived, snoozed, closed
  category VARCHAR(50), -- interested, not_interested, ooo, etc.
  ai_confidence DECIMAL(3,2),

  -- Assignment
  assigned_to UUID REFERENCES users(id),

  -- Timing
  last_message_at TIMESTAMP,
  snoozed_until TIMESTAMP,

  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_inbox_threads_tenant ON inbox_threads(tenant_id);
CREATE INDEX idx_inbox_threads_status ON inbox_threads(status);
CREATE INDEX idx_inbox_threads_contact ON inbox_threads(contact_id);
```

### Inbox Messages (Individual Emails)

```sql
CREATE TABLE inbox_messages (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  thread_id UUID NOT NULL REFERENCES inbox_threads(id) ON DELETE CASCADE,
  tenant_id UUID NOT NULL REFERENCES tenants(id),

  -- Email Metadata
  provider_message_id VARCHAR(255), -- Gmail/Outlook ID
  direction VARCHAR(20), -- inbound, outbound

  -- Content
  from_email VARCHAR(255) NOT NULL,
  to_email VARCHAR(255) NOT NULL,
  cc_emails TEXT[],
  bcc_emails TEXT[],
  subject VARCHAR(255),
  body_text TEXT,
  body_html TEXT,

  -- Tracking
  is_read BOOLEAN DEFAULT FALSE,
  tracked_open BOOLEAN DEFAULT FALSE,
  tracked_click BOOLEAN DEFAULT FALSE,

  sent_at TIMESTAMP,
  received_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_inbox_messages_thread ON inbox_messages(thread_id);
CREATE INDEX idx_inbox_messages_provider_id ON inbox_messages(provider_message_id);
```

### Supporting Tables

```sql
-- Inbox Tags/Labels
CREATE TABLE inbox_tags (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  thread_id UUID NOT NULL REFERENCES inbox_threads(id) ON DELETE CASCADE,
  tag_name VARCHAR(50) NOT NULL,
  color VARCHAR(7),
  created_by UUID REFERENCES users(id),
  created_at TIMESTAMP DEFAULT NOW()
);

-- Internal Notes
CREATE TABLE inbox_notes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  thread_id UUID NOT NULL REFERENCES inbox_threads(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES users(id),
  content TEXT NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);
```

## Message Aggregation Service

Handles ingestion and synchronization of emails from various providers.

```typescript
interface EmailProvider {
  syncMessages(accountId: string, since: Date): Promise<EmailMessage[]>;
  sendMessage(accountId: string, message: OutboundMessage): Promise<string>;
}

class MessageAggregationService {
  /**
   * Main sync loop for an account
   */
  async syncAccount(accountId: string) {
    const account = await db.emailAccounts.findById(accountId);
    const provider = this.getProvider(account.provider);

    // 1. Fetch new messages
    const lastSync = account.last_sync_at || new Date(0);
    const messages = await provider.syncMessages(accountId, lastSync);

    for (const msg of messages) {
      await this.ingestMessage(account, msg);
    }

    // 2. Update sync status
    await db.emailAccounts.update(accountId, { last_sync_at: new Date() });
  }

  /**
   * Process a single incoming message
   */
  async ingestMessage(account: EmailAccount, msg: EmailMessage) {
    const contact = await db.contacts.findByEmail(msg.from);
    let thread = await this.findThread(msg, contact);

    if (!thread) {
      thread = await db.inboxThreads.create({
        tenant_id: account.tenant_id,
        account_id: account.id,
        contact_id: contact?.id,
        subject: msg.subject,
        status: 'open',
        last_message_at: msg.date
      });
    } else {
      await db.inboxThreads.update(thread.id, {
        status: 'open',
        last_message_at: msg.date
      });
    }

    const savedMsg = await db.inboxMessages.create({
      thread_id: thread.id,
      provider_message_id: msg.id,
      direction: 'inbound',
      body_text: msg.text
    });

    // Trigger AI Analysis (Async)
    await queue.add('analyze-email-intent', { messageId: savedMsg.id });

    // Notify Frontend (WebSocket)
    await pubsub.publish(`tenant:${account.tenant_id}:inbox`, {
      type: 'NEW_MESSAGE',
      threadId: thread.id,
      message: savedMsg
    });
  }
}
```

## Real-Time Sync Engine

Uses WebSockets to push updates to the frontend without polling.

```typescript
// WebSocket Handler
io.on('connection', (socket) => {
  const { tenantId, userId } = socket.handshake.auth;

  // Join tenant room
  socket.join(`tenant:${tenantId}:inbox`);

  // Listen for updates
  socket.on('subscribe_thread', (threadId) => {
    socket.join(`thread:${threadId}`);
  });
});

// Event Publisher
const notifyNewMessage = (tenantId, message) => {
  io.to(`tenant:${tenantId}:inbox`).emit('inbox_update', {
    type: 'MESSAGE_RECEIVED',
    payload: message
  });
};
```

## API Endpoints

### Get Threads (with pagination & filtering)

```typescript
router.get('/api/inbox/threads', async (req, res) => {
  const { status, category, assignedTo, page = 1 } = req.query;

  const threads = await db.inboxThreads.find({
    where: {
      tenant_id: req.user.tenantId,
      status: status || 'open',
      ...(category && { category }),
      ...(assignedTo && { assigned_to: assignedTo })
    },
    include: ['contact', 'last_message'],
    order: { last_message_at: 'DESC' },
    skip: (page - 1) * 20,
    take: 20
  });

  res.json(threads);
});
```

### Get Messages in Thread

```typescript
router.get('/api/inbox/threads/:id/messages', async (req, res) => {
  const messages = await db.inboxMessages.find({
    where: { thread_id: req.params.id },
    order: { sent_at: 'ASC' }
  });
  res.json(messages);
});
```

### Send Reply

```typescript
router.post('/api/inbox/threads/:id/reply', async (req, res) => {
  const { body, attachments } = req.body;
  const thread = await db.inboxThreads.findById(req.params.id);

  const providerId = await emailService.sendReply(thread.account_id, {
    to: thread.contact.email,
    subject: `Re: ${thread.subject}`,
    body
  });

  const message = await db.inboxMessages.create({
    thread_id: thread.id,
    direction: 'outbound',
    body_text: body,
    provider_message_id: providerId
  });

  res.json(message);
});
```

### Update Thread Status/Category

```typescript
router.patch('/api/inbox/threads/:id', async (req, res) => {
  const { status, category, assignedTo } = req.body;
  const updated = await db.inboxThreads.update(req.params.id, {
    status, category, assigned_to: assignedTo
  });
  res.json(updated);
});
```

## Background Jobs

| Job | Purpose |
|-----|---------|
| `sync-worker` | Polls email providers for accounts without webhooks |
| `intent-analyzer` | Processes new messages with LLM for categorization |
| `cleanup-worker` | Archives old threads, deletes spam per retention policies |

## Related Documentation

- [Unified Inbox Overview](/docs/features/inbox/unified-inbox/quick-start) - Getting started guide
- [Configuration](/docs/features/inbox/unified-inbox/configuration) - AI categorization and templates
- [Inbox Rotation](/docs/features/inbox/inbox-rotation) - Related feature

---

**Last Updated:** November 25, 2025
**Status:** Available - Core Feature
**Target Release:** Q1 2026
**Owner:** Inbox Team
