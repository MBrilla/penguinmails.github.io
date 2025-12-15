---
title: "Delivery Log Route"
description: "Route specification for the detailed webhook delivery history interface"
last_modified_date: "2025-01-20"
level: 3
persona: Developers
keywords: webhooks, delivery log, history, debugging, retry
---

# Delivery Log Route

**Route:** `/dashboard/settings/webhooks/:id/deliveries`

**Access:** Admin, Developer

**User Story:** *"As a developer, I want to see detailed webhook delivery logs, so I can debug integration issues."*

## Delivery Log Filters

**Filter Bar:**
- Status: All, Success, Failed, Pending Retry
- Event Type: Multi-select dropdown
- Date Range: Date picker (default: Last 7 days)
- Search: Search by event ID or contact email

**Export:**
- "Export CSV" Button: Download filtered logs

## Delivery Log Table

**Detailed View:**

| Timestamp | Event ID | Event Type | Status | Response Time | Attempt | Actions |
|-----------|----------|------------|--------|---------------|---------|---------|
| Nov 25, 2:34 PM | evt_abc123 | email.opened | ✓ 200 OK | 142ms | 1/1 | View |
| Nov 25, 2:33 PM | evt_def456 | email.clicked | ✓ 200 OK | 156ms | 1/1 | View |
| Nov 25, 2:30 PM | evt_ghi789 | email.bounced | ✗ 500 Error | 5000ms | 2/5 | Retry, View |

## Expandable Row Details

Click row to expand and view request/response details.

**Request Details:**
```http
POST https://yourapp.com/webhooks/penguinmails
Content-Type: application/json
X-PenguinMails-Signature: sha256=abc123...
X-PenguinMails-Event: email.opened
X-PenguinMails-Delivery-ID: del_xyz789

{
  "id": "evt_abc123",
  "type": "email.opened",
  "created_at": "2025-11-25T14:34:00Z",
  "data": {
    "email_id": "msg_xyz789",
    "contact": {
      "email": "user@example.com"
    }
  }
}
```

**Response Details:**
```http
200 OK
Content-Type: text/plain
Response Time: 142ms

OK
```

**Actions:**
- "Copy Request" Button: Copy request payload
- "Copy Response" Button: Copy response body
- "Retry Now" Button: Manually retry failed delivery
- "View Event" Link: Navigate to event details

## Failed Deliveries Section

**Failed Deliveries Summary:**
- Total Failed: "11 in last 24 hours"
- Most Common Error: "500 Internal Server Error (7 occurrences)"
- Retry Status: "3 pending retry, 8 exhausted retries"

**Failed Deliveries Table:**

| Timestamp | Event Type | Error | Attempts | Next Retry | Actions |
|-----------|------------|-------|----------|------------|---------|
| 2:30 PM | email.bounced | 500 Error | 2/5 | In 3 minutes | Retry |
| 2:25 PM | email.opened | Timeout | 5/5 | Exhausted | View |

**Bulk Actions:**
- "Retry All Failed" Button: Retry all failed deliveries
- "Clear Failed Log" Button: Archive failed deliveries

## User Journey Context

Troubleshooting delivery issues, monitoring webhook health.

## Related Documentation

- [Retry Logic](/docs/features/integrations/webhook-system#retry-logic--error-handling)
- [Error Handling](/docs/features/integrations/webhook-system#webhook-debugging)
