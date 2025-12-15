---
title: Advanced Webhook Configuration
description: Filtering, retry logic, signature verification, and debugging
last_modified_date: 2025-01-15
level: 2
persona: ["developer"]
keywords: [webhook, filtering, retry, signature, debugging]
---

{% raw %}

# Advanced Webhook Configuration

Event filtering, retry policies, signature verification, and debugging tools.

## Event Filtering

### Filter by Workspace

```text
Webhook Name: Client A Events Only
Event Filters:
  Workspaces: [Client A]
  Events: [email.opened, email.clicked]
```

### Filter by Campaign

```text
Event Filters:
  Campaign ID: camp_xyz123
  Events: [All]
```

### Custom Filters (JSON)

```json
{
  "event_types": ["email.opened", "email.clicked"],
  "filters": {
    "contact.tags": ["vip", "premium"],
    "campaign.type": "promotional"
  }
}
```

## Retry Logic

### Automatic Retry Schedule

| Retry | Delay |
|-------|-------|
| #1 | 1 minute |
| #2 | 5 minutes |
| #3 | 15 minutes |
| #4 | 1 hour |
| #5 | 24 hours |

After 5 failed attempts:
- Mark webhook as "failing"
- Send email notification to admin
- Pause webhook after 100 consecutive failures

### Failed Deliveries View

```text
Recent Deliveries:
  ✓ email.opened     200 OK   Nov 25, 14:30
  ✓ email.clicked    200 OK   Nov 25, 14:29
  ✗ email.bounced    500 Error Nov 25, 14:28 [Retry #2]

[View Failed Event] [Retry Now] [Disable Webhook]
```

## Signature Verification

### Verification Process

1. PenguinMails generates HMAC-SHA256 signature
2. Signature sent in `X-PenguinMails-Signature` header
3. Your app recomputes signature with shared secret
4. Compare signatures using timing-safe comparison

### TypeScript Implementation

```typescript
import crypto from 'crypto';

interface WebhookHandler {
  verifyWebhook(payload: string, signature: string, secret: string): boolean;
}

class WebhookHandlerImpl implements WebhookHandler {
  private webhookSecret: string;

  constructor(secret: string) {
    this.webhookSecret = secret;
  }

  verifyWebhook(payload: string, signature: string, secret: string): boolean {
    try {
      const computed = crypto
        .createHmac('sha256', secret)
        .update(payload, 'utf8')
        .digest('hex');

      const expected = `sha256=${computed}`;
      return crypto.timingSafeEqual(
        Buffer.from(expected),
        Buffer.from(signature)
      );
    } catch (error) {
      console.error('Webhook signature verification error:', error);
      return false;
    }
  }
}
```

### Full Request Handler

```typescript
async handleWebhook(req: express.Request, res: express.Response): Promise<void> {
  try {
    const signature = req.headers['x-penguinmails-signature'] as string;
    const rawBody = (req as any).rawBody;

    if (!signature) {
      res.status(401).json({ error: 'Missing signature header' });
      return;
    }

    if (!this.verifyWebhook(rawBody, signature, this.webhookSecret)) {
      res.status(401).json({ error: 'Invalid signature' });
      return;
    }

    const event = req.body;

    switch (event.type) {
      case 'email.opened':
        await this.handleEmailOpened(event);
        break;
      case 'email.clicked':
        await this.handleEmailClicked(event);
        break;
      case 'email.bounced':
        await this.handleEmailBounced(event);
        break;
      default:
        console.log(`Unhandled event type: ${event.type}`);
    }

    res.status(200).json({ success: true });
  } catch (error) {
    console.error('Webhook processing error:', error);
    res.status(500).json({ error: 'Internal server error' });
  }
}
```

## Webhook Debugging

### Request Inspector

```text
Last 50 Webhook Deliveries:

[Nov 25, 14:30:15] email.opened
  → POST https://yourapp.com/webhooks
  Status: 200 OK
  Duration: 142ms
  [View Request] [View Response]

Request Headers:
  Content-Type: application/json
  X-PenguinMails-Signature: sha256=abc123...
  X-PenguinMails-Event: email.opened
  User-Agent: PenguinMails-Webhooks/1.0
```

### Test Mode

```text
[Send Test Event]

Choose test event type:
  ○ email.opened
  ○ email.clicked
  ○ email.bounced

Use real data from:
  Campaign: [Select Campaign ▼]
  Contact: [Select Contact ▼]

OR
  Use sample data ☑

[Send Test]
```

## Event Replay

Replay missed events after downtime or webhook creation.

```text
Replay Events

From: Nov 20, 2025  14:00
To:   Nov 25, 2025  16:00

Event Types: [email.opened, email.clicked]

Found: 1,247 matching events

[Replay All Events]

Status: Replaying... (234 / 1,247)
```

**Use Cases:**
- Recover from downtime
- Backfill data after webhook creation
- Re-sync after bug fixes

## Related Documentation

- [Webhook Overview](./)
- [Technical Implementation](./technical-implementation)
- [Event Reference](./event-reference)

{% endraw %}
