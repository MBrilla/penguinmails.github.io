---
title: "Manual Reporting & Data Export"
description: "Scheduled reports, data export capabilities, and integration with external analytics tools"
level: "1"
status: "ACTIVE"
roadmap_timeline: "Q4 2025"
priority: "High"
related_features:
  - analytics/core-analytics/overview
  - analytics/enhanced-analytics/overview
related_tasks:
  - epic-1-project-foundation
---

# Manual Reporting & Data Export

**Quick Access**: Export email performance data and generate scheduled reports for external analysis.

## Overview

Manual Reporting provides essential data export and report generation capabilities for users who need to analyze email performance in external tools or share results with stakeholders. This Level 1 feature complements the directional analytics dashboard with flexible export options.

### Key Capabilities

- **Scheduled Reports**: Weekly/monthly automated reports via email
- **Data Export**: CSV, Excel, PDF formats for all metrics
- **External Tool Integration**: Export to Google Sheets, Excel, BI tools
- **Custom Date Ranges**: Flexible reporting periods
- **Multi-Tenant Support**: Workspace-level and tenant-level reports

---

## Documentation Structure

This documentation is organized into focused topic guides:

### [Quick Start Guide](./quick-start-guide)

Get started with manual reporting:

- Generate your first report
- Configure report types and date ranges
- Choose metrics and export formats
- Schedule automated reports
- Quick export from dashboard views

### [Advanced Reporting](./advanced-reporting)

In-depth coverage of advanced features:

- Scheduled report configuration (weekly/monthly)
- Custom data exports (campaign-level, lead-level)
- Integration with external tools (Google Sheets, Excel, BI tools)
- Custom report templates

### [Technical Implementation](./technical-implementation)

Technical details for developers:

- Database schema (scheduled_reports, report_generations, export_jobs)
- Report generation service
- Scheduled report runner (cron jobs and queue workers)
- API endpoints for programmatic exports

---

## Report Types

| Report Type | Description | Best For |
|-------------|-------------|----------|
| Campaign Performance | Metrics for specific campaigns | Campaign managers |
| Overall Performance | Aggregated metrics across all campaigns | Executive overview |
| Deliverability Report | Bounce rates, spam complaints, unsubscribes | Deliverability specialists |
| Workspace Summary | Per-workspace performance breakdown | Team leads |
| Domain Health | Per-domain reputation and metrics | Technical teams |

## Export Formats

| Format | Use Case | Features |
|--------|----------|----------|
| CSV | Universal compatibility | All tools support, simple structure |
| Excel (.xlsx) | Formatted reports | Charts, formatting, multiple sheets |
| PDF | Stakeholder sharing | Styled, print-ready, charts included |
| JSON | API integrations | Programmatic access, custom tools |

---

## Related Documentation

### Feature Completeness Review

- **[Analytics & Reporting Gap Analysis](/.kiro/specs/feature-completeness-review/findings/analytics-reporting)** - Comprehensive review of analytics features and roadmap
- **[Third-Party Dependencies](/docs/features/analytics/third-party-dependencies)** - External services and integrations

### Analytics

- **[Core Analytics](/docs/features/analytics/core-analytics/overview)** - Dashboard and metrics
- **[Enhanced Analytics](/docs/features/analytics/enhanced-analytics/overview)** - Advanced analytics (Q1 2026)

### Integration

- **[API Access](/docs/features/integrations/api-access)** - Programmatic data export
- **[CRM Integration](/docs/features/integrations/crm-integration/overview)** - External tool sync

### Technical

- **[API Architecture](/docs/implementation-technical/api/README)** - Export API endpoints

---

**Last Updated:** December 15, 2025
**Status:** Active - Core Feature (Level 1)
**Owner:** Analytics Team
