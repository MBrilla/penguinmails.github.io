---
title: Webhook Quick Start
description: Create and test your first webhook in PenguinMails
last_modified_date: 2025-01-15
level: 1
persona: ["developer"]
keywords: [webhook, quick-start, tutorial, setup]
---

{% raw %}

# Webhook Quick Start

Create your first webhook and start receiving real-time event notifications.

## Step 1: Navigate to Webhooks

```text
Dashboard → Settings → Integrations → Webhooks → Create Webhook
```

## Step 2: Configure Webhook

```text
Webhook Name: CRM Contact Sync
Endpoint URL: https://yourapp.com/webhooks/penguinmails
Description: Sync email engagement to CRM

Status: ○ Active  ○ Paused
```

## Step 3: Select Events

```text
Email Events:
  ☑ email.sent
  ☑ email.delivered
  ☑ email.opened
  ☑ email.clicked
  ☑ email.bounced
  ☑ email.spam_reported
  ☑ email.unsubscribed

Campaign Events:
  ☐ campaign.launched
  ☐ campaign.completed
  ☐ campaign.paused

Contact Events:
  ☐ contact.created
  ☐ contact.updated
  ☐ contact.unsubscribed
```

## Step 4: Test Webhook

```text
[Send Test Event]

Test payload will be sent to:
https://yourapp.com/webhooks/penguinmails

Sending test event...
✓ Test successful! (Response: 200 OK)
```

## Step 5: Save & Activate

```text
[Save Webhook]

✓ Webhook created and activated
Secret Key: whsec_abc123... [Copy]

Add this to your application for signature verification.
```

## Example Payload

**Email Opened Event:**

```json
{
  "id": "evt_abc123",
  "type": "email.opened",
  "created_at": "2025-11-25T14:30:00Z",
  "data": {
    "email_id": "msg_xyz789",
    "campaign_id": "camp_def456",
    "contact": {
      "id": "cont_ghi012",
      "email": "user@example.com",
      "first_name": "Sarah",
      "last_name": "Johnson"
    },
    "opened_at": "2025-11-25T14:30:00Z",
    "user_agent": "Mozilla/5.0...",
    "ip_address": "192.168.1.1",
    "location": {
      "city": "New York",
      "region": "NY",
      "country": "US"
    }
  }
}
```

## Handling Webhooks

**Node.js Example:**

```javascript
const express = require('express');
const crypto = require('crypto');

app.post('/webhooks/penguinmails', express.json(), (req, res) => {
  // 1. Verify signature
  const signature = req.headers['x-penguinmails-signature'];
  if (!verifySignature(req.body, signature)) {
    return res.status(401).send('Invalid signature');
  }

  // 2. Handle event
  const event = req.body;

  switch (event.type) {
    case 'email.opened':
      console.log(`Email opened by ${event.data.contact.email}`);
      break;
    case 'email.clicked':
      console.log(`Link clicked: ${event.data.url}`);
      break;
  }

  // 3. Respond quickly (< 5 seconds)
  res.status(200).send('OK');
});

function verifySignature(payload, signature) {
  const secret = process.env.WEBHOOK_SECRET;
  const hmac = crypto.createHmac('sha256', secret);
  hmac.update(JSON.stringify(payload));
  const digest = `sha256=${hmac.digest('hex')}`;
  return crypto.timingSafeEqual(Buffer.from(digest), Buffer.from(signature));
}
```

## Related Documentation

- [Webhook Overview](./)
- [Advanced Configuration](./advanced-configuration)
- [Event Reference](./event-reference)

{% endraw %}
