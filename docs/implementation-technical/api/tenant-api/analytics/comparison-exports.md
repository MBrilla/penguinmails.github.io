---
title: "Campaign Comparison & Exports"
description: "API endpoints for comparing campaign performance and exporting analytics data"
last_modified_date: "2025-05-26"
level: 2
persona: "Developer"
keywords: ["campaign-comparison", "analytics-export", "data-export", "statistical-significance"]
---

# Campaign Comparison & Exports

These endpoints enable side-by-side campaign performance comparison with statistical significance testing, along with data export functionality for external analysis.

## Compare Campaigns

**Method**: `POST`
**URL**: `/api/v1/workspaces/{workspaceId}/analytics/compare`
**Purpose**: Compare performance metrics across multiple campaigns

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| workspaceId | string | Yes | Workspace identifier |

### Request Body

```json
{
  "campaign_ids": ["cmp_abc123", "cmp_def456"],
  "metrics": ["open_rate", "click_rate", "reply_rate"],
  "time_range": "30d"
}
```

### Response

```json
{
  "success": true,
  "data": {
    "comparison": [
      {
        "campaign_id": "cmp_abc123",
        "campaign_name": "Q4 Outreach",
        "metrics": {
          "open_rate": 35.5,
          "click_rate": 6.2,
          "reply_rate": 3.1,
          "delivery_rate": 98.5
        }
      },
      {
        "campaign_id": "cmp_def456",
        "campaign_name": "Product Launch",
        "metrics": {
          "open_rate": 28.3,
          "click_rate": 4.8,
          "reply_rate": 2.2,
          "delivery_rate": 97.8
        }
      }
    ],
    "statistical_significance": [
      {
        "metric": "open_rate",
        "winner": "cmp_abc123",
        "p_value": 0.03,
        "is_significant": true
      }
    ]
  }
}
```

---

## Export Analytics Data

**Method**: `POST`
**URL**: `/api/v1/workspaces/{workspaceId}/analytics/export`
**Purpose**: Generate and export analytics data in various formats

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| workspaceId | string | Yes | Workspace identifier |

### Request Body

```json
{
  "export_type": "campaign_performance",
  "format": "csv",
  "time_range": "30d",
  "campaign_ids": ["cmp_abc123"],
  "include_time_series": true
}
```

### Response

```json
{
  "success": true,
  "data": {
    "export_id": "exp_xyz123",
    "status": "processing",
    "estimated_completion": "2025-11-26T10:05:00Z",
    "download_url": null
  }
}
```

---

## Get Export Status

**Method**: `GET`
**URL**: `/api/v1/workspaces/{workspaceId}/analytics/exports/{exportId}`
**Purpose**: Check status of analytics export and retrieve download URL

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| workspaceId | string | Yes | Workspace identifier |
| exportId | string | Yes | Export job identifier |

### Response

```json
{
  "success": true,
  "data": {
    "export_id": "exp_xyz123",
    "status": "completed",
    "format": "csv",
    "file_size_bytes": 524288,
    "created_at": "2025-11-26T10:00:00Z",
    "completed_at": "2025-11-26T10:03:00Z",
    "download_url": "https://s3.amazonaws.com/exports/exp_xyz123.csv?expires=3600",
    "expires_at": "2025-11-26T11:03:00Z"
  }
}
```

---

## Data Models

### Analytics Summary

```typescript
interface AnalyticsSummary {
  emails_sent: number;
  emails_delivered: number;
  emails_bounced: number;
  emails_opened: number;
  emails_clicked: number;
  emails_replied: number;
  spam_complaints: number;
  unsubscribes: number;
}
```

### Analytics Rates

```typescript
interface AnalyticsRates {
  delivery_rate: number;      // Percentage (0-100)
  open_rate: number;           // Percentage (0-100)
  click_rate: number;          // Percentage (0-100)
  click_to_open_rate: number;  // Percentage (0-100)
  reply_rate: number;          // Percentage (0-100)
  bounce_rate: number;         // Percentage (0-100)
  spam_rate: number;           // Percentage (0-100)
  unsubscribe_rate: number;    // Percentage (0-100)
}
```

### Time Series Data Point

```typescript
interface TimeSeriesDataPoint {
  date: string;              // ISO 8601 date
  sent: number;
  delivered: number;
  opened: number;
  clicked: number;
  replied: number;
  bounced?: number;
  spam_complaints?: number;
}
```

---

## Error Responses

### Campaign Not Found

```json
{
  "success": false,
  "error": {
    "code": "CAMPAIGN_NOT_FOUND",
    "message": "Campaign with ID cmp_abc123 not found in workspace ws_xyz789"
  }
}
```

### Invalid Time Range

```json
{
  "success": false,
  "error": {
    "code": "INVALID_TIME_RANGE",
    "message": "Start date must be before end date"
  }
}
```

### Export Failed

```json
{
  "success": false,
  "error": {
    "code": "EXPORT_FAILED",
    "message": "Failed to generate export: insufficient data"
  }
}
```

---

## Related Documentation

### Feature Documentation

- **[Analytics & Reporting Features](/docs/features/analytics/core-analytics/overview)** - Feature overview and capabilities
- **[Enhanced Analytics](/docs/features/analytics/enhanced-analytics/overview)** - Q1 2026 advanced analytics features
- **[Manual Reporting](/docs/features/analytics/manual-reporting)** - Scheduled reports and data export

### API Documentation

- **[Overview](overview)** - Analytics API overview and authentication
- **[Campaign Analytics](campaign-analytics)** - Campaign performance endpoints
- **[Deliverability & Engagement](deliverability-engagement)** - Deliverability and engagement endpoints

### Technical Architecture

- **[OLAP Analytics Schema](/docs/implementation-technical/database-infrastructure/olap-analytics-schema-guide)** - Database architecture for analytics
- **[Queue System](/docs/features/queue/background-jobs)** - Background job processing
