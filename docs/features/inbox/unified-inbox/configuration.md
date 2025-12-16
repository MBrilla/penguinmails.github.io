---
title: "Unified Inbox Configuration"
description: "AI categorization rules, inbox filtering, reply templates, and automation configuration"
level: "2"
status: "AVAILABLE"
roadmap_timeline: "Q1 2026"
priority: "Critical"
keywords: [configuration, ai-categorization, templates, automation, filtering]
---

# Unified Inbox Configuration

Configure the Unified Inbox for optimal workflow efficiency with AI categorization, custom views, templates, and automation rules.

## AI Categorization Rules

Configure how the AI interprets and tags incoming messages.

```yaml
ai_categorization:
  enabled: true
  confidence_threshold: 0.85

  categories:
    - name: "Interested"
      keywords: ["interested", "send more", "pricing", "demo", "call"]
      action: "mark_priority_high"

    - name: "Meeting Request"
      keywords: ["calendar", "schedule", "time to chat", "tuesday"]
      action: "trigger_webhook:meeting_intent"

    - name: "Not Interested"
      keywords: ["unsubscribe", "remove me", "stop", "not interested"]
      action: "unsubscribe_contact"

    - name: "Out of Office"
      keywords: ["out of office", "vacation", "limited access"]
      action: "snooze:3_days"
```

### Category Actions

| Category | Action | Result |
|----------|--------|--------|
| Interested | `mark_priority_high` | Thread appears at top of inbox |
| Meeting Request | `trigger_webhook:meeting_intent` | Notify external systems |
| Not Interested | `unsubscribe_contact` | Auto-unsubscribe contact |
| Out of Office | `snooze:3_days` | Hide thread for 3 days |

## Inbox Filtering & Views

Create custom views for different workflows.

### "High Value Leads" View Example

```json
{
  "name": "High Value Leads",
  "filters": {
    "lead_score": { "gt": 80 },
    "status": ["open", "replied"],
    "tags": ["Interested", "Meeting Request"],
    "last_activity": { "lt": "2 days ago" }
  },
  "columns": ["contact_name", "company", "lead_score", "last_reply", "status"],
  "sort": { "field": "lead_score", "direction": "desc" }
}
```

### Filter Options

| Filter | Type | Description |
|--------|------|-------------|
| `lead_score` | Numeric | Filter by lead score threshold |
| `status` | Array | Open, replied, archived, snoozed |
| `tags` | Array | AI categories or custom tags |
| `last_activity` | Date | Time since last message |
| `assigned_to` | User | Filter by assignment |
| `campaign_id` | UUID | Filter by source campaign |

## Reply Templates & Variables

Standardize responses with dynamic templates.

### Template: "Booking Request"

```html
Hi {{firstName}},

Thanks for the quick reply! I'd love to show you how {{company}} can help with {{pain_point}}.

Are you free for a 15-min chat? You can book a time here:
{{calendar_link}}

Best,
{{sender_name}}
```

### Available Variables

| Variable | Description |
|----------|-------------|
| `{{firstName}}` | Contact's first name |
| `{{lastName}}` | Contact's last name |
| `{{company}}` | Contact's company name |
| `{{pain_point}}` | Identified pain point from campaign |
| `{{calendar_link}}` | Sender's booking link |
| `{{sender_name}}` | Sender's name |
| `{{campaign.name}}` | Source campaign name |

## Automation Rules

Trigger actions based on inbox events.

### Rule: "Auto-CRM Sync"

```yaml
trigger:
  event: "thread_status_changed"
  new_status: "Interested"

actions:
  - type: "sync_to_crm"
    crm_provider: "hubspot"
    object: "deal"
    stage: "qualified_lead"

  - type: "notify_slack"
    channel: "#sales-wins"
    message: "🔥 New interested lead: {{contact.email}} from {{campaign.name}}"
```

### Trigger Events

| Event | Description |
|-------|-------------|
| `thread_status_changed` | Thread status updated |
| `new_message_received` | New inbound message |
| `message_sent` | Reply sent |
| `thread_assigned` | Thread assigned to user |
| `thread_snoozed` | Thread snoozed |

### Action Types

| Action | Description |
|--------|-------------|
| `sync_to_crm` | Push to CRM system |
| `notify_slack` | Send Slack notification |
| `send_email` | Trigger email action |
| `update_contact` | Update contact record |
| `add_tag` | Add tag to thread |

## Related Documentation

- [Unified Inbox Overview](/docs/features/inbox/unified-inbox/quick-start) - Getting started guide
- [Technical Implementation](/docs/features/inbox/unified-inbox/technical-implementation) - Database and API
- [Inbox Rotation](/docs/features/inbox/inbox-rotation) - Related feature

---

**Last Updated:** November 25, 2025
**Status:** Available - Core Feature
