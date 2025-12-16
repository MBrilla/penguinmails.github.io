---
title: "Expansion Metrics"
description: "Revenue growth, product adoption, market expansion, and competitive intelligence"
last_modified_date: "2025-11-19"
level: "3"
persona: "Growth Managers, Product Managers"
---

# Expansion Metrics

Revenue growth, product adoption, market expansion, and competitive intelligence metrics for PenguinMails.

## Revenue Growth

- **Net Revenue Retention**: Revenue retained from existing customers
- **Net Revenue Expansion**: Revenue growth from existing customer base
- **Wallet Share**: Percentage of customer's email budget captured
- **Land and Expand**: Revenue growth through account expansion

## Product Adoption

```typescript
interface FeatureAdoption {
  feature: string;
  totalUsers: number;
  activeUsers: number;
  adoptionRate: number;
  timeToAdopt: number;      // Days from signup to first use
  usageFrequency: number;   // Times used per month
  upgradeCorrelation: number; // Correlation with plan upgrades
}

// Track feature adoption over time
const trackFeatureAdoption = (feature: string, userBase: User[]) => {
  const activeUsers = userBase.filter(user =>
    user.featuresUsed.includes(feature) &&
    user.lastUsed[feature] > new Date(Date.now() - 30 * 24 * 60 * 60 * 1000)
  );

  return {
    feature,
    totalUsers: userBase.length,
    activeUsers: activeUsers.length,
    adoptionRate: (activeUsers.length / userBase.length) * 100,
    averageTimeToAdopt: calculateAverageTimeToAdopt(activeUsers, feature),
    averageUsage: calculateAverageUsage(activeUsers, feature)
  };
};
```

## Account Expansion

- **Upsell Rate**: Percentage of customers upgrading plans
- **Cross-sell Rate**: Percentage adding complementary products
- **Expansion Revenue**: Additional revenue from existing customers
- **Account Growth Rate**: Revenue growth per existing account

## Market Penetration

- **Serviceable Addressable Market (SAM)**: Portion of market we can actually serve
- **Serviceable Obtainable Market (SOM)**: Realistic market share target
- **Market Share**: Current percentage of target market captured
- **Share of Wallet**: Percentage of customer spend captured

## Geographic Expansion

```typescript
interface GeographicMetrics {
  region: string;
  users: number;
  revenue: number;
  growthRate: number;
  marketPotential: number;
  penetrationRate: number;
}

// Calculate market penetration
const calculateMarketPenetration = (metrics: GeographicMetrics) => {
  return {
    ...metrics,
    penetrationRate: (metrics.users / metrics.marketPotential) * 100,
    revenuePerUser: metrics.revenue / metrics.users,
    growthEfficiency: metrics.growthRate / metrics.penetrationRate
  };
};
```

## Vertical Expansion

- **Industry Distribution**: User base across different industries
- **Vertical-Specific Metrics**: Performance in key target industries
- **Industry Growth Rates**: Expansion velocity by market segment
- **Vertical Penetration**: Market share within specific industries

## Competitive Intelligence

### Competitive Benchmarking

- **Market Share Analysis**: Position relative to competitors
- **Feature Parity Assessment**: Comparison of product capabilities
- **Pricing Competitiveness**: Position relative to competitor pricing
- **Customer Satisfaction Comparison**: NPS and satisfaction scores

### Competitive Growth Tracking

- **Competitor Funding Announcements**: Investment activity monitoring
- **Product Launch Tracking**: New feature and product announcements
- **Partnership and Acquisition Activity**: Strategic moves tracking
- **Customer Win/Loss Analysis**: Competitive displacement tracking

### Market Intelligence

- **Industry Trends**: Broader market movement analysis
- **Technology Adoption**: New technology implementation tracking
- **Regulatory Changes**: Compliance requirement monitoring
- **Economic Indicators**: Business climate assessment

## Related Documentation

- [Growth Metrics Overview](/docs/operations/analytics/analytics-performance/growth-metrics/overview) - Main page
- [Acquisition Metrics](/docs/operations/analytics/analytics-performance/growth-metrics/acquisition) - Funnel and channels
- [Growth Efficiency](/docs/operations/analytics/analytics-performance/growth-metrics/efficiency) - Unit economics
