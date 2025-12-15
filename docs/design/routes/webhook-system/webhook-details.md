---
title: "Webhook Details Route"
description: "Route specification for the webhook configuration and monitoring page"
last_modified_date: "2025-01-20"
level: 3
persona: Developers
keywords: webhooks, details, configuration, monitoring, health
---

# Webhook Details Route

**Route:** `/dashboard/settings/webhooks/:id`

**Access:** Admin, Developer

**User Story:** *"As a developer, I want to view webhook configuration and delivery history, so I can monitor and troubleshoot my integration."*

## Webhook Header

**Webhook Name**: "CRM Contact Sync"
- Status Badge: Active (Green) or Failing (Red)
- Actions: Edit, Test, Pause/Resume, Delete

**Endpoint**: `https://yourapp.com/webhooks/penguinmails`
- Copy Button: Copy URL to clipboard

**Created**: "November 1, 2025 by john@company.com"

## Configuration Summary

**Event Subscriptions (Expandable):**
- email.sent
- email.delivered
- email.opened
- email.clicked
- email.bounced

**"Edit Events" Link**: Navigate to edit page

**Event Filters (if configured):**
```json
{
  "workspaces": ["Client A", "Client B"],
  "contact.tags": ["vip"]
}
```

**"Edit Filters" Link**: Navigate to edit page

**Security Settings:**
- Secret Key: `whsec_...xyz` (masked, "Show" toggle requires re-auth)
- "Rotate Secret" Button: Generate new secret, revoke old one
- Retry Attempts: "5 attempts with exponential backoff"
- Timeout: "5 seconds"

## Health & Performance

**Delivery Stats (Last 24 Hours):**
- Total Deliveries: "1,234"
- Successful: "1,223 (99.1%)"
- Failed: "11 (0.9%)"
- Average Response Time: "145ms"

**Success Rate Chart (Line Chart):**
- Shows success rate over time (last 7 days)
- Y-Axis: Success percentage (0-100%)
- X-Axis: Time

**Response Time Chart (Line Chart):**
- Shows average response time over time
- Warning Line: 1000ms (yellow)
- Critical Line: 3000ms (red)

## Recent Deliveries

**Delivery Log Table (Last 50):**

| Timestamp | Event Type | Status | Response Time | Actions |
|-----------|------------|--------|---------------|---------|
| 2:34 PM | email.opened | ✓ 200 OK | 142ms | View |
| 2:33 PM | email.clicked | ✓ 200 OK | 156ms | View |
| 2:30 PM | email.bounced | ✗ 500 Error | 5000ms | Retry |

**Filters:**
- Status: All, Success, Failed, Pending Retry
- Event Type: Dropdown filter
- Date Range: Date picker

**"View Full Log" Link**: Navigate to detailed delivery log

## Quick Actions

**Test Webhook:**
- "Send Test Event" Button: Opens test modal

**Pause Webhook:**
- "Pause Webhook" Button: Stop sending events temporarily

**Rotate Secret:**
- "Rotate Secret Key" Button: Generate new secret (requires confirmation)

**Delete Webhook:**
- "Delete Webhook" Button: Permanently remove webhook (requires confirmation)

## User Journey Context

Regular monitoring and troubleshooting.

## Related Documentation

- [Webhook Monitoring](/docs/features/integrations/webhook-system#webhook-debugging)
- [Troubleshooting Guide](/docs/features/integrations/webhook-system#retry-logic--error-handling)
