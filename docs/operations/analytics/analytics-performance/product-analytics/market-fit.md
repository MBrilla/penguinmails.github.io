---
title: "Product-Market Fit"
description: "Product-market fit analysis and feature lifecycle management"
last_modified_date: "2025-01-15"
level: "2"
persona: "Product Managers, Strategy Teams"
---

# Product-Market Fit

Product-market fit analysis, feature lifecycle management, and success criteria.

## Product-Market Fit Analysis

### Fit Indicators

- **Usage Intensity**: How deeply users engage with core features
- **Retention by Cohort**: How different user groups retain over time
- **Referral Rate**: Willingness to recommend the product
- **Competitive Advantage**: Perceived differentiation from alternatives

### Fit Measurement

```typescript
interface ProductMarketFit {
  metrics: {
    retentionRate: number;          // 30-day retention > 50%
    referralRate: number;           // Net Promoter Score > 30
    usageIntensity: number;         // Daily active users > 40%
    competitiveMoat: number;        // Unique value proposition strength
  };
  segments: {
    champions: UserSegment;         // Highly satisfied users
    neutrals: UserSegment;          // Satisfied but not enthusiastic
    detractors: UserSegment;        // Dissatisfied users
  };
  fit: {
    score: number;                  // Composite fit score 0-100
    confidence: 'weak' | 'moderate' | 'strong';
    recommendations: string[];
  };
}

const calculateProductMarketFit = (data: ProductMarketFit): number => {
  const weights = {
    retentionRate: 0.3,
    referralRate: 0.25,
    usageIntensity: 0.25,
    competitiveMoat: 0.2
  };

  return Object.entries(weights).reduce((score, [key, weight]) => {
    const value = data.metrics[key as keyof typeof data.metrics];
    return score + (value * weight);
  }, 0);
};
```

### Market Feedback Integration

- **User Surveys**: Regular feedback collection and analysis
- **Support Ticket Analysis**: Common pain points and feature requests
- **Usage Pattern Analysis**: Behavioral indicators of satisfaction
- **Competitive Analysis**: Positioning relative to market alternatives

## Feature Lifecycle Management

### Feature Development Pipeline

```typescript
interface FeatureLifecycle {
  stage: 'ideation' | 'design' | 'development' | 'testing' | 'launch' | 'maintenance' | 'sunset';
  metrics: {
    developmentTime: number;
    adoptionRate: number;
    satisfactionScore: number;
    maintenanceCost: number;
    revenueImpact: number;
  };
  stakeholders: {
    product: string[];
    engineering: string[];
    design: string[];
    marketing: string[];
  };
  timeline: {
    planned: Date;
    actual: Date;
    milestones: Milestone[];
  };
}

interface Milestone {
  name: string;
  date: Date;
  completed: boolean;
  metrics: Record<string, number>;
}
```

### Feature Success Criteria

| Criteria | Description | Target |
|----------|-------------|--------|
| Adoption Target | Minimum percentage of users using feature | Varies by feature type |
| Satisfaction Target | Minimum user satisfaction score | > 4.0 / 5.0 |
| Performance Target | Acceptable response times and error rates | < 200ms, < 0.1% errors |
| Business Impact | Expected revenue or efficiency improvements | Defined per feature |

### Feature Health Monitoring

- **Usage Trends**: Adoption rate changes over time
- **Performance Trends**: Response time and reliability metrics
- **Support Load**: Help tickets related to the feature
- **Competitive Position**: Feature advantage relative to competitors

## Fit Score Thresholds

| Score Range | Confidence Level | Interpretation |
|-------------|------------------|----------------|
| 80-100 | Strong | Excellent product-market fit |
| 60-79 | Moderate | Good fit, room for improvement |
| 40-59 | Weak | Needs significant optimization |
| 0-39 | Poor | Major pivots may be needed |

---

**Related Documents:**
- [Overview](overview.md)
- [Experimentation](experimentation.md)
- [Optimization & Reporting](optimization-reporting.md)
