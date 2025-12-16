---
title: "Tenant Analytics API Overview"
description: "Overview of Analytics API endpoints for tenant workspace performance tracking and reporting"
last_modified_date: "2025-05-26"
level: 2
persona: "Developer"
keywords: ["tenant-api", "analytics", "api-overview", "performance-tracking"]
---

# Tenant Analytics API

The Analytics API provides endpoints for tracking campaign performance, engagement metrics, and deliverability analytics within tenant workspaces. These endpoints enable data-driven decision making through comprehensive performance monitoring and reporting capabilities.

## Key Capabilities

The Analytics API delivers comprehensive performance insights through campaign performance metrics that track opens, clicks, replies, and conversion rates across all campaigns. Engagement analytics provide detailed behavioral data including time-series analysis and heatmap visualization. Deliverability monitoring tracks inbox placement rates, bounce rates, and spam complaint metrics. The comparison tools enable side-by-side performance analysis of multiple campaigns with statistical significance testing. Export functionality supports data extraction in CSV and JSON formats for external analysis.

## Technical Architecture

All analytics endpoints operate within the tenant context, requiring workspace-level authentication. Data is aggregated in near real-time with a maximum delay of 5 minutes from event occurrence to metric availability.

## Authentication

All analytics endpoints require workspace-level authentication using bearer tokens. See the [Authentication Guide](/docs/implementation-technical/api/authentication) for details.

```typescript
headers: {
  'Authorization': 'Bearer {access_token}',
  'X-Workspace-ID': '{workspace_id}'
}
```

## API Endpoints Summary

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/api/v1/workspaces/{workspaceId}/campaigns/{campaignId}/analytics` | GET | Campaign performance metrics |
| `/api/v1/workspaces/{workspaceId}/analytics/overview` | GET | Workspace-wide analytics summary |
| `/api/v1/workspaces/{workspaceId}/analytics/deliverability` | GET | Deliverability and reputation metrics |
| `/api/v1/workspaces/{workspaceId}/analytics/engagement-heatmap` | GET | Engagement patterns by time |
| `/api/v1/workspaces/{workspaceId}/analytics/compare` | POST | Multi-campaign comparison |
| `/api/v1/workspaces/{workspaceId}/analytics/export` | POST | Export analytics data |
| `/api/v1/workspaces/{workspaceId}/analytics/exports/{exportId}` | GET | Check export status |

## Rate Limits

Analytics endpoints have specific rate limits to ensure system stability:

- **Campaign Analytics**: 100 requests per minute per workspace
- **Workspace Overview**: 60 requests per minute per workspace
- **Deliverability Analytics**: 30 requests per minute per workspace
- **Export Requests**: 10 requests per minute per workspace

## Related Documentation

- **[Campaign Analytics](campaign-analytics)** - Campaign and workspace analytics endpoints
- **[Deliverability & Engagement](deliverability-engagement)** - Deliverability and engagement heatmap endpoints
- **[Comparison & Exports](comparison-exports)** - Campaign comparison and export functionality
- **[Campaign Management API](/docs/implementation-technical/api/tenant-api/campaigns)** - Campaign CRUD operations
- **[Analytics Features](/docs/features/analytics/core-analytics/overview)** - Feature overview and capabilities
