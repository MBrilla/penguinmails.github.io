---
title: "KPI Definitions"
description: "Detailed formulas, targets, and measurement specifications for all KPIs"
last_modified_date: "2025-01-15"
level: "2"
persona: "Analytics Teams, Department Leads"
---

# KPI Definitions

Comprehensive definitions for all key performance indicators including formulas, targets, and measurement specifications.

## Growth & Acquisition KPIs

### Customer Acquisition Cost (CAC)

- **Formula:** Total marketing + sales spend / Number of new customers
- **Target:** <$XXX per customer
- **Measurement:** Monthly, rolling 12-month average
- **Breakdown:** Paid ads, content marketing, sales team, partnerships

### Customer Lifetime Value (LTV)

- **Formula:** Average revenue per user × customer lifespan
- **Target:** >$XXXX per customer
- **Measurement:** Rolling 12-month cohorts
- **Factors:** Subscription tier, retention rate, upsell revenue

### Monthly Recurring Revenue (MRR)

- **Formula:** Sum of all active subscriptions
- **Target:** $XXXK by end of year 1
- **Measurement:** End of month calculation
- **Components:** New subscriptions, expansions, churn

### Annual Recurring Revenue (ARR)

- **Formula:** MRR × 12
- **Target:** $XM by end of year 1
- **Measurement:** Annual calculation
- **Use Case:** Investor reporting, strategic planning

## Retention & Engagement KPIs

### Monthly Churn Rate

- **Formula:** Customers lost during month / Total customers at start of month
- **Target:** <5%
- **Measurement:** Monthly, segmented by cohort
- **Types:** Voluntary churn, involuntary churn, expansion churn

### Net Revenue Retention

- **Formula:** (Starting ARR + Expansion - Churn) / Starting ARR
- **Target:** >110%
- **Measurement:** Monthly cohorts
- **Components:** Upgrades, downgrades, churn

### Customer Satisfaction (NPS)

- **Formula:** % Promoters - % Detractors
- **Target:** >50
- **Measurement:** Monthly surveys
- **Segmentation:** By plan tier, usage level, support interactions

### Product Engagement Score

- **Formula:** Composite of daily active users, feature usage, session length
- **Target:** >75/100
- **Measurement:** Daily, weekly aggregation
- **Components:** Login frequency, feature usage depth, time spent

## Product Performance KPIs

### Time to First Value

- **Definition:** Time from signup to first successful email send
- **Target:** <30 minutes
- **Measurement:** Automated tracking in onboarding flow
- **Breakdown:** By user segment, device type, referral source

### Feature Adoption Rate

- **Formula:** Users using feature / Total eligible users
- **Target:** >60% for core features
- **Measurement:** Weekly, by feature category
- **Categories:** Campaign creation, IP management, analytics

### Support Ticket Resolution Time

- **Formula:** Average time from ticket creation to resolution
- **Target:** <4 hours for critical, <24 hours for normal
- **Measurement:** Daily, by priority level
- **Breakdown:** By issue type, support channel, time of day

## Email Performance KPIs

### Deliverability Rate

- **Formula:** Emails delivered / Emails sent
- **Target:** >98%
- **Measurement:** Real-time, per campaign
- **Factors:** IP reputation, content quality, recipient engagement

### Inbox Placement Rate

- **Formula:** Emails in inbox / Emails delivered
- **Target:** >95%
- **Measurement:** Third-party monitoring services
- **Breakdown:** By ISP, domain, sending volume

### Bounce Rate

- **Formula:** Hard bounces / Total sends
- **Target:** <2%
- **Measurement:** Real-time, automated alerts at 3%
- **Types:** Hard bounces, soft bounces, spam complaints

### Sender Reputation Score

- **Formula:** Composite score from multiple reputation providers
- **Target:** >85/100
- **Measurement:** Daily, per IP/domain
- **Providers:** Google Postmaster, Microsoft SNDS, etc.

## Data Collection Framework

### Data Sources

- **Product Analytics:** PostHog events, user behavior tracking
- **Business Intelligence:** Stripe payments, subscription data
- **Customer Success:** Support tickets, NPS surveys
- **External Monitoring:** Email reputation services, uptime monitors

### Analysis Framework

- **Cohort Analysis:** User behavior by signup month
- **Funnel Analysis:** Conversion rates through user journey
- **Attribution Modeling:** Revenue attribution to marketing channels
- **Predictive Modeling:** Churn prediction, LTV forecasting

---

**Related Documents:**
- [Metrics Overview](overview.md)
- [Dashboards and Alerts](dashboards-alerts.md)
- [Performance Monitoring](performance-monitoring.md)
