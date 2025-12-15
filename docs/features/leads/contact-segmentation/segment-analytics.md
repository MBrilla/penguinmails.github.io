---
title: "Segment Analytics"
description: "Segment exclusions, performance tracking, and A/B testing for optimized targeting"
last_modified_date: "2025-12-15"
level: "2"
parent: "contact-segmentation"
---

# Segment Analytics

Track segment performance, manage exclusions, and run A/B tests to optimize your targeting strategy.

---

## Segment Exclusions

### Exclude Other Segments

```text
Include:
  - Segment: "All Active Subscribers"

Exclude:
  - Segment: "Recent Purchasers"
  - Segment: "Unengaged (90+ days)"
```

### Suppress Based on Recent Activity

```text
Include:
  - All contacts in "Product Users"

Exclude:
  - Received campaign: Within last 7 days
  - Campaign name: contains "Product"
```

---

## Segment Performance Tracking

### View Segment Analytics

```text
Segment: "Active Subscribers" (2,847 contacts)

Performance (Last 30 Days):
  Average Open Rate: 32%
  Average Click Rate: 8%
  Unsubscribe Rate: 0.2%
  Revenue Generated: $12,450

Trend:
  - Segment size: +12% month-over-month
  - Engagement: +5% vs all contacts
```

### Key Metrics to Monitor

| Metric | Description | Benchmark |
|--------|-------------|-----------|
| **Segment Size** | Number of contacts matching criteria | Varies by segment |
| **Open Rate** | % of emails opened by segment | 20-30% |
| **Click Rate** | % of emails clicked by segment | 3-5% |
| **Conversion Rate** | % of segment that converted | 1-3% |
| **Revenue** | Revenue attributed to segment | Depends on goals |
| **Growth Rate** | Month-over-month segment size change | Positive trend |

---

## Segment A/B Testing

### Test Segment Criteria

```text
Hypothesis: "High engagement = Lead Score > 60"

Test Setup:
  Segment A: Lead Score > 60 (500 contacts)
  Segment B: Lead Score 40-60 (500 contacts)

  Send same campaign to both
  Compare: Open rate, Click rate, Conversion rate

Result: Segment A 2.3x higher conversion
Action: Adjust scoring model
```

### A/B Testing Best Practices

1. **Clear Hypothesis**: Define what you're testing and expected outcome
2. **Equal Sample Sizes**: Use similar-sized segments for fair comparison
3. **Same Campaign**: Send identical content to isolate segment impact
4. **Sufficient Time**: Allow enough time for results to be statistically significant
5. **Single Variable**: Test one segmentation criteria at a time

### Common A/B Tests

| Test | Segment A | Segment B | Measure |
|------|-----------|-----------|---------|
| **Engagement Threshold** | Opens ≥ 5 | Opens ≥ 3 | Click rate |
| **Recency** | Active 30 days | Active 60 days | Conversion |
| **Lead Score** | Score > 70 | Score > 50 | Revenue |
| **Demographics** | Enterprise | SMB | Response rate |

---

## Optimization Strategies

### Segment Hygiene

- **Review Regularly**: Check segment criteria quarterly
- **Prune Inactive**: Remove segments not used in 90+ days
- **Consolidate**: Merge similar segments to reduce complexity
- **Document**: Maintain clear naming conventions and descriptions

### Performance Improvement

1. **Low Open Rates**: Refine timing or subject line criteria
2. **High Unsubscribes**: Narrow segment to more engaged contacts
3. **Low Conversions**: Add behavioral signals to segment rules
4. **Declining Size**: Loosen criteria or improve lead generation

---

## Next Steps

- [Technical Implementation](./technical-implementation) - Database schema and API reference
- [Quick Start Guide](./quick-start-guide) - Review fundamentals

---

**Related:** [Lead Scoring](/docs/features/leads/lead-scoring) | [Campaign Management](/docs/features/campaigns/campaign-management/hub)
