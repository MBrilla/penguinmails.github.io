---
title: Webhook Event Reference
description: Complete list of webhook event types and payload structures
last_modified_date: 2025-01-15
level: 2
persona: ["developer"]
keywords: [webhook, events, payload, reference]
---

# Webhook Event Reference

Complete reference for all webhook event types and their payload structures.

## Email Events

| Event Type | Description | Key Fields |
|------------|-------------|------------|
| `email.sent` | Email queued for sending | emailId, campaignId, contact |
| `email.delivered` | Email successfully delivered | emailId, deliveredAt, smtpResponse |
| `email.opened` | Email opened by recipient | emailId, openedAt, location, userAgent |
| `email.clicked` | Link clicked in email | emailId, url, clickedAt |
| `email.bounced` | Email bounced | emailId, bounceType, bounceReason |
| `email.spam_reported` | Marked as spam | emailId, reportedAt |
| `email.unsubscribed` | Recipient unsubscribed | emailId, contactId, unsubscribedAt |

### email.opened Payload

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

### email.clicked Payload

```json
{
  "id": "evt_def456",
  "type": "email.clicked",
  "created_at": "2025-11-25T14:31:00Z",
  "data": {
    "email_id": "msg_xyz789",
    "campaign_id": "camp_def456",
    "contact": {
      "id": "cont_ghi012",
      "email": "user@example.com"
    },
    "url": "https://yoursite.com/offer",
    "clicked_at": "2025-11-25T14:31:00Z",
    "link_id": "link_abc123"
  }
}
```

### email.bounced Payload

```json
{
  "id": "evt_ghi789",
  "type": "email.bounced",
  "created_at": "2025-11-25T14:32:00Z",
  "data": {
    "email_id": "msg_xyz789",
    "campaign_id": "camp_def456",
    "contact": {
      "id": "cont_ghi012",
      "email": "user@example.com"
    },
    "bounce_type": "hard",
    "bounce_reason": "mailbox_not_found",
    "smtp_code": "550",
    "bounced_at": "2025-11-25T14:32:00Z"
  }
}
```

## Campaign Events

| Event Type | Description | Key Fields |
|------------|-------------|------------|
| `campaign.launched` | Campaign started sending | campaignId, name, audienceSize |
| `campaign.completed` | Campaign finished sending | campaignId, stats |
| `campaign.paused` | Campaign paused by user | campaignId, pausedBy |

### campaign.launched Payload

```json
{
  "id": "evt_jkl012",
  "type": "campaign.launched",
  "created_at": "2025-11-25T10:00:00Z",
  "data": {
    "campaign_id": "camp_def456",
    "name": "Holiday Sale 2025",
    "audience_size": 15000,
    "scheduled_at": "2025-11-25T10:00:00Z",
    "launched_by": "user_abc123"
  }
}
```

### campaign.completed Payload

```json
{
  "id": "evt_mno345",
  "type": "campaign.completed",
  "created_at": "2025-11-25T12:00:00Z",
  "data": {
    "campaign_id": "camp_def456",
    "name": "Holiday Sale 2025",
    "stats": {
      "sent": 14850,
      "delivered": 14500,
      "opened": 4350,
      "clicked": 870,
      "bounced": 150,
      "unsubscribed": 23
    },
    "completed_at": "2025-11-25T12:00:00Z"
  }
}
```

## Contact Events

| Event Type | Description | Key Fields |
|------------|-------------|------------|
| `contact.created` | New contact added | contactId, email, source |
| `contact.updated` | Contact information updated | contactId, changedFields |
| `contact.unsubscribed` | Contact unsubscribed from all | contactId, reason |

### contact.created Payload

```json
{
  "id": "evt_pqr678",
  "type": "contact.created",
  "created_at": "2025-11-25T09:00:00Z",
  "data": {
    "contact_id": "cont_stu901",
    "email": "newuser@example.com",
    "first_name": "John",
    "last_name": "Doe",
    "source": "api_import",
    "tags": ["newsletter", "prospects"],
    "created_at": "2025-11-25T09:00:00Z"
  }
}
```

## Common Payload Structure

All events follow this structure:

```json
{
  "id": "evt_unique_id",
  "type": "event.type",
  "created_at": "2025-11-25T14:30:00Z",
  "data": {
    // Event-specific data
  }
}
```

## HTTP Headers

All webhook requests include these headers:

| Header | Description |
|--------|-------------|
| `Content-Type` | `application/json` |
| `X-PenguinMails-Signature` | HMAC-SHA256 signature |
| `X-PenguinMails-Event` | Event type |
| `X-PenguinMails-Delivery-ID` | Unique delivery ID |
| `User-Agent` | `PenguinMails-Webhooks/1.0` |

## Related Documentation

- [Webhook Overview](./)
- [Quick Start](./quick-start)
- [Advanced Configuration](./advanced-configuration)
