---
title: Customer Success Integration
description: Health monitoring and retention campaign automation
last_modified_date: 2025-01-15
level: 3
persona: ["developer", "analyst"]
keywords: [customer-success, health-score, retention, churn-prevention]
---
{% raw %}

# Customer Success Integration

Integration patterns for marketing engagement data feeding customer health scores and automated retention campaigns.

## Customer Health Monitoring

**Pattern:** Marketing engagement data feeding customer health scores

### Health Score Calculator

```typescript
interface CustomerHealthScore {
  calculateHealthScore(tenantId: string): Promise<HealthScore>;
  triggerHealthInterventions(score: HealthScore): Promise<void>;
  syncHealthToCRM(tenantId: string, healthScore: HealthScore): Promise<void>;
}

class CustomerHealthIntegration {
  async calculateHealthScore(tenantId: string): Promise<HealthScore> {
    const [
      usageMetrics,
      engagementMetrics,
      supportMetrics,
      financialMetrics,
      marketingMetrics
    ] = await Promise.all([
      this.getUsageMetrics(tenantId),
      this.getEngagementMetrics(tenantId),
      this.getSupportMetrics(tenantId),
      this.getFinancialMetrics(tenantId),
      this.getMarketingEngagementMetrics(tenantId)
    ]);

    const healthScore = {
      overall_score: this.calculateWeightedScore({
        usage_frequency: usageMetrics.frequency * 0.25,
        feature_adoption: usageMetrics.adoption_rate * 0.20,
        engagement_trend: engagementMetrics.trend * 0.15,
        support_health: supportMetrics.negative_indicators * -0.15,
        payment_stability: financialMetrics.payment_history * 0.15,
        marketing_engagement: marketingMetrics.engagement_score * 0.10
      }),
      risk_indicators: this.identifyRiskIndicators({
        usage: usageMetrics,
        engagement: engagementMetrics,
        support: supportMetrics
      }),
      health_trend: await this.calculateHealthTrend(tenantId),
      last_updated: new Date().toISOString()
    };

    return healthScore;
  }

  private calculateWeightedScore(metrics: Record<string, number>): number {
    const weights = {
      usage_frequency: 0.25,
      feature_adoption: 0.20,
      engagement_trend: 0.15,
      support_health: 0.15,
      payment_stability: 0.15,
      marketing_engagement: 0.10
    };

    return Math.round(
      Object.entries(metrics).reduce((total, [key, value]) => {
        return total + (value * weights[key]);
      }, 0) * 100
    ) / 100;
  }
}
```

### Marketing Engagement Metrics

```typescript
interface MarketingEngagementMetrics {
  email_engagement: {
    open_rate_30d: number;
    click_rate_30d: number;
    unsubscribe_rate: number;
    last_email_activity: string;
  };
  campaign_participation: {
    campaigns_enrolled: number;
    campaigns_completed: number;
    response_rate: number;
  };
  content_consumption: {
    blog_reads_30d: number;
    webinar_attendance: number;
    resource_downloads: number;
  };
  advocacy_indicators: {
    referral_activity: number;
    review_submissions: number;
    case_study_participation: number;
  };
}
```

## Retention Campaign Automation

**Pattern:** Health score triggers automated retention campaigns

### Campaign Trigger Rules

```typescript
interface RetentionCampaignTriggers {
  high_risk_triggers: {
    health_score_drop: number;        // >20 point drop in 30 days
    usage_decline: number;            // >50% decline in usage
    support_escalation: boolean;      // Support ticket escalation
    non_response: number;             // No email engagement in 45 days
  };
  medium_risk_triggers: {
    usage_plateau: boolean;           // Flat usage for 60+ days
    feature_underutilization: number; // <30% adoption of key features
    payment_delays: number;           // Late payments in last 90 days
  };
}

const retentionCampaignEngine = {
  async evaluateRetentionTriggers(tenantId: string): Promise<void> {
    const healthScore = await this.calculateHealthScore(tenantId);
    const triggers = this.checkRetentionTriggers(healthScore);

    if (triggers.high_risk.length > 0) {
      await this.triggerHighRiskCampaign(tenantId, triggers.high_risk);
      await this.notifyCustomerSuccess(tenantId, 'high_risk');
    } else if (triggers.medium_risk.length > 0) {
      await this.triggerMediumRiskCampaign(tenantId, triggers.medium_risk);
    }

    await this.logRetentionIntervention(tenantId, triggers, healthScore);
  }
};
```

## Related Documentation

- [Integration Overview](./)
- [Product Domain Integration](./product-domain)
- [Finance Domain Integration](./finance-domain)

{% endraw %}
