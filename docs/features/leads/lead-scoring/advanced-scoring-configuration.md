---
title: "Advanced Scoring Configuration"
description: "Custom behavioral and demographic scoring rules, decay settings, and automation triggers"
last_modified_date: "2025-12-15"
level: "2"
status: "PLANNED"
roadmap_timeline: "Q1 2026"
priority: "High"
parent: lead-scoring
---

# Advanced Scoring Configuration

Customize your scoring model with behavioral, demographic, and intent-based rules.

## Custom Scoring Rules

### Behavioral Scoring

#### Engagement Actions

```yaml
email_actions:
  opened_email:
    points: 5
    decay: true  # Points decay over time

  clicked_link:
    points: 10
    multiplier: 1.5  # 15 points if clicked multiple times

  clicked_pricing_page:
    points: 25
    description: "High intent action"

  downloaded_resource:
    points: 15
    specific_resources:
      whitepaper: 20
      case_study: 15
      ebook: 10

  watched_demo_video:
    points: 30
    threshold: 75%  # Must watch 75% to count

  replied_to_email:
    points: 20

  forwarded_email:
    points: 12
```

#### Website Activity (if integrated)

```yaml
website_actions:
  visited_pricing_page:
    points: 20

  visited_features_page:
    points: 10

  started_trial:
    points: 50

  requested_demo:
    points: 60

  added_to_cart:
    points: 40

  time_on_site:
    gt_5_minutes: 5
    gt_15_minutes: 10
```

#### Negative Actions

```yaml
negative_actions:
  unsubscribed:
    points: -100
    set_score_to: 0

  marked_spam:
    points: -50

  email_bounced_hard:
    points: -25
    set_score_to: 0

  email_bounced_soft:
    points: -5

  inactive_90_days:
    points: -30

  inactive_180_days:
    points: -50
```

---

## Demographic Scoring

### Firmographic Data

```yaml
company_attributes:
  company_size:
    1-10: 0
    11-50: 5
    51-200: 10
    201-1000: 15
    1001+: 20

  industry:
    saas: 20
    technology: 15
    ecommerce: 15
    healthcare: 10
    finance: 10
    other: 0

  revenue:
    lt_1m: 0
    1m_10m: 10
    10m_50m: 15
    50m_plus: 20
```

### Role-Based Scoring

```yaml
job_title_keywords:
  decision_makers:
    ceo: 25
    cto: 25
    vp: 20
    director: 15
    head_of: 15

  influencers:
    manager: 10
    lead: 8
    senior: 8

  end_users:
    specialist: 3
    coordinator: 3
    analyst: 5
```

### Geographic Scoring

```yaml
location:
  tier_1_markets:  # US, UK, Canada, Australia
    points: 10

  tier_2_markets:  # Western Europe
    points: 5

  tier_3_markets:  # Rest of world
    points: 0
```

---

## Score Decay & Recency

### Time-Based Decay

Keep scores accurate by decaying inactive leads:

```yaml
decay_rules:
  engagement_decay:
    enabled: true
    rate: 5_percent
    interval: 30_days
    min_score: 0

  example:
    initial_score: 80
    after_30_days: 76  # -5%
    after_60_days: 72  # -5% again
    after_90_days: 68
```

### Recency Boosting

Reward recent engagement with score multipliers:

```yaml
recency_multipliers:
  action_within_24h:
    multiplier: 2.0

  action_within_7d:
    multiplier: 1.5

  action_within_30d:
    multiplier: 1.0

  action_older_than_30d:
    multiplier: 0.5
```

---

## Multi-Dimensional Scoring

### Separate Scores for Different Aspects

Break down scoring into meaningful dimensions:

```yaml
scoring_dimensions:
  engagement_score:  # 0-40 points
    email_opens: 5
    email_clicks: 10
    email_replies: 15

  fit_score:  # 0-40 points
    company_size: 15
    industry: 15
    job_title: 10

  intent_score:  # 0-20 points
    pricing_page_visit: 10
    demo_request: 20
    trial_started: 20

  composite_score:  # Total 0-100
    formula: engagement + fit + intent
```

---

## Score-Based Automation

### Auto-Segmentation

Automatically segment leads as their scores change:

```text
When lead score reaches 75:
  → Add to "Hot Leads" segment
  → Trigger "Sales Qualified Lead" workflow
  → Notify sales team
  → Send "Book a Demo" campaign
```

### Lead Lifecycle Stages

Map scores to lifecycle stages:

```text
Score 0-25:    Status = "Cold Lead"
Score 26-50:   Status = "Nurture"
Score 51-75:   Status = "Marketing Qualified Lead (MQL)"
Score 76-100:  Status = "Sales Qualified Lead (SQL)"
```

### CRM Sync

Sync high-scoring leads to your CRM:

```text
When lead score >= 75:
  → Create lead in Salesforce
  → Assign to sales rep (round-robin)
  → Set priority = "High"
  → Add to sales follow-up queue
```

---

## Related Topics

- **[Quick Start Guide](./quick-start-guide)** - Basic scoring concepts
- **[Score Analytics](./score-analytics)** - Measure scoring effectiveness
- **[Technical Implementation](./technical-implementation)** - Developer details

---

**[Back to Lead Scoring Overview](./README)**
