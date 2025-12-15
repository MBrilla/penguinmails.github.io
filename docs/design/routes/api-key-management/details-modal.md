---
title: "API Key Details Modal"
description: Modal showing API key details with usage and security tabs
last_modified_date: 2025-01-15
level: 3
persona: ["frontend-developer"]
keywords: [modal, API-keys, details, usage, security]
---

# API Key Details Modal

**Trigger:** Click on API key row in table
**Modal Type:** Large centered modal with tabs

## Modal Structure

```text
┌─────────────────────────────────────────────────┐
│ Production Server                         [X]   │
├─────────────────────────────────────────────────┤
│ [Overview] [Usage] [Security]                   │
├─────────────────────────────────────────────────┤
│                                                 │
│ Key ID: 550e8400-e29b-41d4-a716-446655440000   │
│ Masked Key: pm_live_a1b...o5p6          [Copy] │
│                                                 │
│ Permissions                                     │
│ [send_email] [read_analytics] [manage_contacts]│
│                                                 │
│ Rate Limit: 300 requests/min                   │
│ Status: Active                                  │
│                                                 │
│ Created: Nov 26, 2025 at 10:00 AM             │
│ Last Used: 2 hours ago                         │
│                                                 │
│ Usage Statistics                                │
│ Total Requests: 15,234                         │
│ Error Count: 42 (0.28%)                        │
│                                                 │
│ [Regenerate Key]  [Revoke Key]                 │
└─────────────────────────────────────────────────┘
```

## Tabs

### Overview Tab

Key metadata and basic statistics:

| Field | Description |
|-------|-------------|
| Key ID | UUID identifier |
| Masked Key | Truncated key with copy button |
| Permissions | Badge list of scopes |
| Rate Limit | Requests per minute |
| Status | Active or Revoked |
| Created | Creation timestamp |
| Last Used | Last API call timestamp |
| Total Requests | Lifetime request count |
| Error Count | Total errors with percentage |

**Actions**: Regenerate Key, Revoke Key buttons

### Usage Tab

| Component | Description |
|-----------|-------------|
| Requests per day chart | Line chart for last 30 days |
| Error rate trend | Error percentage over time |
| Top endpoints table | Endpoint, count, avg response time |
| Geographic distribution | Map if data available |

### Security Tab

| Component | Description |
|-----------|-------------|
| Audit log | Last 50 events |
| IP addresses | Last 10 unique IPs used |
| Security recommendations | Best practices |
| Rotation reminder | Alert if >90 days old |

## API Call

```typescript
GET /api/v1/platform/api-keys/{key_id}

Response (200 OK):
{
  "key_id": "550e8400-e29b-41d4-a716-446655440000",
  "name": "Production Server",
  "masked_key": "pm_live_a1b...o5p6",
  "permissions": ["send_email", "read_analytics", "manage_contacts"],
  "rate_limit": 300,
  "status": "active",
  "created_at": "2025-11-26T10:00:00Z",
  "last_used": "2025-11-26T15:30:00Z",
  "request_count": 15234,
  "error_count": 42,
  "usage_by_day": [
    { "date": "2025-11-26", "requests": 1234, "errors": 5 },
    { "date": "2025-11-25", "requests": 2345, "errors": 8 }
  ],
  "top_endpoints": [
    { "endpoint": "/api/v1/emails/send", "count": 12000, "avg_response_time_ms": 250 },
    { "endpoint": "/api/v1/analytics/campaigns", "count": 3234, "avg_response_time_ms": 150 }
  ]
}
```
