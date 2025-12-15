---
title: "Churn, Retention & Financial Dashboard"
description: "Churn and retention analysis, cohort tracking, financial dashboard views, and executive reporting"
last_modified_date: "2025-12-15"
level: "3"
parent: "Payment Analytics & Financial Operations"
---

# Churn, Retention & Financial Dashboard

This guide covers churn and retention analytics, cohort analysis, and financial dashboard visualization for executive reporting.

---

## Churn and Retention Analysis

### Churn Metrics

- **Monthly Churn Rate**: Percentage of customers lost in a month
- **Annual Churn Rate**: Percentage of customers lost in a year
- **Revenue Churn**: Dollar value of lost subscriptions
- **Gross Churn vs Net Churn**: Including vs excluding expansions

### Retention Calculations

```typescript
// Cohort Analysis
const calculateCohortRetention = (cohortData: Map<string, number[]>) => {
  const retentionRates: Map<string, number[]> = new Map();

  for (const [cohort, monthlyUsers] of cohortData) {
    const initialUsers = monthlyUsers[0];
    const retention = monthlyUsers.map(users => (users / initialUsers) * 100);
    retentionRates.set(cohort, retention);
  }

  return retentionRates;
};

// Customer Lifetime Value
const calculateLTV = (
  averageRevenuePerUser: number,
  grossMargin: number,
  churnRate: number
) => {
  const monthlyChurn = churnRate / 100;
  return (averageRevenuePerUser * grossMargin) / monthlyChurn;
};
```

### Retention Benchmarks

| Metric | Good | Great | Excellent |
|--------|------|-------|-----------|
| Monthly Churn | <5% | <3% | <2% |
| Net Revenue Retention | >100% | >110% | >120% |
| Logo Retention | >90% | >95% | >98% |
| LTV:CAC Ratio | >3:1 | >4:1 | >5:1 |

---

## Financial Dashboard

### Executive Summary

```markdown
Financial Overview
├── MRR: $[X] (↑X% MoM)
├── ARR: $[X] (↑X% YoY)
├── Net Revenue Retention: X%
├── Gross Margin: X%
└── Cash Runway: X months
```

### Revenue Analytics

```markdown
Revenue Breakdown
├── Subscriptions: X% of total
├── Add-ons: X% of total
├── Services: X% of total
└── Other: X% of total

Growth Trends
├── New Customers: X this month
├── Expansion Revenue: $X this month
├── Churned Revenue: $X this month
└── Net New MRR: $X this month
```

### Cost Analysis

```markdown
Cost Structure
├── Personnel: X% of total
├── Infrastructure: X% of total
├── Marketing: X% of total
└── Operations: X% of total

Cost Trends
├── Total Burn: $X this month
├── Burn Rate: $X per month
├── CAC: $X per customer
└── CAC Payback: X months
```

---

## Dashboard Widgets

### Key Performance Indicators

| Widget | Data Source | Refresh Rate |
|--------|-------------|--------------|
| MRR Trend | Subscriptions table | Real-time |
| Churn Rate | Customer events | Daily |
| Revenue by Plan | Plans + Subscriptions | Real-time |
| Cash Position | Finance system | Daily |
| CAC by Channel | Marketing analytics | Weekly |

### Cohort Visualization

```typescript
interface CohortData {
  cohortMonth: string;
  totalCustomers: number;
  retentionByMonth: number[];
  revenueByMonth: number[];
  averageLTV: number;
}

// Display format
const cohortTable = cohorts.map(cohort => ({
  month: cohort.cohortMonth,
  size: cohort.totalCustomers,
  m1: `${cohort.retentionByMonth[0]}%`,
  m3: `${cohort.retentionByMonth[2]}%`,
  m6: `${cohort.retentionByMonth[5]}%`,
  m12: `${cohort.retentionByMonth[11]}%`,
  ltv: `$${cohort.averageLTV}`
}));
```

---

## Executive Reporting

### Monthly Financial Report

1. **Revenue Summary**
   - Total MRR and change vs. prior month
   - Revenue breakdown by source
   - Top 10 accounts by revenue

2. **Customer Health**
   - Net customer adds/losses
   - Churn analysis and reasons
   - Expansion and contraction details

3. **Financial Position**
   - Cash balance and runway
   - Operating expenses trend
   - Key ratio changes

### Quarterly Business Review

1. **Strategic Metrics**
   - Progress against ARR targets
   - Market share estimates
   - Competitive positioning

2. **Operational Efficiency**
   - CAC payback trends
   - Gross margin evolution
   - Unit economics health

3. **Forward Outlook**
   - Revenue projections
   - Investment priorities
   - Risk factors

---

## Alert Thresholds

| Metric | Warning | Critical |
|--------|---------|----------|
| Monthly Churn | >4% | >6% |
| NRR | <105% | <100% |
| Cash Runway | <9 months | <6 months |
| CAC Payback | >15 months | >18 months |
| Gross Margin | <65% | <60% |

---

## Next Steps

- **[Financial Analytics Framework](./financial-analytics-framework)** - Core financial metrics
- **[Payment & Billing Journey](./payment-billing-journey)** - Revenue collection flows
- **[Edge Cases & Recovery](./edge-cases-recovery)** - Churn prevention strategies

---

**Keywords**: churn analysis, retention metrics, cohort analysis, financial dashboard, executive reporting, LTV, CAC
