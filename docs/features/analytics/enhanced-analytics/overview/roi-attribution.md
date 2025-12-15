---
title: "ROI Attribution"
description: Revenue attribution models and campaign ROI calculation
last_modified_date: 2025-01-15
level: 3
persona: ["developer", "marketing-analyst"]
keywords: [ROI, attribution, revenue, conversion, analytics]
---

# ROI Attribution

Multi-touch attribution models for accurate campaign revenue tracking and ROI calculation.

## Attribution Service

Calculates campaign ROI using configurable attribution models.

```typescript
class ROIAttributionService {
  async calculateCampaignROI(
    campaignId: string,
    attributionModel: 'first_touch' | 'last_touch' | 'linear' | 'time_decay' | 'position_based'
  ): Promise<{
    revenue: number;
    cost: number;
    roi: number;
    attributedConversions: number;
  }> {
    const campaign = await db.campaigns.findById(campaignId);
    const conversions = await this.getConversions(campaignId);
    const attributedRevenue = await this.applyAttributionModel(
      conversions,
      attributionModel
    );
    const cost = this.calculateCampaignCost(campaign);
    const roi = ((attributedRevenue - cost) / cost) * 100;

    return {
      revenue: attributedRevenue,
      cost,
      roi,
      attributedConversions: conversions.length,
    };
  }

  private async applyAttributionModel(
    conversions: Conversion[],
    model: string
  ): Promise<number> {
    let totalRevenue = 0;

    for (const conversion of conversions) {
      const touchpoints = await this.getTouchpoints(conversion.contactId);
      const attribution = this.calculateAttribution(touchpoints, model);
      totalRevenue += conversion.revenue * attribution;
    }

    return totalRevenue;
  }

  private calculateAttribution(touchpoints: Touchpoint[], model: string): number {
    switch (model) {
      case 'first_touch':
        return touchpoints[0]?.campaignId === this.campaignId ? 1 : 0;

      case 'last_touch':
        return touchpoints[touchpoints.length - 1]?.campaignId === this.campaignId ? 1 : 0;

      case 'linear':
        const campaignTouchpoints = touchpoints.filter(t => t.campaignId === this.campaignId);
        return campaignTouchpoints.length / touchpoints.length;

      case 'time_decay':
        return this.calculateTimeDecayAttribution(touchpoints);

      case 'position_based':
        return this.calculatePositionBasedAttribution(touchpoints);
    }
  }
}
```

## Attribution Models

| Model | Description | Best For |
|-------|-------------|----------|
| First Touch | 100% credit to first campaign | Brand awareness campaigns |
| Last Touch | 100% credit to last campaign | Direct response campaigns |
| Linear | Equal credit to all touchpoints | Multi-step nurture sequences |
| Time Decay | More credit to recent touchpoints | Short sales cycles |
| Position Based | 40% first, 40% last, 20% middle | Balanced attribution |

## ROI Tracking Metrics

Key metrics calculated by the attribution system:

- **Revenue per Email**: total_revenue / emails_sent
- **Revenue per Recipient**: total_revenue / unique_recipients
- **Cost per Acquisition**: campaign_cost / conversions
- **Return on Investment**: (revenue - cost) / cost * 100

## Technical Stack

The enhanced analytics system uses the following components:

| Component | Technology | Purpose |
|-----------|------------|---------|
| Backend | Node.js with TypeScript | API and business logic |
| ML/AI | Gemini AI API | Predictive analytics |
| Database | PostgreSQL + TimescaleDB | Time-series data storage |
| Analytics | PostHog | Event tracking |
| Visualization | Chart.js/Recharts | Dashboard charts |
| Real-time | Redis | Real-time metrics |
| Queue | PostgreSQL + Redis | Background job processing |

## Related Documentation

- [Enhanced Metrics](./enhanced-metrics) - KPIs and scoring
- [Predictive Analytics](./predictive-analytics) - ML predictions
- [Implementation Plan](./implementation-plan) - Rollout details
