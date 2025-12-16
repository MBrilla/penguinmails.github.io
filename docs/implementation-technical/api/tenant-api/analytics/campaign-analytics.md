---
title: "Campaign Analytics Endpoints"
description: "API endpoints for campaign performance metrics and workspace analytics overview"
last_modified_date: "2025-05-26"
level: 2
persona: "Developer"
keywords: ["campaign-analytics", "performance-metrics", "workspace-analytics", "api-endpoints"]
---

# Campaign Analytics Endpoints

These endpoints provide campaign-level performance metrics and workspace-wide analytics summaries. Use these endpoints to track email engagement, monitor campaign success, and generate performance reports.

## Get Campaign Analytics

**Method**: `GET`
**URL**: `/api/v1/workspaces/{workspaceId}/campaigns/{campaignId}/analytics`
**Purpose**: Retrieve comprehensive analytics for a specific campaign

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| workspaceId | string | Yes | Workspace identifier |
| campaignId | string | Yes | Campaign identifier |

### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| time_range | string | No | Time range: `7d`, `30d`, `90d`, `all` (default: `30d`) |
| granularity | string | No | Data granularity: `hour`, `day`, `week` (default: `day`) |
| include_time_series | boolean | No | Include time series data (default: `false`) |

### Request Example

```bash
curl -X GET "https://api.penguinmails.com/api/v1/workspaces/ws_xyz789/campaigns/cmp_abc123/analytics?time_range=30d&include_time_series=true" \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json"
```

### Response

```json
{
  "success": true,
  "data": {
    "campaign_id": "cmp_abc123",
    "campaign_name": "Q4 Outreach Campaign",
    "time_range": {
      "start": "2025-10-26T00:00:00Z",
      "end": "2025-11-26T00:00:00Z"
    },
    "summary": {
      "emails_sent": 5000,
      "emails_delivered": 4850,
      "emails_bounced": 150,
      "emails_opened": 1940,
      "emails_clicked": 485,
      "emails_replied": 145,
      "spam_complaints": 5,
      "unsubscribes": 25
    },
    "rates": {
      "delivery_rate": 97.0,
      "open_rate": 40.0,
      "click_rate": 10.0,
      "click_to_open_rate": 25.0,
      "reply_rate": 3.0,
      "bounce_rate": 3.0,
      "spam_rate": 0.1,
      "unsubscribe_rate": 0.5
    },
    "time_series": [
      {
        "date": "2025-11-01",
        "sent": 200,
        "delivered": 195,
        "opened": 78,
        "clicked": 20,
        "replied": 6
      },
      {
        "date": "2025-11-02",
        "sent": 200,
        "delivered": 194,
        "opened": 82,
        "clicked": 22,
        "replied": 7
      }
    ]
  }
}
```

---

## Get Workspace Analytics Overview

**Method**: `GET`
**URL**: `/api/v1/workspaces/{workspaceId}/analytics/overview`
**Purpose**: Get aggregated analytics across all campaigns in a workspace

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| workspaceId | string | Yes | Workspace identifier |

### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| time_range | string | No | Time range: `7d`, `30d`, `90d`, `ytd` (default: `30d`) |
| campaign_status | string | No | Filter by status: `active`, `completed`, `all` (default: `all`) |

### Request Example

```bash
curl -X GET "https://api.penguinmails.com/api/v1/workspaces/ws_xyz789/analytics/overview?time_range=30d" \
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
    "totals": {
      "campaigns_active": 5,
      "campaigns_completed": 12,
      "emails_sent": 45000,
      "emails_delivered": 43650,
      "unique_opens": 15277,
      "unique_clicks": 3927
    },
    "average_rates": {
      "delivery_rate": 97.0,
      "open_rate": 35.0,
      "click_rate": 9.0,
      "reply_rate": 2.5
    },
    "top_performing_campaigns": [
      {
        "campaign_id": "cmp_abc123",
        "campaign_name": "Q4 Outreach",
        "open_rate": 42.5,
        "reply_rate": 4.2
      }
    ],
    "trends": {
      "open_rate_change": 5.2,
      "reply_rate_change": 0.8,
      "period_comparison": "vs_previous_30d"
    }
  }
}
```

## Related Documentation

- **[Overview](overview)** - Analytics API overview and authentication
- **[Deliverability & Engagement](deliverability-engagement)** - Deliverability and engagement endpoints
- **[Comparison & Exports](comparison-exports)** - Campaign comparison and data exports
- **[Campaign Management API](/docs/implementation-technical/api/tenant-api/campaigns)** - Campaign CRUD operations
