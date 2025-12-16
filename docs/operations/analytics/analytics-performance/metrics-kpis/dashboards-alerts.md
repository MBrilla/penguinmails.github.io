---
title: "Dashboards and Alert Thresholds"
description: "KPI dashboards, alert thresholds, and monitoring configuration"
last_modified_date: "2025-01-15"
level: "2"
persona: "Operations Teams, Executives"
---

# Dashboards and Alert Thresholds

Dashboard views for different stakeholders and alert threshold configuration for proactive monitoring.

## KPI Dashboards

### Executive Dashboard

The executive dashboard provides high-level visibility into business health:

- **MRR Growth:** Month-over-month, year-over-year comparisons
- **Customer Metrics:** Acquisition, retention, LTV/CAC ratio
- **Product Health:** Uptime, critical bugs, support tickets
- **Financial Health:** Burn rate, runway, profitability

### Product Dashboard

Product teams monitor user experience and feature performance:

- **User Engagement:** DAU/MAU, session length, feature usage
- **Onboarding Success:** Completion rates, time to value
- **Email Performance:** Deliverability, reputation scores
- **Quality Metrics:** Bug rates, performance issues

### Growth Dashboard

Growth teams track acquisition and expansion metrics:

- **Acquisition Channels:** CAC by channel, conversion rates
- **Market Penetration:** Market share, competitive positioning
- **Expansion Revenue:** Upsell rates, plan migrations
- **Customer Success:** NPS, retention by cohort

### Dashboard Layers

Different dashboard views serve different purposes:

- **Executive Dashboard:** High-level business and system health metrics
- **Operations Dashboard:** Detailed system performance and alerting
- **Scaling Dashboard:** Traffic heatmap and infrastructure capacity planning
- **Development Dashboard:** Application performance and debugging
- **Customer Dashboard:** User-facing metrics and SLA tracking

> **Design Note:** For detailed visual specifications of these dashboards, refer to the [Unified Analytics Views & Experience Design](/docs/design/analytics-views) document.

## Alert Thresholds

### Critical Alerts (Immediate Action Required)

Critical alerts require immediate response within minutes:

| Alert Type | Threshold | Response Time |
|------------|-----------|---------------|
| System Downtime | >5 minutes | Immediate |
| Deliverability Drop | >5% below target | Immediate |
| Security Incident | Any confirmed breach | Immediate |
| Data Loss | Any production data loss | Immediate |

### Warning Alerts (Action Within 24 Hours)

Warning alerts require response within one business day:

| Alert Type | Threshold | Response Time |
|------------|-----------|---------------|
| MRR Decline | >10% month-over-month | 24 hours |
| Churn Spike | >15% above baseline | 24 hours |
| Support Backlog | >50 open tickets | 24 hours |
| Performance Degradation | >20% slower than baseline | 24 hours |

### Trend Alerts (Weekly Review)

Trend alerts are reviewed in weekly operations meetings:

- **Engagement Decline:** >10% drop in key metrics
- **Competition Changes:** New features from competitors
- **Market Shifts:** Changes in email deliverability landscape

## Alert Channels

Alerts are routed through multiple channels based on severity:

- **Slack:** Real-time alerts for on-call team
- **Email:** Escalation notifications for management
- **SMS:** Critical alerts for emergency response
- **Dashboard:** Visual indicators and historical tracking

## Monitoring Integration

### Infrastructure Monitoring

- **Hostwind VPS:** Server metrics and resource monitoring
- **Database Monitoring:** NileDB performance and query optimization
- **Network Monitoring:** Bandwidth usage and latency tracking
- **Container Monitoring:** Docker container health and resource usage

### Application Performance Monitoring (APM)

- **Error Tracking:** Sentry for real-time error monitoring (Future/2026)
- **Performance Profiling:** Application response time analysis
- **Memory Leak Detection:** Automated memory usage monitoring
- **Database Query Monitoring:** Slow query identification and optimization

### Business Intelligence

- **User Analytics:** PostHog for user behavior tracking
- **Revenue Analytics:** Stripe Connect dashboard integration
- **Email Analytics:** Send rate, deliverability, and engagement metrics
- **Custom Dashboards:** Tailored views for different user roles

---

**Related Documents:**
- [KPI Definitions](kpi-definitions.md)
- [Performance Monitoring](performance-monitoring.md)
- [Incident Response](incident-response.md)
