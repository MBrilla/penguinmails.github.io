---
title: "AI Recommendations"
description: AI-powered recommendation engine and custom dashboard capabilities
last_modified_date: 2025-01-15
level: 3
persona: ["developer", "product-manager"]
keywords: [AI, recommendations, dashboards, segmentation, optimization]
---

# AI Recommendations

AI-powered recommendation engine for automated campaign optimization and custom analytics dashboards.

## AI Recommendation Engine

Generates automated recommendations for campaign optimization across multiple dimensions.

```typescript
class AIRecommendationEngine {
  async generateCampaignRecommendations(
    campaignId: string
  ): Promise<Recommendation[]> {
    const campaign = await db.campaigns.findById(campaignId);
    const recommendations: Recommendation[] = [];

    const sendTimeRec = await this.recommendSendTime(campaign);
    if (sendTimeRec) recommendations.push(sendTimeRec);

    const subjectRec = await this.recommendSubjectLine(campaign);
    if (subjectRec) recommendations.push(subjectRec);

    const contentRec = await this.recommendContentChanges(campaign);
    if (contentRec) recommendations.push(contentRec);

    const audienceRec = await this.recommendAudienceChanges(campaign);
    if (audienceRec) recommendations.push(audienceRec);

    const deliverabilityRec = await this.recommendDeliverabilityImprovements(campaign);
    if (deliverabilityRec) recommendations.push(deliverabilityRec);

    return recommendations;
  }

  private async recommendSendTime(campaign: Campaign): Promise<Recommendation | null> {
    const currentSendTime = campaign.scheduledAt;
    const optimalTime = await this.calculateOptimalSendTime(campaign.segmentId);

    if (this.timeDifferenceHours(currentSendTime, optimalTime) > 2) {
      return {
        type: 'send_time',
        priority: 'high',
        title: 'Optimize Send Time',
        description: `Send at ${optimalTime.toLocaleString()} for 23% higher open rate`,
        impact: '+23% open rate',
        effort: 'Low',
        action: { type: 'reschedule', newTime: optimalTime },
      };
    }
    return null;
  }

  private async recommendSubjectLine(campaign: Campaign): Promise<Recommendation | null> {
    const prediction = await new SubjectLinePredictor().predictOpenRate(
      campaign.subject,
      campaign.segmentId
    );

    if (prediction.recommendations.length > 0) {
      return {
        type: 'subject_line',
        priority: 'medium',
        title: 'Improve Subject Line',
        description: prediction.recommendations[0],
        impact: `+${Math.round((prediction.potentialImprovement || 0) * 100)}% open rate`,
        effort: 'Low',
        action: { type: 'edit_subject', suggestions: prediction.recommendations },
      };
    }
    return null;
  }
}
```

## Recommendation Types

The system generates recommendations across five key areas:

1. **Send Time Optimization**: "Send at 10 AM Tuesday for 23% higher open rate"
2. **Subject Line Improvement**: "Try shorter subject lines (< 50 chars) for this audience"
3. **Mailbox Selection**: "Use mailbox B for higher deliverability to @gmail.com"
4. **Sequence Timing**: "Add 2-day delay between steps 2 and 3"
5. **Content Optimization**: "Recipients engage more with bullet points vs paragraphs"

## Custom Dashboards

Configurable dashboard system with drag-and-drop widgets.

```typescript
interface DashboardWidget {
  id: string;
  type: 'metric' | 'chart' | 'table' | 'funnel';
  title: string;
  config: any;
  position: { x: number; y: number; w: number; h: number };
}

interface CustomDashboard {
  id: string;
  name: string;
  description: string;
  widgets: DashboardWidget[];
  filters: DashboardFilter[];
  isDefault: boolean;
}

class CustomDashboardService {
  async createDashboard(
    tenantId: string,
    dashboard: Partial<CustomDashboard>
  ): Promise<CustomDashboard> {
    return await db.customDashboards.create({
      tenantId,
      name: dashboard.name,
      description: dashboard.description,
      widgets: dashboard.widgets || [],
      filters: dashboard.filters || [],
      isDefault: dashboard.isDefault || false,
    });
  }

  async renderWidget(widget: DashboardWidget): Promise<any> {
    switch (widget.type) {
      case 'metric':
        return await this.renderMetricWidget(widget);
      case 'chart':
        return await this.renderChartWidget(widget);
      case 'table':
        return await this.renderTableWidget(widget);
      case 'funnel':
        return await this.renderFunnelWidget(widget);
    }
  }

  private async renderMetricWidget(widget: DashboardWidget): Promise<any> {
    const { metric, timeRange, comparison } = widget.config;
    const currentValue = await this.calculateMetric(metric, timeRange);
    const previousValue = comparison
      ? await this.calculateMetric(metric, this.getPreviousPeriod(timeRange))
      : null;

    return {
      value: currentValue,
      previousValue,
      change: previousValue ? ((currentValue - previousValue) / previousValue) * 100 : null,
      trend: currentValue > previousValue ? 'up' : 'down',
    };
  }
}
```

## Advanced Segmentation

Dynamic and predictive audience segments powered by behavioral analysis and ML models.

```yaml
advanced_segmentation:
  behavioral_segments:
    - name: "Highly Engaged"
      criteria:
        - open_rate >= 50%
        - click_rate >= 10%
        - last_activity within 7 days

    - name: "At Risk"
      criteria:
        - open_rate < 10%
        - last_activity > 30 days
        - previous_open_rate >= 30%  # Was engaged

    - name: "Champions"
      criteria:
        - lead_score >= 90
        - conversion_count >= 3
        - avg_order_value >= $500

  predictive_segments:
    - name: "Likely to Convert"
      model: conversion_prediction
      threshold: 0.75

    - name: "Churn Risk"
      model: churn_prediction
      threshold: 0.60

    - name: "High Lifetime Value"
      model: ltv_prediction
      threshold: $1000

  dynamic_segments:
    - name: "Recent Browsers"
      criteria:
        - website_visit within 24 hours
        - viewed_product = true
        - not_purchased = true
      refresh: real_time
```
