---
title: Rate Limits and Error Handling
description: API rate limits, quotas, error codes, and retry logic
last_modified_date: 2025-01-15
level: 3
persona: ["developer"]
keywords: [rate-limit, errors, quotas, retry, API]
---

# Rate Limits and Error Handling

API rate limits, quotas, error handling, and retry logic.

## Rate Limit Headers

All API responses include rate limiting information:

```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 995
X-RateLimit-Reset: 1705324800
X-RateLimit-Window: 3600
```

## Rate Limit Tiers

| Tier | Requests/Hour | Burst Limit | Cost/Month |
|------|---------------|-------------|------------|
| Starter | 1,000 | 100 | $49 |
| Professional | 10,000 | 1,000 | $199 |
| Enterprise | 100,000 | 10,000 | $799 |
| Custom | Unlimited | Unlimited | Custom |

## Get Current Usage

```http
GET /api/v1/integrations/usage
```

**Response:**

```json
{
  "current_period": {
    "start": "2025-01-15T00:00:00Z",
    "end": "2025-01-15T23:59:59Z",
    "requests_made": 1250,
    "requests_limit": 10000,
    "percentage_used": 12.5
  },
  "projected_monthly": {
    "estimated_requests": 37500,
    "estimated_cost": 199.00,
    "tier": "Professional"
  }
}
```

## Error Response Format

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid campaign configuration",
    "details": {
      "field": "target_audience.criteria.company_size",
      "issue": "Invalid size range provided",
      "expected": "Valid company size range (1-100000)",
      "received": "150-abc"
    },
    "request_id": "req_789xyz",
    "timestamp": "2025-01-15T10:30:00Z"
  }
}
```

## Common Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `AUTHENTICATION_ERROR` | 401 | Invalid or missing authentication |
| `AUTHORIZATION_ERROR` | 403 | Insufficient permissions |
| `VALIDATION_ERROR` | 400 | Invalid request data |
| `NOT_FOUND` | 404 | Requested resource not found |
| `RATE_LIMIT_EXCEEDED` | 429 | Rate limit exceeded |
| `QUOTA_EXCEEDED` | 402 | API quota exceeded |
| `INTERNAL_ERROR` | 500 | Server error |

## Retry Logic

### Exponential Backoff Configuration

```json
{
  "retry_policy": {
    "max_attempts": 3,
    "base_delay": "1s",
    "max_delay": "60s",
    "backoff_multiplier": 2,
    "retryable_codes": [429, 500, 502, 503, 504]
  }
}
```

### Recommended Retry Strategy

| Attempt | Delay |
|---------|-------|
| 1 | 1 second |
| 2 | 2 seconds |
| 3 | 4 seconds |

Only retry for transient errors (429, 5xx). Do not retry for client errors (4xx except 429).

## Related Documentation

- [API Overview](./)
- [Authentication](./authentication)
- [SDKs & Testing](./sdks-testing)
