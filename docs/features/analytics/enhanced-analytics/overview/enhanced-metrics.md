---
title: "Enhanced Metrics"
description: Advanced KPIs and composite scoring metrics for email analytics
last_modified_date: 2025-01-15
level: 3
persona: ["developer", "data-analyst"]
keywords: [metrics, KPIs, deliverability, engagement, scoring]
---

# Enhanced Metrics

Advanced KPIs and composite scoring for comprehensive email performance measurement.

## Deliverability Score (0-100)

Multi-factor score combining infrastructure, reputation, engagement, and complaint metrics.

```yaml
deliverability_score:
  components:
    infrastructure: 30  # SPF/DKIM/DMARC
    reputation: 25  # Sender reputation
    engagement: 25  # Open/click rates
    complaints: 20  # Bounce/spam rates

  calculation:
    infrastructure_score:
      spf_pass: 10
      dkim_pass: 10
      dmarc_pass: 10

    reputation_score:
      domain_reputation: 15
      ip_reputation: 10

    engagement_score:
      open_rate_percentile: 15
      click_rate_percentile: 10

    complaints_score:
      bounce_penalty: -10_per_percent
      spam_penalty: -50_per_percent
```

## Engagement Velocity

Measures the rate of engagement change over time to identify trending performance.

```yaml
engagement_velocity:
  description: "Rate of engagement change over time"
  formula: "(current_period_engagement - previous_period_engagement) / previous_period_engagement * 100"

  example:
    week_1_engagement: 25%
    week_2_engagement: 30%
    velocity: +20%  # Positive trend

  interpretation:
    positive: "Engagement improving"
    negative: "Engagement declining"
    neutral: "Engagement stable"
```

## Mailbox Health Index

Predictive health scoring per mailbox combining multiple factors.

```yaml
mailbox_health_index:
  description: "Predictive health scoring per mailbox"

  factors:
    - sending_volume_consistency
    - bounce_rate_trend
    - spam_complaint_trend
    - engagement_trend
    - blacklist_status
    - authentication_status

  score_ranges:
    90-100: "Excellent - No action needed"
    75-89: "Good - Monitor closely"
    50-74: "Fair - Improvement needed"
    0-49: "Poor - Immediate action required"
```

## Campaign Performance Index

Multi-factor campaign success metric combining deliverability, engagement, and conversion.

```yaml
campaign_performance_index:
  description: "Multi-factor campaign success metric"

  components:
    deliverability: 30%
    engagement: 40%
    conversion: 30%

  calculation:
    deliverability_component:
      - inbox_rate * 0.7
      - (1 - bounce_rate) * 0.3

    engagement_component:
      - open_rate * 0.5
      - click_rate * 0.3
      - reply_rate * 0.2

    conversion_component:
      - conversion_rate * 1.0
```

## ROI Tracking Metrics

Revenue and cost metrics for campaign performance evaluation.

```yaml
roi_tracking:
  metrics:
    revenue_per_email: total_revenue / emails_sent
    revenue_per_recipient: total_revenue / unique_recipients
    cost_per_acquisition: campaign_cost / conversions
    return_on_investment: (revenue - cost) / cost * 100

  attribution_models:
    - first_touch  # First campaign contact engaged with
    - last_touch  # Last campaign before conversion
    - linear  # Equal credit to all touchpoints
    - time_decay  # More credit to recent touchpoints
    - position_based  # 40% first, 40% last, 20% middle
```

## Data Accuracy Improvements

Improving tracking accuracy from 75% to 90% through enhanced collection methods.

| Improvement | Description |
|-------------|-------------|
| Multi-Pixel Tracking | Implement fingerprinting for open detection |
| Click Attribution | Enhanced click tracking with device fingerprinting |
| Bounce Classification | ML-based hard vs soft bounce classification |
| Engagement Signals | Track read time, scroll depth, forward rate |
| Cross-Reference | Validate against ESP data when available |
| Bot Detection | Filter bot opens/clicks using behavioral analysis |
| Privacy-Safe Tracking | Respect Apple MPP and privacy features |
