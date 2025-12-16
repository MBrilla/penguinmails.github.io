---
title: "ESP Advanced Configuration"
description: "Routing rules, webhook integration, and deliverability monitoring"
last_modified_date: "2025-01-15"
level: "3"
persona: "Operations Teams, Developers"
---

# ESP Advanced Configuration

Configure routing rules, webhook integration, and deliverability monitoring for optimal ESP usage.

## Routing Rules

Send different email types through different providers for optimal delivery and cost efficiency.

### Email Routing Configuration

```yaml
Email Routing Configuration:

Transactional Emails:
  Provider: Postmark
  Types:
    - Password Reset
    - Email Verification
    - Order Confirmation
    - Account Alerts

Marketing Emails:
  Provider: Mailgun
  Types:
    - Newsletters
    - Product Announcements
    - Promotional Campaigns

Cold Outreach:
  Provider: Built-in SMTP
  Types:
    - Sales outreach
    - Lead generation
    - Custom campaigns
```

### Configuration Steps

Navigate to Settings → Integrations → ESP Routing:

```
Default Provider: Built-in SMTP
Override Rules:
  ☑ Transactional → Postmark
  ☑ Marketing → Mailgun
  ☐ All emails → [ESP]

[Save Routing Rules]
```

## Webhook Integration

Receive real-time delivery events from ESPs for immediate status updates.

### Postmark Webhooks

```javascript
// Webhook Endpoint
POST https://api.penguinmails.com/webhooks/postmark

// Event Types
{
  "RecordType": "Delivery", // or "Bounce", "Open", "Click"
  "MessageID": "abc123",
  "Recipient": "user@example.com",
  "DeliveredAt": "2025-11-24T10:30:00Z"
}
```

### Mailgun Webhooks

```javascript
// Webhook Endpoint
POST https://api.penguinmails.com/webhooks/mailgun

// Event Types
{
  "event": "delivered", // or "failed", "opened", "clicked"
  "message": {
    "headers": {
      "message-id": "abc123"
    }
  },
  "recipient": "user@example.com",
  "timestamp": 1700830200
}
```

### Webhook Processing

PenguinMails processes ESP webhooks automatically:

1. ESPs send events to PenguinMails webhook endpoints
2. Events update campaign analytics in real-time
3. Bounce handling triggers suppression list updates
4. Open/click tracking updates engagement metrics

## Deliverability Monitoring

Compare ESP performance to optimize sending strategy.

### Deliverability Dashboard

```
Deliverability Dashboard

Provider      | Delivered | Bounced | Opened | Clicked
──────────────┼───────────┼─────────┼────────┼────────
Postmark      | 99.2%     | 0.8%    | 45.3%  | 12.1%
Mailgun       | 98.5%     | 1.5%    | 38.2%  | 9.7%
Built-in SMTP | 94.1%     | 5.9%    | 31.5%  | 7.3%

Recommendation: Use Postmark for critical emails
```

### Monitoring Best Practices

Monitor these metrics across all ESPs:

- **Delivery rate** - Percentage of emails successfully delivered
- **Bounce rate** - Hard and soft bounces
- **Open rate** - Engagement indicator
- **Click rate** - Content effectiveness
- **Spam complaint rate** - Reputation indicator

## ESP Selection Guide

Choose the right ESP for each use case:

| Email Type | Recommended ESP | Reason |
|------------|-----------------|--------|
| Password reset | Postmark | Critical delivery, fast |
| Welcome email | Postmark | First impression |
| Weekly newsletter | Mailgun | Bulk pricing, A/B testing |
| Promotional campaign | Mailgun | List segmentation |
| Automated drip | Either | Depends on volume |
| Cold outreach | Built-in | Cost-effective, control |

## Deliverability Tips

1. **Warm up gradually** - Start small, increase volume over time
2. **Authenticate everything** - SPF, DKIM, DMARC for all domains
3. **Monitor reputation** - Check ESP dashboards daily
4. **Clean lists** - Remove bounces immediately
5. **Segment sends** - Don't mix transactional + marketing
6. **Test thoroughly** - Send tests before campaigns
7. **Track metrics** - Monitor open, click, bounce rates

---

**Related Documents:**
- [ESP Integration Overview](overview.md)
- [Technical Implementation](technical-implementation.md)
- [Domain Management](/docs/features/domains/sender-authentication)
