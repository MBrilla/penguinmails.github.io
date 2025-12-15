---
title: "ESP Analytics Route"
description: "Route specification for ESP performance analytics dashboard"
last_modified_date: "2025-11-25"
level: "3"
persona: "Developers, Operations Teams"
---

# ESP Analytics Route

## `/dashboard/settings/integrations/esp/analytics` - ESP Analytics

**User Story**: *"As an admin, I want to compare ESP performance, so I can optimize my email routing strategy."*

## Provider Comparison Dashboard

**Time Range Selector**:
- Buttons: Last 7 Days, Last 30 Days, Last 90 Days, Custom Range

**Metrics Cards** (Top Row):
- **Total Emails Sent**: "156,789"
- **Average Delivery Rate**: "97.8%"
- **Average Open Rate**: "38.5%"
- **Average Click Rate**: "9.2%"

## Deliverability Comparison

**Table View**:

| Provider | Sent | Delivered | Bounced | Opened | Clicked | Delivery Rate |
|----------|------|-----------|---------|--------|---------|---------------|
| Postmark | 5,234 | 5,192 | 42 (0.8%) | 2,352 (45.3%) | 634 (12.1%) | 99.2% |
| Mailgun | 95,123 | 93,678 | 1,445 (1.5%) | 35,789 (38.2%) | 9,234 (9.7%) | 98.5% |
| Built-in | 56,432 | 53,098 | 3,334 (5.9%) | 16,726 (31.5%) | 3,876 (7.3%) | 94.1% |

**Chart View**:
- **Line Chart**: Delivery rate over time for each provider
- **Bar Chart**: Sent volume by provider
- **Pie Chart**: Email type distribution

## Performance Insights

**Recommendations Section**:

- ✅ **Postmark**: "Excellent for transactional emails. 99.2% delivery rate."
- ⚠️ **Mailgun**: "Good performance. Consider warming up new domains."
- ⚠️ **Built-in SMTP**: "Lower delivery rate. Review IP reputation and warmup strategy."

**Cost Analysis**:

```text
Cost per Delivered Email:

- Postmark: $0.00126 (5,192 delivered)
- Mailgun: $0.00059 (93,678 delivered)
- Built-in: $0.00000 (53,098 delivered)

Total Cost This Month: $61.25
Projected Annual Cost: $735.00
```

## Bounce Analysis

**Bounce Reasons** (Pie Chart):

- Hard Bounce (User Unknown): 45%
- Soft Bounce (Mailbox Full): 30%
- Spam Block: 15%
- Invalid Domain: 10%

**Top Bouncing Domains**:

1. `example.com` - 234 bounces
2. `test.org` - 156 bounces
3. `invalid.net` - 89 bounces

**"Export Bounce List" Button**: Download CSV of bounced emails.

## Engagement Metrics

**Open Rate by Provider** (Bar Chart):

- Postmark: 45.3%
- Mailgun: 38.2%
- Built-in: 31.5%

**Click Rate by Provider** (Bar Chart):

- Postmark: 12.1%
- Mailgun: 9.7%
- Built-in: 7.3%

**Time to Open** (Line Chart):
- Shows average time from send to first open for each provider

## User Journey Context

Regular monitoring (weekly/monthly) to optimize ESP usage and routing.

## Technical Integration

- **Data Source**: Aggregated from webhook events and ESP APIs
- **Caching**: Metrics cached for 15 minutes
- **Export**: Generate CSV/PDF reports

## Related Documentation

- [Analytics Overview](/docs/features/analytics/core-analytics/overview)
- [Deliverability Monitoring](/docs/features/warmup/monitoring/overview)
