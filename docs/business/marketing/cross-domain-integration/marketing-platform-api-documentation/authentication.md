---
title: API Authentication
description: OAuth 2.0, API keys, JWT tokens, and security headers
last_modified_date: 2025-01-15
level: 3
persona: ["developer"]
keywords: [authentication, OAuth, API-key, JWT, security]
---

# API Authentication

Authentication methods and security requirements for Marketing Platform APIs.

## Authentication Methods

### OAuth 2.0 (Recommended)

```json
{
  "auth_method": "oauth2",
  "grant_types": ["client_credentials", "authorization_code"],
  "token_endpoint": "https://api.marketingplatform.com/oauth/token",
  "scope": "read:campaigns write:campaigns read:analytics",
  "token_lifetime": "3600 seconds"
}
```

### API Key Authentication

```json
{
  "auth_method": "api_key",
  "header_name": "X-API-Key",
  "key_format": "pk_live_[32_character_hex_string]",
  "rate_limits": {
    "requests_per_hour": 1000,
    "burst_limit": 100
  }
}
```

### JWT Token Authentication

```json
{
  "auth_method": "jwt",
  "algorithm": "RS256",
  "claims": ["sub", "aud", "exp", "scope"],
  "token_endpoint": "https://api.marketingplatform.com/auth/jwt"
}
```

## Security Headers

All API requests must include these headers:

```http
Authorization: Bearer {access_token}
Content-Type: application/json
X-API-Version: v1
X-Request-ID: {uuid}
User-Agent: {integration_name}/{version}
```

## Related Documentation

- [API Overview](./)
- [Core APIs](./core-apis)
- [Rate Limits & Errors](./rate-limits-errors)
