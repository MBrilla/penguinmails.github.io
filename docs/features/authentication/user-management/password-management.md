---
title: "Password Management"
last_modified_date: "2025-12-15"
level: "2"
persona: "All Users, Developers"
---

# Password Management

Forgot, reset, and change password workflows.

## Forgot Password

Reset password via email link:

```text
Forgot Password

Enter your email address and we'll send you a link to reset your password.

Email Address: _______________

[Send Reset Link]
```

### Reset Flow

1. User enters email
2. System sends reset link (if email exists)
3. Link expires in 1 hour
4. User clicks link
5. Enter new password
6. Password updated
7. Auto-login with new password

### API Endpoints

```javascript
// Step 1: Request reset
POST /api/v1/auth/forgot-password
{
  "email": "user@example.com"
}

Response:
{
  "success": true,
  "message": "If that email exists, we sent a reset link"
  // Note: Don't reveal if email exists (security)
}

// Step 2: Reset password
POST /api/v1/auth/reset-password
{
  "token": "reset_token_abc123",
  "new_password": "NewSecurePass456!"
}

Response:
{
  "success": true,
  "message": "Password reset successful",
  "access_token": "eyJhbGc..." // Auto-login
}
```

## Change Password

Change password while logged in:

```text
Change Password

Current Password: _______________
New Password: _______________
Confirm New Password: _______________

[Update Password]
```

### Password Requirements

- ✅ Minimum 8 characters
- ✅ At least one uppercase letter
- ✅ At least one lowercase letter
- ✅ At least one number
- ✅ At least one special character (optional but recommended)

### API Endpoint

```javascript
POST /api/v1/auth/change-password
Authorization: Bearer {access_token}

{
  "current_password": "OldPass123!",
  "new_password": "NewPass456!"
}

Response:
{
  "success": true,
  "message": "Password updated successfully"
}
```

---

[← Back to User Management](./README)
