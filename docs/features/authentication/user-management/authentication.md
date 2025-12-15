---
title: "Authentication"
last_modified_date: "2025-12-15"
level: "1"
persona: "All Users, Developers"
---

# Authentication

Signup, login, logout, and email verification flows.

## Sign Up (Registration)

Create a new account and tenant:

```text
Sign Up Form:
- Email Address *
- Full Name *
- Password * (min 8 characters)
- Company Name *
- [ ] I agree to Terms of Service

[Create Account]
```

### Sign Up Flow

1. User submits registration form
2. Backend creates tenant + owner user
3. Email verification sent
4. User clicks verification link
5. Account activated
6. Redirected to onboarding

### API Endpoint

```javascript
POST /api/v1/auth/signup

{
  "email": "user@example.com",
  "name": "John Doe",
  "password": "SecurePass123!",
  "company_name": "Acme Corp"
}

Response:
{
  "user_id": "user_abc123",
  "tenant_id": "tenant_xyz789",
  "email": "user@example.com",
  "email_verified": false,
  "verification_sent": true
}
```

## Login

Secure email/password authentication:

```text
Login Form:
- Email Address
- Password
- [x] Remember me (optional)

[Login] | [Forgot Password?]
```

### Login Flow

1. User enters email/password
2. NileDB validates credentials
3. JWT token generated (includes tenant_id)
4. Session established
5. Redirect to dashboard

### API Endpoint

```javascript
POST /api/v1/auth/login

{
  "email": "user@example.com",
  "password": "SecurePass123!"
}

Response:
{
  "access_token": "eyJhbGciOiJIUzI1NiIs...",
  "refresh_token": "eyJhbGciOiJIUzI1NiIs...",
  "user": {
    "id": "user_abc123",
    "email": "user@example.com",
    "name": "John Doe",
    "tenant_id": "tenant_xyz789",
    "role": "owner"
  },
  "expires_in": 3600
}
```

## Logout

End user session:

```javascript
POST /api/v1/auth/logout
Authorization: Bearer {access_token}

Response:
{
  "success": true,
  "message": "Successfully logged out"
}

// Backend:
// 1. Invalidate access token
// 2. Revoke refresh token
// 3. Clear server-side session
```

**Client-side:**

- Remove tokens from storage
- Clear user state
- Redirect to login page

## Email Verification

Verify email address after signup:

### Verification Email

```text
Subject: Verify your PenguinMails account

Hi John,

Welcome to PenguinMails! Please verify your email address:

[Verify Email Address]

Or copy this link:
https://app.penguinmails.com/verify-email?token=abc123

This link expires in 24 hours.
```

### Verification Flow

```javascript
GET /api/v1/auth/verify-email?token={verification_token}

Response:
{
  "success": true,
  "email_verified": true,
  "redirect_url": "/onboarding"
}
```

### Re-send Verification

```javascript
POST /api/v1/auth/resend-verification
{
  "email": "user@example.com"
}
```

---

[← Back to User Management](./README)
