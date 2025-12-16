---
title: "Email System Queue Integration"
description: "Queue system integration for email processing architecture"
last_modified_date: "2025-01-09"
level: "2"
persona: "Backend Developers, DevOps Engineers"
---

# Email System Queue Integration

## Queue System Architecture

### Email Processing Queue

```typescript
// Queue handler for creating complete email hierarchy
async function handleIncomingEmail(emailData: any) {
  const storage_key = `email_${emailData.messageId}_${Date.now()}`;

  return await db.$transaction(async (tx) => {
    // 1. Create email content first
    const contentObject = await tx.email_content.create({
      data: {
        storage_key,
        tenant_id: emailData.tenant_id,
        content_text: emailData.body.plain || emailData.body.html,
        content_html: emailData.body.html || convertToHtml(emailData.body.plain),
        headers: emailData.headers,
        raw_size_bytes: Buffer.byteLength(emailData.raw),
        created: new Date(),
        retention_days: 2555
      }
    });

    // 2. Create message reference
    const messageRef = await tx.email_messages.create({
      data: {
        storage_key,
        tenant_id: emailData.tenant_id,
        email_account_id: emailData.email_account_id,
        direction: emailData.direction,
        message_type: emailData.type || 'email',
        from_email: emailData.from,
        to_email: emailData.to.join(','),
        subject: emailData.subject,
        status: 'received',
        processed: new Date()
      }
    });

    // 3. Create attachments if present
    const attachmentRefs = [];
    if (emailData.attachments && emailData.attachments.length > 0) {
      for (const attachment of emailData.attachments) {
        const attachmentRef = await tx.attachments.create({
          data: {
            parent_storage_key: storage_key,
            filename: attachment.filename,
            mime_type: attachment.mime_type,
            size_bytes: attachment.size,
            content: attachment.content,
            storage_disposition: attachment.disposition || 'attachment'
          }
        });
        attachmentRefs.push(attachmentRef);
      }
    }

    return {
      success: true,
      storage_key,
      content_object_id: contentObject.storage_key,
      message_ref_id: messageRef.storage_key,
      attachment_count: attachmentRefs.length
    };
  });
}
```

---

## IMAP Integration

### IMAP Worker Implementation

```javascript
// Updated IMAP worker for queue architecture
async function startIncomingWorker() {
  const client = new ImapFlow({
    host: 'mailu.penguinmails.com',
    port: 993,
    secure: true,
    auth: { user: process.env.MAIL_USER, pass: process.env.MAIL_PASS }
  });

  await client.connect();
  const lock = await client.getMailboxLock('INBOX');

  try {
    for await (let msg of client.fetch('1:*', { envelope: true, source: true })) {
      // Add to queue for processing
      await addEmailToQueue({
        direction: 'inbound',
        messageId: msg.envelope.messageId,
        from: msg.envelope.from?.[0]?.address,
        to: msg.envelope.to?.map(r => r.address) || [],
        subject: msg.envelope.subject,
        body: msg.source.toString(),
        raw: msg.source.toString(),
        tenant_id: await getTenantFromEmail(msg.envelope.to?.[0]?.address)
      });
    }
  } finally {
    lock.release();
  }
  await client.logout();
}
```

---

## Queue Architecture Benefits

### Processing Flow

The queue-driven architecture provides:

- **Reliable Processing**: Retry logic with exponential backoff
- **Priority Handling**: High-priority emails processed first
- **Scalability**: Horizontal scaling of queue workers
- **Monitoring**: Real-time queue depth and processing metrics

### Integration Points

| Component | Integration Type | Purpose |
|-----------|------------------|---------|
| IMAP Server | Event-driven | Incoming email ingestion |
| SMTP Server | Webhook | Outbound delivery status |
| Analytics | ETL Pipeline | Performance aggregation |
| Campaigns | Direct Reference | Campaign association |

---

## Related Sections

- [Overview](/docs/implementation-technical/architecture-system/email-system-implementation/overview) - Architecture overview
- [Database Schema](/docs/implementation-technical/architecture-system/email-system-implementation/database-schema) - Schema definitions
- [Performance Monitoring](/docs/implementation-technical/architecture-system/email-system-implementation/performance-monitoring) - Metrics
