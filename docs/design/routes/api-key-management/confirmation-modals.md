---
title: "Confirmation Modals"
description: Regenerate and revoke API key confirmation modals
last_modified_date: 2025-01-15
level: 3
persona: ["frontend-developer"]
keywords: [modal, API-keys, regenerate, revoke, confirmation]
---

# Confirmation Modals

Confirmation dialogs for destructive API key operations.

## Regenerate API Key Modal

**Trigger:** Click "Regenerate" button
**Modal Type:** Small centered confirmation modal

### Structure

```text
┌─────────────────────────────────────────────────┐
│ ⚠️ Regenerate API Key?                    [X]   │
├─────────────────────────────────────────────────┤
│                                                 │
│ This will immediately revoke the old key and   │
│ generate a new one. Any applications using the │
│ old key will stop working until you update     │
│ them with the new key.                         │
│                                                 │
│ Key: Production Server                         │
│ Current Key: pm_live_a1b...o5p6                │
│                                                 │
│ [Cancel]                    [Regenerate Key]   │
└─────────────────────────────────────────────────┘
```

### Actions

| Button | Action |
|--------|--------|
| Cancel | Closes modal without regenerating |
| Regenerate Key | Regenerates key, opens success modal with new key |

### API Call

```typescript
POST /api/v1/platform/api-keys/{key_id}/regenerate

Response (200 OK):
{
  "api_key": "pm_live_c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8",
  "key_id": "550e8400-e29b-41d4-a716-446655440000",
  "name": "Production Server",
  "permissions": ["send_email", "read_analytics", "manage_contacts"],
  "rate_limit": 300,
  "created_at": "2025-11-26T16:00:00Z",
  "warning": "Old key has been revoked. Update your application immediately."
}
```

## Revoke API Key Modal

**Trigger:** Click "Revoke" button
**Modal Type:** Small centered confirmation modal

### Structure

```text
┌─────────────────────────────────────────────────┐
│ ⚠️ Revoke API Key?                        [X]   │
├─────────────────────────────────────────────────┤
│                                                 │
│ This action cannot be undone. The key will be  │
│ permanently revoked and any applications using │
│ it will stop working immediately.              │
│                                                 │
│ Key: Production Server                         │
│ Current Key: pm_live_a1b...o5p6                │
│                                                 │
│ [Cancel]                         [Revoke Key]  │
└─────────────────────────────────────────────────┘
```

### Actions

| Button | Action |
|--------|--------|
| Cancel | Closes modal without revoking |
| Revoke Key | Revokes key, shows success toast, refreshes list |

### API Call

```typescript
DELETE /api/v1/platform/api-keys/{key_id}

Response (200 OK):
{
  "message": "API key revoked successfully",
  "key_id": "550e8400-e29b-41d4-a716-446655440000",
  "revoked_at": "2025-11-26T16:30:00Z"
}
```

## Design Guidelines

### Warning Style

- Yellow/amber background for caution
- Clear, concise warning text
- Display affected key name and masked value

### Button Placement

- Cancel button on left (secondary style)
- Destructive action on right (danger style for revoke)
- Clear visual distinction between cancel and confirm
