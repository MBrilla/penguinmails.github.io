---
title: "Account Deletion"
last_modified_date: "2025-12-15"
level: "2"
persona: "All Users, Developers"
---

# Account Deletion

Delete user account with recovery grace period.

## Delete Account

```text
⚠️ Delete Account

This will permanently delete your account and all associated data.

Type "DELETE" to confirm: _______________

[Delete My Account]
```

## Deletion Process

```javascript
DELETE /api/v1/users/me
Authorization: Bearer {access_token}

{
  "confirmation": "DELETE",
  "password": "UserPass123!" // Confirm with password
}

Response:
{
  "success": true,
  "scheduled_deletion": "2025-12-24T10:00:00Z", // 30 days
  "message": "Account scheduled for deletion"
}
```

## What Happens

| Timeline | Action |
|----------|--------|
| Immediate | Account marked for deletion, access revoked |
| 30 days | Grace period for account recovery |
| After 30 days | Permanent deletion of all data |

---

[← Back to User Management](./README)
