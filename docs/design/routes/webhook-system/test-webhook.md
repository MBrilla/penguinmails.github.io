---
title: "Test Webhook Route"
description: "Route specification for the webhook testing interface with code examples"
last_modified_date: "2025-01-20"
level: 3
persona: Developers
keywords: webhooks, testing, signature verification, code examples
---

# Test Webhook Route

**Route:** `/dashboard/settings/webhooks/:id/test`

**Access:** Admin, Developer

**User Story:** *"As a developer, I want to test my webhook with different event types, so I can verify my integration works correctly."*

## Test Configuration

**Event Type Selector:**
- Dropdown: Select event type to test
  - Options: email.sent, email.delivered, email.opened, email.clicked, email.bounced, etc.

**Data Source:**
- Radio buttons:
  - ○ Use Sample Data (default)
  - ○ Use Real Data from Campaign/Contact

**If "Use Real Data" selected:**
- Campaign: Dropdown (select campaign)
- Contact: Dropdown (select contact)
- Help text: "Test event will use actual data from selected campaign/contact."

## Test Payload Preview

**Payload Display (JSON):**
```json
{
  "id": "evt_test_1234567890",
  "type": "email.opened",
  "created_at": "2025-11-25T14:30:00Z",
  "data": {
    "email_id": "msg_sample",
    "campaign_id": "camp_sample",
    "contact": {
      "id": "cont_sample",
      "email": "test@example.com",
      "first_name": "Test",
      "last_name": "User"
    },
    "opened_at": "2025-11-25T14:30:00Z",
    "user_agent": "Mozilla/5.0...",
    "ip_address": "192.168.1.1"
  }
}
```

**"Edit Payload" Button**: Opens JSON editor for custom payload

## Send Test

**"Send Test Event" Button**: Sends test event to webhook endpoint

**Test Progress:**
```text
Sending test event to https://yourapp.com/webhooks...
⏳ Waiting for response...
```

**Test Result (Success):**
```text
✓ Test successful!

Request:
  POST https://yourapp.com/webhooks/penguinmails
  Headers: X-PenguinMails-Signature: sha256=...

Response:
  Status: 200 OK
  Response Time: 142ms
  Body: "OK"

[View Full Request] [View Full Response]
```

**Test Result (Failure):**
```text
✗ Test failed

Error: Connection timeout after 5000ms

Possible causes:
- Endpoint is unreachable
- Firewall blocking requests
- Server not responding

[Retry Test] [Edit Webhook] [View Troubleshooting Guide]
```

## Code Examples

**Signature Verification Code (Tabbed):**

### Node.js

```javascript
const crypto = require('crypto');

function verifySignature(payload, signature, secret) {
  const hmac = crypto.createHmac('sha256', secret);
  hmac.update(JSON.stringify(payload));
  const digest = `sha256=${hmac.digest('hex')}`;
  return crypto.timingSafeEqual(
    Buffer.from(digest),
    Buffer.from(signature)
  );
}

// Usage
app.post('/webhooks/penguinmails', (req, res) => {
  const signature = req.headers['x-penguinmails-signature'];
  const secret = 'whsec_...'; // Your webhook secret

  if (!verifySignature(req.body, signature, secret)) {
    return res.status(401).send('Invalid signature');
  }

  // Process event
  console.log('Event:', req.body.type);
  res.status(200).send('OK');
});
```

### TypeScript

```typescript
import crypto from 'crypto';
import express, { Request, Response } from 'express';

function verifySignature(payload: string, signature: string, secret: string): boolean {
  const hmac = crypto.createHmac('sha256', secret);
  hmac.update(payload, 'utf8');
  const computed = hmac.digest('hex');
  const expected = `sha256=${computed}`;
  return crypto.timingSafeEqual(
    Buffer.from(expected),
    Buffer.from(signature)
  );
}

// Usage
const app = express();
app.use(express.json());

app.post('/webhooks/penguinmails', (req: Request, res: Response) => {
  const signature = req.headers['x-penguinmails-signature'] as string;
  const secret = 'whsec_...'; // Your webhook secret

  if (!verifySignature(JSON.stringify(req.body), signature, secret)) {
    return res.status(401).send('Invalid signature');
  }

  // Process event
  console.log('Event:', req.body.type);
  return res.status(200).send('OK');
});
```

**Copy Button**: Copy code snippet

## User Journey Context

Initial setup and ongoing testing during development.

## Related Documentation

- [Signature Verification](/docs/features/integrations/webhook-system#signature-verification)
- [Testing Guide](/docs/features/integrations/webhook-system#webhook-debugging)
