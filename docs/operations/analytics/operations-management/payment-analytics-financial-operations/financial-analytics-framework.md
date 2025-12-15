---
title: "Financial Analytics Framework"
description: "Revenue metrics, cost structure analysis, profitability metrics, and cash flow management"
last_modified_date: "2025-12-15"
level: "3"
parent: "Payment Analytics & Financial Operations"
---

# Financial Analytics Framework

This guide covers comprehensive financial analytics including revenue tracking, cost analysis, profitability metrics, and cash flow management.

---

## Revenue Metrics

### Core Revenue Streams

- **Subscription Revenue**: Monthly/annual recurring revenue from platform plans
- **Add-on Revenue**: Dedicated IPs, additional domains, premium support
- **Professional Services**: Custom integrations, consulting, training
- **Marketplace Revenue**: Third-party integrations and templates

### Revenue Calculation Methods

```typescript
// Monthly Recurring Revenue
const calculateMRR = (subscriptions: Subscription[]) => {
  return subscriptions
    .filter(sub => sub.status === 'active')
    .reduce((total, sub) => {
      return total + (sub.plan.price * (sub.billingCycle === 'annual' ? 1));
    }, 0);
};

// Annual Recurring Revenue
const calculateARR = (mrr: number) => mrr * 12;

// Net Revenue Retention
const calculateNRR = (startingARR: number, endingARR: number, expansions: number) => {
  return ((endingARR + expansions) / startingARR) * 100;
};
```

### Key Revenue KPIs

| KPI | Description | Target |
|-----|-------------|--------|
| MRR Growth Rate | Month-over-month subscription revenue growth | >5% |
| ARR Growth Rate | Year-over-year annualized revenue growth | >50% |
| ARPU | Total revenue divided by active users | Varies by tier |
| LTV | Predicted revenue from customer relationship | >3x CAC |
| Payback Period | Time to recover customer acquisition costs | <12 months |

---

## Cost Structure Analysis

### Fixed Costs

- **Infrastructure Costs**: VPS hosting, database, CDN, monitoring
- **Software Licenses**: Development tools, third-party services
- **Insurance**: Business liability, cyber security, workers' compensation
- **Legal & Compliance**: Regulatory compliance, legal fees, audits

### Variable Costs

- **Payment Processing Fees**: Stripe Connect fees (2.9% + $0.30 per transaction)
- **Email Delivery Costs**: ESP fees based on volume ($0.0001-$0.001 per email)
- **Customer Acquisition**: Marketing spend, sales commissions
- **Customer Success**: Support staffing, training materials

### Cost Allocation

```typescript
interface CostBreakdown {
  infrastructure: number;    // 35% of total costs
  personnel: number;         // 45% of total costs
  marketing: number;         // 10% of total costs
  operations: number;        // 7% of total costs
  legal: number;             // 3% of total costs
}

const costBreakdown: CostBreakdown = {
  infrastructure: 0.35,
  personnel: 0.45,
  marketing: 0.10,
  operations: 0.07,
  legal: 0.03
};
```

---

## Profitability Metrics

### Unit Economics

- **Customer Acquisition Cost (CAC)**: Total marketing spend divided by new customers
- **Gross Margin**: Revenue minus cost of goods sold
- **Contribution Margin**: Revenue minus variable costs
- **Net Profit Margin**: Net income as percentage of revenue

### Break-even Analysis

```typescript
const calculateBreakEven = (
  fixedCosts: number,
  averageRevenuePerUser: number,
  averageVariableCostPerUser: number
) => {
  const contributionMarginPerUser = averageRevenuePerUser - averageVariableCostPerUser;
  return Math.ceil(fixedCosts / contributionMarginPerUser);
};
```

---

## Cash Flow Management

### Operating Cash Flow

- **Cash Inflows**: Subscription payments, one-time services, marketplace revenue
- **Cash Outflows**: Operating expenses, cost of sales, taxes
- **Working Capital**: Current assets minus current liabilities
- **Cash Conversion Cycle**: Time to convert investments to cash

### Cash Flow Forecasting

```typescript
interface CashFlowProjection {
  month: string;
  startingCash: number;
  inflows: {
    subscriptions: number;
    services: number;
    other: number;
  };
  outflows: {
    operations: number;
    marketing: number;
    capex: number;
  };
  netCashFlow: number;
  endingCash: number;
}
```

---

## Key Financial Formulas

| Metric | Formula |
|--------|---------|
| MRR | Sum of all active subscription prices |
| ARR | MRR × 12 |
| NRR | ((Ending ARR + Expansions) / Starting ARR) × 100 |
| CAC | Total Marketing Spend / New Customers |
| LTV | (ARPU × Gross Margin) / Churn Rate |
| Payback | CAC / (ARPU × Gross Margin) |

---

## Next Steps

- **[Churn Retention & Dashboard](./churn-retention-dashboard)** - Retention analytics and dashboards
- **[Payment & Billing Journey](./payment-billing-journey)** - User payment flows
- **[Edge Cases & Recovery](./edge-cases-recovery)** - Financial error handling

---

**Keywords**: revenue metrics, cost analysis, profitability, cash flow, MRR, ARR, unit economics, break-even
