---
title: "Event Reference Route"
description: "Route specification for the webhook event documentation page"
last_modified_date: "2025-01-20"
level: 3
persona: Developers
keywords: webhooks, events, payload schema, documentation
---

# Event Reference Route

**Route:** `/dashboard/settings/webhooks/events`

**Access:** Admin, Developer

**User Story:** *"As a developer, I want to see all available webhook events and their payloads, so I can choose which events to subscribe to."*

## Event Categories

**Sidebar Navigation:**
- Email Events (7)
- Campaign Events (3)
- Contact Events (3)
- System Events (2)

## Event Documentation

### Example: email.opened Event

**Event Type:** `email.opened`

**Description:** "Triggered when a recipient opens an email. Includes location and device information."

**Frequency:** "High - Can be triggered multiple times per email"

**Payload Schema:**
```json
{
  "id": "string (UUID)",
  "type": "email.opened",
  "created_at": "string (ISO 8601)",
  "data": {
    "email_id": "string (UUID)",
    "campaign_id": "string (UUID)",
    "contact": {
      "id": "string (UUID)",
      "email": "string",
      "first_name": "string",
      "last_name": "string"
    },
    "opened_at": "string (ISO 8601)",
    "user_agent": "string",
    "ip_address": "string",
    "location": {
      "city": "string",
      "region": "string",
      "country": "string"
    }
  }
}
```

**Example Payload:**
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
    "user_agent": "Mozilla/5.0 (iPhone; CPU iPhone OS 14_0 like Mac OS X)",
    "ip_address": "192.168.1.1",
    "location": {
      "city": "New York",
      "region": "NY",
      "country": "US"
    }
  }
}
```

**Copy Button:** Copy example payload

**Related Events:**
- email.sent
- email.delivered
- email.clicked

**Use Cases:**
- Update CRM with engagement data
- Trigger follow-up workflows
- Track email performance

## Event Comparison Table

**All Events Summary:**

| Event Type | Frequency | Retry Safe | Typical Use Case |
|------------|-----------|------------|------------------|
| email.sent | High | Yes | Track sending status |
| email.delivered | High | Yes | Confirm delivery |
| email.opened | Very High | Yes | Measure engagement |
| email.clicked | High | Yes | Track link clicks |
| email.bounced | Medium | Yes | Update contact status |
| campaign.launched | Low | Yes | Trigger notifications |
| contact.created | Medium | Yes | Sync to CRM |

## User Journey Context

Reference during webhook setup and development.

## Related Documentation

- [Event Reference](/docs/features/integrations/webhook-system#event-reference)
- [Webhook System Overview](/docs/features/integrations/webhook-system)
