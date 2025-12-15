---
title: "Advanced Reporting"
description: "Scheduled reports, custom data exports, and external tool integration"
last_modified_date: "2025-12-15"
level: "2"
status: "ACTIVE"
parent: "manual-reporting"
---

# Advanced Reporting

Advanced configuration options for scheduled reports, custom data exports, and external tool integration.

## Scheduled Report Configuration

### Weekly Performance Report

**Configuration:**

```yaml
name: "Weekly Campaign Performance"
frequency: weekly
day_of_week: monday
time: "09:00"
timezone: "America/New_York"

recipients:
  - team@company.com
  - manager@company.com

format: pdf
include:
  - summary_metrics
  - top_campaigns
  - deliverability_trends
  - weekly_comparison
```

**Delivered Report Includes:**

- Executive summary with key metrics
- Week-over-week comparison
- Top/bottom performing campaigns
- Deliverability trends chart
- Action items and recommendations

### Monthly Executive Report

**Configuration:**

```yaml
name: "Monthly Executive Summary"
frequency: monthly
day_of_month: 1
time: "08:00"

recipients:
  - executives@company.com

format: pdf_executive
include:
  - executive_summary
  - growth_metrics
  - workspace_breakdown
  - roi_analysis
  - month_over_month_trends
```

**Delivered Report Includes:**

- Month-over-month growth
- Per-workspace performance
- ROI metrics (emails sent vs opportunities created)
- Strategic recommendations
- Next month priorities

---

## Custom Data Exports

### Campaign-Level Export

**Available Fields:**

```typescript
interface CampaignExport {
  // Campaign info
  campaignId: string;
  campaignName: string;
  createdAt: Date;
  launchedAt: Date;
  status: string;

  // Sending metrics
  totalEmails: number;
  deliveredEmails: number;
  bouncedEmails: number;
  deliveryRate: number;

  // Engagement (directional)
  opens: number;
  openRate: number;
  clicks: number;
  clickRate: number;
  clickToOpenRate: number;

  // Negative metrics
  spamComplaints: number;
  spamRate: number;
  unsubscribes: number;
  unsubscribeRate: number;

  // Timing
  avgTimeToOpen: number;  // minutes
  avgTimeToClick: number; // minutes

  // Metadata
  workspaceName: string;
  domainUsed: string;
  emailsSentPerDay: number;
}
```

**Export Process:**

1. Select campaigns (individual or bulk)
2. Choose fields to include
3. Select format (CSV, Excel)
4. Click "Export"

### Lead-Level Export

**Available Fields:**

```typescript
interface LeadExport {
  // Lead info
  leadId: string;
  email: string;
  firstName: string;
  lastName: string;
  company: string;

  // Custom fields
  customFields: Record<string, any>;

  // Engagement
  emailsSent: number;
  emailsDelivered: number;
  emailsOpened: number;
  emailsClicked: number;
  lastOpenedAt: Date;
  lastClickedAt: Date;

  // Status
  status: string;
  leadScore: number;
  tags: string[];

  // Metadata
  createdAt: Date;
  updatedAt: Date;
  source: string;
}
```

---

## Integration with External Tools

### Google Sheets Integration

**Setup:**

1. Go to Settings → Integrations → Google Sheets
2. Click "Connect Google Account"
3. Authorize PenguinMails
4. Create auto-export rule

**Auto-Export Configuration:**

```yaml
destination:
  type: google_sheets
  spreadsheet_id: "abc123..."
  sheet_name: "Campaign Performance"

schedule:
  frequency: daily
  time: "06:00"

data:
  export_type: campaign_metrics
  date_range: yesterday
  append_mode: true  # Add new rows vs overwrite
```

**Result:** Daily campaign metrics automatically appended to Google Sheet.

### Excel / BI Tool Integration

**Export Formats:**

- **CSV**: Universal format, compatible with all tools
- **Excel (.xlsx)**: Formatted spreadsheets with charts
- **JSON**: For custom integrations and APIs
- **Parquet**: For big data tools (future)

**API-Based Export:**

```typescript
// Programmatic data export via API
const response = await fetch('/api/exports/create', {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer YOUR_API_KEY',
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    exportType: 'campaign_metrics',
    dateRange: {
      from: '2025-10-01',
      to: '2025-10-31',
    },
    format: 'csv',
    fields: ['campaignName', 'totalEmails', 'openRate', 'clickRate'],
  }),
});

const { exportId, downloadUrl } = await response.json();
```

---

## Report Customization

### Custom Report Templates

Create reusable report templates:

**Template Configuration:**

```yaml
name: "Client Monthly Report"
description: "Monthly performance report for client review"

sections:
  - type: summary_metrics
    metrics: [sent, delivered, opened, clicked]

  - type: chart
    chart_type: line
    metric: delivery_rate
    title: "Delivery Rate Trend"

  - type: table
    title: "Top 10 Campaigns"
    columns: [name, sent, delivered, opens, clicks]
    sort_by: clicks
    limit: 10

  - type: text
    content: |
      ## Executive Summary
      This month showed strong performance...

  - type: recommendations
    auto_generated: true

styling:
  logo: "https://your-logo-url.com/logo.png"
  primary_color: "#007bff"
  font: "Arial"
```

**Using Templates:**

1. Create template once
2. Generate report from template → Select date range
3. Report generated with latest data

---

## Next Steps

- **[Quick Start Guide](./quick-start-guide)** - Basic report generation and export
- **[Technical Implementation](./technical-implementation)** - Database schema, services, and API endpoints

---

**Last Updated:** December 15, 2025
**Parent:** [Manual Reporting & Data Export](./README)
