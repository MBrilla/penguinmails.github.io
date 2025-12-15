---
title: "Session Management"
last_modified_date: "2025-12-15"
level: "2"
persona: "Developers"
---

# Session Management

JWT tokens and session security.

## JWT Tokens

Authentication uses JWT (JSON Web Tokens):

```javascript
{
  "header": {
    "alg": "HS256",
    "typ": "JWT"
  },
  "payload": {
    "user_id": "user_abc123",
    "tenant_id": "tenant_xyz789",
    "email": "user@example.com",
    "role": "admin",
    "iat": 1700830200, // Issued at
    "exp": 1700833800  // Expires (1 hour)
  },
  "signature": "..."
}
```

### Token Types

| Type | Duration | Purpose |
|------|----------|---------|
| Access Token | 1 hour | API requests |
| Refresh Token | 30 days | Get new access tokens |

### Token Refresh

```javascript
POST /api/v1/auth/refresh
{
  "refresh_token": "eyJhbGciOiJI..."
}

Response:
{
  "access_token": "eyJhbGciOiJI...", // New access token
  "expires_in": 3600
}
```

## Session Security

### Security Features

1. **Automatic Logout** - 30 minutes of inactivity
2. **Token Expiration** - Access tokens expire in 1 hour
3. **Refresh Rotation** - New refresh token on each refresh
4. **Revocation** - Tokens can be revoked server-side
5. **IP Tracking** - Log IP addresses for security monitoring

### Session Endpoints

```javascript
// Get active sessions
GET /api/v1/auth/sessions

Response:
{
  "sessions": [
    {
      "session_id": "sess_abc123",
      "device": "Chrome on macOS",
      "ip_address": "192.168.1.1",
      "last_active": "2025-11-24T10:30:00Z",
      "current": true
    }
  ]
}

// Revoke session
DELETE /api/v1/auth/sessions/{session_id}
```

---

[← Back to User Management](./README)
