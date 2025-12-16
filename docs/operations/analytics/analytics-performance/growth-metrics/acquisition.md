---
title: "Acquisition Metrics"
description: "Top-of-funnel metrics, conversion funnels, and channel performance tracking"
last_modified_date: "2025-11-19"
level: "3"
persona: "Growth Managers, Marketing Analysts"
---

# Acquisition Metrics

Top-of-funnel metrics, conversion funnels, and channel performance tracking for PenguinMails growth.

## Top-of-Funnel Metrics

- **Website Visitors**: Total unique visitors to marketing pages
- **Marketing Qualified Leads (MQLs)**: Prospects showing buying intent
- **Sales Qualified Leads (SQLs)**: Leads ready for sales engagement
- **Trial Signups**: Users who start free trial or demo

## Conversion Metrics

```typescript
interface ConversionFunnel {
  visitors: number;
  signups: number;
  activations: number;      // First campaign sent
  conversions: number;      // Become paying customers

  rates: {
    signupRate: number;     // signups / visitors
    activationRate: number; // activations / signups
    conversionRate: number; // conversions / activations
    overallRate: number;    // conversions / visitors
  };
}

// Calculate conversion rates
const calculateConversionRates = (funnel: ConversionFunnel) => {
  return {
    signupRate: (funnel.signups / funnel.visitors) * 100,
    activationRate: (funnel.activations / funnel.signups) * 100,
    conversionRate: (funnel.conversions / funnel.activations) * 100,
    overallRate: (funnel.conversions / funnel.visitors) * 100
  };
};
```

## Channel Performance

- **Cost Per Acquisition (CPA)**: Marketing spend divided by new customers
- **Customer Acquisition Cost (CAC)**: Total acquisition costs per customer
- **Payback Period**: Time to recover CAC through customer revenue
- **Return on Ad Spend (ROAS)**: Revenue generated per dollar spent on advertising

## Channel Performance Dashboard

```markdown
Channel Overview
├── Organic Search: X users ($X CAC)
├── Paid Search: X users ($X CAC)
├── Content Marketing: X users ($X CAC)
├── Social Media: X users ($X CAC)
└── Direct: X users ($X CAC)

Channel Efficiency
├── Best Performing: [Channel] (X% of users)
├── Most Efficient: [Channel] ($X CAC)
├── Highest Volume: [Channel] (X users)
└── Best ROI: [Channel] (X:1 ratio)
```

## Content Performance

```typescript
interface ContentMetrics {
  contentType: string;
  impressions: number;
  engagements: number;
  conversions: number;
  cost: number;

  performance: {
    engagementRate: number;    // engagements / impressions
    conversionRate: number;    // conversions / engagements
    costPerEngagement: number; // cost / engagements
    costPerConversion: number; // cost / conversions
    roi: number;              // revenue / cost
  };
}

// Evaluate content effectiveness
const evaluateContentPerformance = (content: ContentMetrics) => {
  return {
    ...content,
    performance: {
      engagementRate: (content.engagements / content.impressions) * 100,
      conversionRate: (content.conversions / content.engagements) * 100,
      costPerEngagement: content.cost / content.engagements,
      costPerConversion: content.cost / content.conversions,
      roi: (content.conversions * 50) / content.cost // Assuming $50 avg deal size
    }
  };
};
```

## Paid Advertising Optimization

- **Cost Per Click (CPC)**: Average cost for ad clicks
- **Click-Through Rate (CTR)**: Percentage of impressions that become clicks
- **Conversion Rate**: Percentage of clicks that become customers
- **Return on Ad Spend (ROAS)**: Revenue generated per ad dollar spent

## Leading Indicators

- **Trial-to-Paid Conversion Trends**: Early signal of acquisition quality
- **Product Qualified Leads (PQLs)**: Users demonstrating high product engagement
- **Support Ticket Reduction**: Indicator of improved product experience
- **Feature Request Volume**: Signal of product-market fit strength

## Related Documentation

- [Growth Metrics Overview](/docs/operations/analytics/analytics-performance/growth-metrics/overview) - Main page
- [Expansion Metrics](/docs/operations/analytics/analytics-performance/growth-metrics/expansion) - Revenue and market expansion
- [Growth Efficiency](/docs/operations/analytics/analytics-performance/growth-metrics/efficiency) - Unit economics
