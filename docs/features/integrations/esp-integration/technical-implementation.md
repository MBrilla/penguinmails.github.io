---
title: "ESP Technical Implementation"
description: "API integration, failover configuration, and cost optimization"
last_modified_date: "2025-01-15"
level: "3"
persona: "Developers, DevOps"
---

# ESP Technical Implementation

API integration examples, failover configuration, and cost optimization strategies.

## API Integration

Send emails via ESP programmatically using the PenguinMails API.

### Postmark API Example

```javascript
POST /api/v1/emails/send
Authorization: Bearer {api_key}

{
  "provider": "postmark",
  "from": "noreply@yourdomain.com",
  "to": "user@example.com",
  "subject": "Welcome to PenguinMails",
  "html_body": "<h1>Welcome!</h1><p>Thanks for signing up.</p>",
  "track_opens": true,
  "track_links": true
}
```

### Mailgun API Example

```javascript
POST /api/v1/emails/send
Authorization: Bearer {api_key}

{
  "provider": "mailgun",
  "from": "newsletter@yourdomain.com",
  "to": ["user1@example.com", "user2@example.com"],
  "subject": "Monthly Newsletter",
  "html_body": "<html>...</html>",
  "tags": ["newsletter", "november"],
  "campaign_id": "camp_abc123"
}
```

## Failover Configuration

Configure automatic failover if primary ESP fails.

### Failover Strategy

```yaml
Failover Strategy:

Primary: Postmark
Fallback: Mailgun
Final Fallback: Built-in SMTP

Retry Logic:
  - Primary fails → Wait 30s → Retry
  - Primary fails again → Switch to Fallback
  - Fallback fails → Switch to Final Fallback
  - All fail → Queue for later retry
```

### Configuration Steps

Navigate to Settings → Integrations → ESP Failover:

```
Enable Failover: ☑ Yes

Priority Order:
1. Postmark (transactional)
2. Mailgun (marketing)
3. Built-in SMTP (final fallback)

Retry Attempts: 3
Retry Delay: 30 seconds

[Save Failover Config]
```

## Cost Optimization

Smart ESP routing minimizes costs while maintaining delivery quality.

### Optimization Strategy

- **Free tier first** - Use Postmark's 100 free emails/month for initial sends
- **Volume tiers** - Route high-volume through cheapest provider
- **Priority routing** - Critical emails through premium ESP
- **Batch processing** - Combine sends to reduce API calls

### Cost Comparison

```
Scenario: 100,000 emails/month

Built-in SMTP: $0 (infrastructure cost only)
Postmark: $125/month ($1.25/1k)
Mailgun: $135/month (Foundation plan)

Hybrid Approach:
- 5,000 transactional via Postmark: $5
- 95,000 marketing via Mailgun: $55 (Foundation)
Total: $60/month

Savings: $70/month vs all-Postmark
```

### Cost-Effective Routing

Route emails based on type and volume:

| Volume/Month | Transactional | Marketing |
|--------------|---------------|-----------|
| < 1,000 | Postmark (free tier) | Built-in SMTP |
| 1,000-10,000 | Postmark | Mailgun Flex |
| 10,000-50,000 | Postmark | Mailgun Foundation |
| > 50,000 | Postmark | Mailgun Scale |

## Support Resources

### ESP Documentation

- **Postmark Docs** - https://postmarkapp.com/developer
- **Mailgun Docs** - https://documentation.mailgun.com

### PenguinMails Support

- **Help Center** - ESP integration guides
- **Support Email** - integrations@penguinmails.com
- **Community Forum** - ESP best practices

## Related Documentation

### Feature Documentation

- [API Access](/docs/features/integrations/api-access) - REST API and authentication
- [Webhook System](/docs/features/integrations/webhook-system) - Event notifications
- [CRM Integration](/docs/features/integrations/crm-integration/overview) - Salesforce, HubSpot integration

### Email Infrastructure

- [Campaign Management](/docs/features/campaigns/campaign-management/hub) - Creating campaigns
- [Email Warmups](/docs/features/warmup/email-warmups/overview) - Infrastructure warmup
- [Domain Management](/docs/features/domains/sender-authentication) - SPF, DKIM, DMARC

### API Documentation

- [Tenant API](/docs/implementation-technical/api/tenant-api) - ESP configuration endpoints
- [Platform API](/docs/implementation-technical/api/platform-api) - Platform-level endpoints
- [API Reference](/docs/implementation-technical/api/README) - Complete API documentation

---

**Last Updated:** January 2025
**Supported ESPs:** Postmark, Mailgun
**Coming Soon:** SendGrid, Amazon SES
