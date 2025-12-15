---
title: "API Key List Route"
description: API key list page with table structure and actions
last_modified_date: 2025-01-15
level: 3
persona: ["frontend-developer"]
keywords: [routes, API-keys, table, list, UI]
---

# API Key List Route

**Path:** `/dashboard/settings/developers/api-keys`
**Method:** GET
**Authentication:** Required (tenant user session)
**Layout:** Settings page with sidebar navigation

## Page Structure

```text
┌─────────────────────────────────────────────────────────────┐
│ Settings Sidebar │ API Keys                                  │
├──────────────────┼───────────────────────────────────────────┤
│ Profile          │ ┌─────────────────────────────────────┐   │
│ Team             │ │ API Keys                            │   │
│ Billing          │ │                                     │   │
│ Infrastructure   │ │ Manage API keys for programmatic   │   │
│ > Developers     │ │ access to PenguinMails.            │   │
│   - API Keys     │ │                                     │   │
│   - Webhooks     │ │ [Create API Key]                   │   │
│ Security         │ └─────────────────────────────────────┘   │
│                  │                                            │
│                  │ ┌────────────────────────────────────────┐│
│                  │ │ Name          │ Key        │ Permissions││
│                  │ │ Production    │ pm_live... │ [send]     ││
│                  │ │ Server        │            │ [analytics]││
│                  │ │ [Copy] [Regenerate] [Revoke]           ││
│                  │ └────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────┘
```

## UI Components

### Page Header

- **Title**: "API Keys"
- **Description**: "Manage API keys for programmatic access to PenguinMails."
- **Primary action**: "Create API Key" button (top right)

### API Key Table

| Column | Description |
|--------|-------------|
| Name | User-provided key name |
| Key | Masked key value (`pm_live_abc...xyz`) |
| Permissions | Badge list of scopes |
| Rate Limit | Requests per minute |
| Status | Active (green) or Revoked (red) |
| Last Used | Relative time ("2 hours ago") |
| Actions | Copy, Regenerate, Revoke buttons |

**Row Details (expandable):**
- Created date
- Total requests (lifetime)
- Error count
- Request chart (last 7 days)

### Table Actions

| Action | Behavior |
|--------|----------|
| Copy | Copies masked key to clipboard, shows toast |
| Regenerate | Opens confirmation modal, generates new key |
| Revoke | Opens confirmation modal, marks key inactive |

### Empty State

**Displayed when:** No API keys exist

- **Icon**: Key icon
- **Title**: "No API keys yet"
- **Description**: "Create your first API key to start using the PenguinMails API programmatically."
- **Action**: "Create API Key" button

## API Call

```typescript
GET /api/v1/platform/api-keys

Response:
{
  "api_keys": [
    {
      "key_id": "550e8400-e29b-41d4-a716-446655440000",
      "name": "Production Server",
      "masked_key": "pm_live_a1b...o5p6",
      "permissions": ["send_email", "read_analytics"],
      "rate_limit": 300,
      "status": "active",
      "created_at": "2025-11-26T10:00:00Z",
      "last_used": "2025-11-26T15:30:00Z",
      "request_count": 15234,
      "error_count": 42
    }
  ]
}
```
