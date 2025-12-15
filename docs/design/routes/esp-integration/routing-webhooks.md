---
title: "ESP Routing and Webhooks Routes"
description: "Route specifications for email routing rules and webhook configuration"
last_modified_date: "2025-11-25"
level: "3"
persona: "Developers, Operations Teams"
---

# ESP Routing and Webhooks Routes

## `/dashboard/settings/integrations/esp/routing` - Email Routing Rules

**User Story**: *"As an admin, I want to route different email types through different ESPs, so I can optimize deliverability and costs."*

### Routing Strategy Overview

**Visual Diagram**:

```text
Email Type → Provider

┌─────────────────────┐
│ Transactional       │ → Postmark
├─────────────────────┤
│ Marketing           │ → Mailgun
├─────────────────────┤
│ Cold Outreach       │ → Built-in SMTP
├─────────────────────┤
│ Default (Fallback)  │ → Built-in SMTP
└─────────────────────┘
```

### Routing Rules Builder

**Rule Cards** (Drag-and-drop reorderable):

Each rule card contains:

#### Rule 1: Transactional Emails

- **Email Type**: Dropdown
  - Options: Transactional, Marketing, Cold Outreach, Newsletter, Promotional, Custom
- **Conditions** (Optional):
  - Add condition: "Subject contains", "Recipient domain is", "Tag equals"
- **Route To**: Dropdown
  - Options: Postmark, Mailgun, Built-in SMTP
- **Priority**: Number (1 = highest)
- **Enabled**: Toggle switch
- **Actions**: Edit, Delete, Duplicate

**"+ Add Routing Rule" Button**: Creates new rule card.

### Email Type Definitions

**Expandable Section** showing what qualifies as each type:

**Transactional**:
- Password resets
- Email verification
- Order confirmations
- Account alerts
- System notifications

**Marketing**:
- Newsletters
- Product announcements
- Promotional campaigns
- Event invitations

**Cold Outreach**:
- Sales prospecting
- Lead generation
- Custom campaigns

### Failover Configuration

**Failover Strategy Section**:

- **Enable Failover**: Toggle switch
- **Retry Attempts**: Number input (1-5, default: 3)
- **Retry Delay**: Seconds (10-300, default: 30)

**Failover Order** (Drag-and-drop):

1. Primary: Postmark
2. Secondary: Mailgun
3. Final: Built-in SMTP

**Help Text**: "If primary ESP fails, automatically retry with secondary after delay."

### Cost Optimization

**Smart Routing Section**:

- ☑ **Use Free Tiers First**: Route through ESPs with remaining free quota
- ☑ **Volume-Based Routing**: Route high-volume sends through cheapest provider
- ☐ **Time-Based Routing**: Route based on time of day (e.g., off-peak hours)

**Estimated Monthly Cost**:

```text
Current Configuration:

- Postmark: 5,000 emails × $0.00125 = $6.25
- Mailgun: 95,000 emails × $0.00058 = $55.00
- Built-in SMTP: 100,000 emails × $0 = $0.00

Total: $61.25/month
```

**Save Button**: Bottom-right, saves routing configuration.

### Technical Integration

- **Rule Engine**: Evaluates rules in priority order
- **Caching**: Routing decisions cached for 5 minutes
- **Monitoring**: Track routing decisions in analytics

### Related Documentation

- [ESP Routing Strategy](/docs/features/integrations/esp-integration#routing-rules)
- [Cost Optimization](/docs/features/integrations/esp-integration#cost-optimization)

---

## `/dashboard/settings/integrations/esp/webhooks` - Webhook Configuration

**User Story**: *"As an admin, I want to receive real-time delivery events from ESPs, so I can track email performance accurately."*

### Webhook Endpoints

**Postmark Webhook**:

- **Endpoint URL**: `https://api.penguinmails.com/webhooks/postmark`
- **Copy Button**: Copy URL to clipboard
- **Status**: Active (Green) or Inactive (Gray)
- **Events Received Today**: "1,234"
- **Last Event**: "2 minutes ago"

**Mailgun Webhook**:

- **Endpoint URL**: `https://api.penguinmails.com/webhooks/mailgun`
- **Copy Button**: Copy URL to clipboard
- **Status**: Active (Green) or Inactive (Gray)
- **Events Received Today**: "5,678"
- **Last Event**: "5 minutes ago"

### Event Subscriptions

**Postmark Events** (Checkboxes):

- ☑ **Delivery**: Email successfully delivered
- ☑ **Bounce**: Email bounced (hard or soft)
- ☑ **Open**: Recipient opened email
- ☑ **Click**: Recipient clicked link
- ☐ **Spam Complaint**: Recipient marked as spam
- ☐ **Subscription Change**: Unsubscribe or resubscribe

**Mailgun Events** (Checkboxes):

- ☑ **Delivered**: Email successfully delivered
- ☑ **Failed**: Permanent delivery failure
- ☑ **Opened**: Recipient opened email
- ☑ **Clicked**: Recipient clicked link
- ☐ **Unsubscribed**: Recipient unsubscribed
- ☐ **Complained**: Spam complaint

### Webhook Security

**Signature Verification**:

- **Postmark**: Uses `X-Postmark-Signature` header
- **Mailgun**: Uses `X-Mailgun-Signature` header with timestamp
- **Status**: Verified (Green checkmark) or "Configure signing key"

**IP Whitelist** (Optional):

- **Postmark IPs**: `50.31.156.6, 50.31.156.77, ...`
- **Mailgun IPs**: `209.61.151.0/24, ...`
- **"Add to Firewall" Button**: Generates firewall rules

### Webhook Logs

**Recent Events Table**:

- Columns: Timestamp, Provider, Event Type, Email, Status, Details
- **Filters**: Provider, Event Type, Date Range
- **Search**: Search by email or message ID
- **Export**: Download logs as CSV

**Example Rows**:

| Timestamp | Provider | Event | Email | Status | Details |
|-----------|----------|-------|-------|--------|---------|
| 2:34 PM | Postmark | Delivery | user@example.com | Success | Delivered in 1.2s |
| 2:33 PM | Mailgun | Open | lead@company.com | Success | Opened on mobile |
| 2:30 PM | Postmark | Bounce | invalid@test.com | Hard Bounce | User unknown |

**"View Full Log" Link**: Opens detailed webhook event viewer.

### Webhook Testing

**Test Webhook Section**:

- **Provider**: Dropdown (Postmark, Mailgun)
- **Event Type**: Dropdown (Delivery, Bounce, Open, Click)
- **"Send Test Event" Button**: Triggers test webhook from ESP
- **Result Display**: Shows received payload and processing status

### Technical Integration

- **Webhook Handler**: Express.js endpoint with signature verification
- **Event Processing**: Async job queue for webhook events
- **Retry Logic**: Exponential backoff for failed webhook processing

### Related Documentation

- [Webhook Integration](/docs/features/integrations/webhook-system)
- [ESP Webhooks](/docs/features/integrations/esp-integration#webhook-integration)
