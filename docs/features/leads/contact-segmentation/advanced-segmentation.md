---
title: "Advanced Segmentation"
description: "Complex rule logic, behavioral segmentation, lead score integration, and custom field filtering"
last_modified_date: "2025-12-15"
level: "2"
parent: "contact-segmentation"
---

# Advanced Segmentation

Master complex segmentation strategies using multi-criteria rules, behavioral patterns, and lead scoring integration.

---

## Complex Rule Logic

### AND/OR Combinations

**Engaged OR High-Value:**

```text
Match ANY of these groups:

Group 1 (Engaged):
  - Opens: ≥ 3 in last 30 days
  - Clicks: ≥ 1 in last 30 days

Group 2 (High-Value):
  - Custom Field: lead_score ≥ 75
  - Custom Field: customer_ltv ≥ $1,000
```

**Advanced Multi-Condition:**

```text
Match ALL:
  Group 1 (Location):
    - Country: United States OR Canada
    - Timezone: PST, MST, CST, EST

  Group 2 (Engagement):
    - Last opened: Within 60 days
    - Total clicks: ≥ 5

  Group 3 (Not):
    - Tag: does NOT include "unqualified"
    - Custom Field: bounced ≠ true
```

---

## Behavioral Segmentation

### Email Engagement Patterns

**Click-But-Never-Convert:**

```text
Conditions:
  - Total clicks: ≥ 10
  - Total conversions: 0
  - Last clicked: Within 30 days
```

**Opens Mobile vs Desktop:**

```text
Conditions:
  - Last opened device: Mobile
  - Open count (mobile): ≥ 5
  - Open count (desktop): 0
```

**Time-Based Engagement:**

```text
Conditions:
  - Most common open time: 9am-12pm
  - Preferred day: Monday-Friday
```

### Campaign-Specific Behavior

**Clicked Specific Link:**

```text
Conditions:
  - Campaign: "Product Launch Q4"
  - Clicked URL: contains "pricing"
  - Did NOT click URL: contains "demo"
```

**Opened But Didn't Click:**

```text
Conditions:
  - Campaign: "Feature Announcement"
  - Opened: Yes
  - Clicked any link: No
  - Days since sent: ≥ 3
```

---

## Lead Score Integration

### Score-Based Segments

```text
Hot Leads (Score 80-100):
  - Lead Score: 80-100
  - Last activity: Within 14 days
  - Opened last email: Yes

Warm Leads (Score 50-79):
  - Lead Score: 50-79
  - Last activity: Within 30 days

Cold Leads (Score 0-49):
  - Lead Score: 0-49
  OR
  - Last activity: More than 60 days ago
```

---

## Custom Field Segmentation

### Multi-Select Custom Fields

```text
Conditions:
  - Custom Field: interests
  - Contains ANY: "email marketing", "automation", "analytics"
```

### Date-Based Custom Fields

```text
Trial Expiring Soon:
  - Custom Field: trial_end_date
  - Is between: today and 7 days from now

Anniversary Outreach:
  - Custom Field: signup_date
  - Anniversary: this month
```

### Numeric Range Custom Fields

```text
High-Value Cohort:
  - Custom Field: total_revenue
  - Is greater than: $5,000
  - Custom Field: lifetime_opens
  - Is greater than: 50
```

---

## Next Steps

- [Segment Analytics](./segment-analytics) - Track performance and run A/B tests
- [Technical Implementation](./technical-implementation) - Understand the underlying architecture

---

**Related:** [Lead Scoring](/docs/features/leads/lead-scoring) | [Campaign Management](/docs/features/campaigns/campaign-management/hub)
