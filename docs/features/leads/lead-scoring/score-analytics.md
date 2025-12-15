---
title: "Score Analytics"
description: "Track lead score distributions, trends, and conversion correlations to optimize your scoring model"
last_modified_date: "2025-12-15"
level: "2"
status: "PLANNED"
roadmap_timeline: "Q1 2026"
priority: "High"
parent: lead-scoring
---

# Score Analytics

Monitor and analyze your lead scoring effectiveness to continuously improve your model.

## Score Distribution

### Overview Dashboard

Understand how your leads are distributed across score ranges:

```text
Lead Score Distribution:

0-25:  ████████████████ 45% (2,250 contacts)
26-50: ██████████ 28% (1,400 contacts)
51-75: ██████ 18% (900 contacts)
76-100: ███ 9% (450 contacts)

Average Score: 38
Median Score: 32
```

### Distribution Insights

| Metric | Value | Benchmark |
|--------|-------|-----------|
| Average Score | 38 | 35-45 |
| Median Score | 32 | 30-40 |
| % Hot Leads (76+) | 9% | 5-15% |
| % Cold Leads (0-25) | 45% | 40-60% |

---

## Score Trends

### Movement Analysis

Track how scores are changing over time:

```text
Score Movement (Last 30 Days):

Increased Score: 1,200 contacts (+15%)
Decreased Score: 800 contacts (-10%)
No Change: 3,000 contacts
```

### Top Scoring Actions

Identify which actions drive the most scoring activity:

```text
Top Scoring Actions:
  1. Demo Requested: +60 pts (120 actions)
  2. Pricing Page Click: +25 pts (450 actions)
  3. Email Reply: +20 pts (230 actions)
  4. Resource Download: +15 pts (380 actions)
  5. Email Click: +10 pts (2,100 actions)
```

### Weekly Trend Chart

```text
Week        Avg Score   Hot Leads   New MQLs
─────────────────────────────────────────────
Week 1      36          410         85
Week 2      37          425         92
Week 3      38          440         88
Week 4      38          450         95
```

---

## Conversion Correlation

### Score-to-Conversion Analysis

Understand how scores correlate with outcomes:

```text
Conversion Rates by Score Range:

Score Range   Leads   Conversions   Rate
──────────────────────────────────────────
76-100        450     135           30%
51-75         900     126           14%
26-50         1,400   70            5%
0-25          2,250   22            1%
```

### Optimal Threshold Analysis

Determine the best score threshold for handoff to sales:

```text
Threshold Analysis:

Score >= 60:  Precision 45%, Recall 82%
Score >= 70:  Precision 52%, Recall 71%
Score >= 75:  Precision 58%, Recall 65% ← Recommended
Score >= 80:  Precision 68%, Recall 52%
Score >= 90:  Precision 78%, Recall 31%
```

---

## Model Performance

### Score Accuracy Metrics

Evaluate how well your scoring model predicts conversions:

| Metric | Current | Target |
|--------|---------|--------|
| Precision (at threshold 75) | 58% | 60%+ |
| Recall (at threshold 75) | 65% | 70%+ |
| Lead Velocity Rate | +12%/mo | +10%+ |
| MQL-to-SQL Conversion | 42% | 40%+ |

### Scoring Rule Impact

See which rules contribute most to final scores:

```text
Rule Contribution Analysis:

Behavioral Rules:     48% of total points
  - Email engagement: 25%
  - Website activity: 15%
  - Resource downloads: 8%

Demographic Rules:    35% of total points
  - Company size: 15%
  - Job title: 12%
  - Industry: 8%

Intent Signals:       17% of total points
  - Demo requests: 10%
  - Pricing views: 7%
```

---

## Reports & Exports

### Scheduled Reports

Configure automated score reports:

- **Daily**: New hot leads (score 76+)
- **Weekly**: Score movement summary
- **Monthly**: Full scoring model performance

### Export Options

- CSV export of all contact scores
- Score history for individual contacts
- Aggregate analytics for dashboards

---

## Model Optimization Tips

### When to Adjust Your Model

1. **Conversion rates don't correlate with scores** - Review scoring rules
2. **Too many/few hot leads** - Adjust thresholds or point values
3. **Scores cluster in one range** - Diversify scoring signals
4. **High decay rates** - Check engagement opportunities

### A/B Testing Scoring Models

Test alternative scoring approaches:

```text
Model A (Current):     Model B (Test):
- Email open: 5 pts    - Email open: 3 pts
- Click: 10 pts        - Click: 15 pts
- Reply: 15 pts        - Reply: 20 pts

Result after 30 days:
- Model B shows 12% better MQL-to-conversion rate
```

---

## Related Topics

- **[Quick Start Guide](./quick-start-guide)** - Basic scoring concepts
- **[Advanced Configuration](./advanced-scoring-configuration)** - Customize rules
- **[Technical Implementation](./technical-implementation)** - Developer details

---

**[Back to Lead Scoring Overview](./README)**
