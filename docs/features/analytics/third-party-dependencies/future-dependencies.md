---
title: "Future Analytics Dependencies"
description: "Planned dependencies and investigation spikes for analytics enhancements"
last_modified_date: "2025-01-15"
level: "3"
persona: "Technical Teams, Product Teams"
---

# Future Analytics Dependencies

Planned dependencies and investigation spikes for analytics system enhancements.

## Planned Dependencies

### Gemini AI - ML/AI Framework (Q1 2026)

**Purpose:** Predictive analytics models and AI-powered insights

### Usage

- Send time optimization (per-contact predictions)
- Subject line performance prediction
- Deliverability forecasting
- Churn prediction and retention recommendations
- Campaign optimization suggestions

**Cost:** Usage-based API pricing

**Target:** Q1 2026 (Enhanced Analytics milestone)

### Why Gemini AI

API-based approach eliminates need for self-hosted ML infrastructure (TensorFlow/PyTorch), reducing operational complexity and upfront costs.

**Alternative:** TensorFlow/PyTorch for self-hosted ML models if API costs become prohibitive at scale.

### Dependencies

- 30+ days of historical campaign data
- Prompt engineering framework
- Historical data export pipeline
- API error handling and fallback logic

## Future Considerations (Investigation Spikes)

### Large-Scale Data Processing Solutions (Q2 2026)

**Description:** Investigate solutions for large-scale analytics data processing as platform scales

**Current Approach:** PostHog for big data analytics, OLAP database (PostgreSQL + TimescaleDB) for time-series analytics, manual database cleanup for operational data

### Investigation Triggers

- Analytics queries taking >5 seconds
- Database storage exceeding 500GB for analytics data
- Complex multi-step ETL requirements
- PostHog limitations for custom aggregations

**Target:** Q2 2026 (performance-driven)

**Approach:** Conduct spike to evaluate needs, then implement only if validated

**Effort:** Medium (2-3 weeks for spike + implementation)

### Enterprise Data Warehouse Integration (Q3 2026)

**Description:** Integration with enterprise data warehouses for real-time data sync

**Current Approach:** CSV/Excel/JSON export for external analysis

### Potential Integrations

- Snowflake
- BigQuery
- Redshift

### Investigation Triggers

- 3+ enterprise customers requesting data warehouse integration
- Real-time streaming requirements
- Compliance requirements for data residency

**Target:** Q3 2026 (customer-driven)

**Approach:** Customer validation required before implementation

**Effort:** Large (3-4 weeks)

## Data Export Integrations

### Google Sheets Integration

**Purpose:** Auto-export analytics data to Google Sheets

**Status:** Planned (manual reporting feature)

**Implementation:** OAuth-based connection with scheduled exports

Use cases:
- Automated daily/weekly data exports
- Client reporting for agencies
- Custom analysis in spreadsheets

### BI Tool Compatibility

**Purpose:** Export to external BI tools (Tableau, Power BI, Looker)

**Formats Supported:**
- CSV (universal compatibility)
- Excel (.xlsx) with formatting
- JSON (for custom integrations)
- Parquet (future - for big data tools)

**Status:** Supported via data export API

### CRM Integration (Q2 2026)

**Purpose:** Multi-touch attribution and conversion tracking

**Status:** Planned for Q2 2026 (customer-driven)

Use cases:
- Revenue attribution across customer journey
- Conversion event tracking
- Customer lifecycle analysis

Dependencies:
- Customer journey tracking system
- Revenue tracking infrastructure
- CRM API integration

## Future Cost Optimization

### Loop.so Replacement (Q3 2026)

| State | Monthly Cost |
|-------|--------------|
| Current (Loop.so) | $29 |
| After migration (in-house SMTP) | $0 |

**Additional Annual Savings:** $348/year

### PostHog Self-Hosting (Future)

Triggers for self-hosting:
- Event volume exceeds free tier (1M events/month)
- Data residency requirements
- Cost optimization at scale
- Custom feature requirements

Benefits:
- No usage-based costs
- Full data control
- Custom feature development
- No vendor lock-in

**Complexity:** Medium (1-2 weeks setup + ongoing maintenance)

---

**Related Documents:**
- [Overview](overview.md)
- [Current Dependencies](current-dependencies.md)
- [Enhanced Analytics Roadmap](/docs/features/analytics/enhanced-analytics/overview)
