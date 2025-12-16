---
title: "Deliverability & Engagement Endpoints"
description: "API endpoints for deliverability analytics and engagement heatmap visualization"
last_modified_date: "2025-05-26"
level: 2
persona: "Developer"
keywords: ["deliverability", "engagement", "heatmap", "inbox-placement", "reputation"]
---

# Deliverability & Engagement Endpoints

These endpoints provide deliverability analytics for monitoring inbox placement and sender reputation, along with engagement heatmap data for optimizing send times.

## Get Deliverability Analytics

**Method**: `GET`
**URL**: `/api/v1/workspaces/{workspaceId}/analytics/deliverability`
**Purpose**: Analyze email deliverability metrics and sender reputation

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| workspaceId | string | Yes | Workspace identifier |

### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| time_range | string | No | Time range: `7d`, `30d`, `90d` (default: `30d`) |
| sender_address | string | No | Filter by sender email address |
| domain | string | No | Filter by sending domain |

### Request Example

```bash
curl -X GET "https://api.penguinmails.com/api/v1/workspaces/ws_xyz789/analytics/deliverability?time_range=30d" \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json"
```

### Response

```json
{
  "success": true,
  "data": {
    "workspace_id": "ws_xyz789",
    "time_range": {
      "start": "2025-10-26T00:00:00Z",
      "end": "2025-11-26T00:00:00Z"
    },
    "overall_health": {
      "score": 92,
      "status": "healthy",
      "trend": "improving"
    },
    "deliverability_breakdown": {
      "inbox_rate": 94.5,
      "spam_rate": 3.2,
      "bounce_rate": 2.3,
      "pending_rate": 0.0
    },
    "bounce_analysis": {
      "hard_bounces": 45,
      "soft_bounces": 105,
      "top_bounce_reasons": [
        {
          "reason": "mailbox_not_found",
          "count": 32,
          "percentage": 21.3
        },
        {
          "reason": "domain_not_found",
          "count": 13,
          "percentage": 8.7
        }
      ]
    },
    "provider_breakdown": [
      {
        "provider": "Gmail",
        "inbox_rate": 95.2,
        "volume_percentage": 45.0
      },
      {
        "provider": "Outlook",
        "inbox_rate": 93.1,
        "volume_percentage": 30.0
      },
      {
        "provider": "Yahoo",
        "inbox_rate": 91.5,
        "volume_percentage": 15.0
      }
    ],
    "authentication_status": {
      "spf_pass_rate": 99.8,
      "dkim_pass_rate": 99.9,
      "dmarc_pass_rate": 99.7
    }
  }
}
```

---

## Get Engagement Heatmap

**Method**: `GET`
**URL**: `/api/v1/workspaces/{workspaceId}/analytics/engagement-heatmap`
**Purpose**: Get engagement patterns by day of week and hour

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| workspaceId | string | Yes | Workspace identifier |

### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| time_range | string | No | Time range: `30d`, `90d`, `180d` (default: `90d`) |
| metric | string | No | Metric: `opens`, `clicks`, `replies` (default: `opens`) |
| timezone | string | No | Timezone for aggregation (default: `UTC`) |

### Request Example

```bash
curl -X GET "https://api.penguinmails.com/api/v1/workspaces/ws_xyz789/analytics/engagement-heatmap?metric=opens&timezone=America/New_York" \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json"
```

### Response

```json
{
  "success": true,
  "data": {
    "workspace_id": "ws_xyz789",
    "metric": "opens",
    "timezone": "America/New_York",
    "time_range": {
      "start": "2025-08-26T00:00:00Z",
      "end": "2025-11-26T00:00:00Z"
    },
    "heatmap": [
      {
        "day": "Monday",
        "day_index": 1,
        "hours": [
          {"hour": 0, "value": 12, "normalized": 0.15},
          {"hour": 1, "value": 5, "normalized": 0.06},
          {"hour": 9, "value": 85, "normalized": 1.0},
          {"hour": 10, "value": 78, "normalized": 0.92},
          {"hour": 14, "value": 65, "normalized": 0.76}
        ]
      },
      {
        "day": "Tuesday",
        "day_index": 2,
        "hours": [
          {"hour": 9, "value": 92, "normalized": 1.0},
          {"hour": 10, "value": 88, "normalized": 0.96},
          {"hour": 11, "value": 72, "normalized": 0.78}
        ]
      }
    ],
    "optimal_send_times": [
      {"day": "Tuesday", "hour": 9, "score": 0.98},
      {"day": "Wednesday", "hour": 10, "score": 0.95},
      {"day": "Thursday", "hour": 9, "score": 0.93}
    ]
  }
}
```

## Understanding Heatmap Data

The engagement heatmap provides insights for optimizing email send times:

**Normalized Values**: Values between 0 and 1 represent engagement intensity relative to the peak hour. A value of 1.0 indicates the hour with maximum engagement.

**Optimal Send Times**: The API calculates recommended send times based on historical engagement patterns, returning the top 3 time slots with highest predicted engagement.

**Timezone Handling**: Engagement data is aggregated according to the specified timezone, allowing accurate analysis across different recipient regions.

## Related Documentation

- **[Overview](overview)** - Analytics API overview and authentication
- **[Campaign Analytics](campaign-analytics)** - Campaign performance endpoints
- **[Comparison & Exports](comparison-exports)** - Campaign comparison and exports
- **[Sender Reputation](/docs/features/domains/sender-reputation)** - Health score and reputation tools
