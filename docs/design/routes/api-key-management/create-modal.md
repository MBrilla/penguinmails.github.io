---
title: "Create API Key Modal"
description: Modal for creating new API keys with permissions
last_modified_date: 2025-01-15
level: 3
persona: ["frontend-developer"]
keywords: [modal, API-keys, create, permissions, form]
---

# Create API Key Modal

**Trigger:** Click "Create API Key" button
**Modal Type:** Centered modal with backdrop

## Modal Structure

```text
┌─────────────────────────────────────────────────┐
│ Create API Key                            [X]   │
├─────────────────────────────────────────────────┤
│                                                 │
│ Name *                                          │
│ ┌─────────────────────────────────────────────┐│
│ │ Production Server                           ││
│ └─────────────────────────────────────────────┘│
│                                                 │
│ Permissions *                                   │
│ ☑ Send emails (send_email)                     │
│ ☑ Read analytics (read_analytics)              │
│ ☑ Manage contacts (manage_contacts)            │
│ ☐ Manage campaigns (manage_campaigns)          │
│ ☐ Manage templates (manage_templates)          │
│                                                 │
│ Rate Limit                                      │
│ 300 requests per minute (Pro plan)             │
│                                                 │
│ [Cancel]                    [Generate Key]     │
└─────────────────────────────────────────────────┘
```

## Form Fields

### Name (required)

- **Type**: Text input
- **Placeholder**: "e.g., Production Server, Staging Environment"
- **Validation**: 1-50 characters, alphanumeric and spaces
- **Errors**: "Name is required" or "Name must be 1-50 characters"

### Permissions (required)

Multi-select checkboxes with available scopes:

| Permission | Scope | Description |
|------------|-------|-------------|
| Send emails | `send_email` | Send emails via API |
| Read analytics | `read_analytics` | Read campaign and email analytics |
| Manage contacts | `manage_contacts` | Create, update, delete contacts |
| Manage campaigns | `manage_campaigns` | Create, update, delete campaigns |
| Manage templates | `manage_templates` | Create, update, delete templates |
| Manage domains | `manage_domains` | Add, verify, delete domains |
| Read inbox | `read_inbox` | Read unified inbox messages |
| Manage webhooks | `manage_webhooks` | Configure webhooks |

**Validation**: At least one permission required
**Error**: "Select at least one permission"

### Rate Limit (display only)

Tier-based rate limits (read-only display):

| Tier | Rate Limit |
|------|------------|
| Starter | 60 requests/min |
| Pro | 300 requests/min |
| Enterprise | 1000 requests/min |

**Link**: "Upgrade plan to increase rate limit"

## Modal Actions

| Button | Action |
|--------|--------|
| Cancel | Closes modal without creating key |
| Generate Key | Validates form, creates API key, opens success modal |

## API Call

```typescript
POST /api/v1/platform/api-keys

Request:
{
  "name": "Production Server",
  "permissions": ["send_email", "read_analytics", "manage_contacts"]
}

Response (201 Created):
{
  "api_key": "pm_live_a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6",
  "key_id": "550e8400-e29b-41d4-a716-446655440000",
  "name": "Production Server",
  "permissions": ["send_email", "read_analytics", "manage_contacts"],
  "rate_limit": 300,
  "created_at": "2025-11-26T10:00:00Z",
  "warning": "Store this key securely. It will not be shown again."
}
```
