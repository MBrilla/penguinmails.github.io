---
title: "Lead Scoring Quick Start Guide"
description: "Get started with lead scoring basics - understanding scores, default rules, and campaign targeting"
last_modified_date: "2025-12-15"
level: "1"
status: "PLANNED"
roadmap_timeline: "Q1 2026"
priority: "High"
parent: lead-scoring
---

# Lead Scoring Quick Start Guide

Get up and running with lead scoring to identify your hottest prospects.

## Understanding Lead Scores

### Score Range: 0-100

| Score Range | Status | Meaning |
|-------------|--------|---------|
| 0-25 | Cold/Unengaged | Little to no engagement |
| 26-50 | Warming Up | Beginning to show interest |
| 51-75 | Interested/Engaged | Active engagement |
| 76-100 | Hot/Ready to Buy | High intent signals |

**Recalculation**: Scores update in real-time as contacts take actions.

---

## Your First Scoring Model

### Default Scoring Rules

PenguinMails comes with sensible default scoring rules to get you started immediately.

#### Email Engagement (Positive)

```text
✔ Email Opened: +5 points
✔ Link Clicked: +10 points
✔ Replied to Email: +15 points
```

#### Negative Actions

```text
✗ Email Bounced: -10 points
✗ Marked as Spam: -50 points
✗ Unsubscribed: -100 points (auto-set to 0)
```

#### Demographics

```text
✔ Job Title (Decision Maker): +20 points
✔ Company Size (51-200): +10 points
✔ Company Size (200+): +15 points
```

#### Time Decay

```text
Score decreases 5% every 30 days of inactivity
```

---

## View Lead Scores

Access your scored contacts through the Contacts view:

```text
Contacts → View All

Sort by: Lead Score (High to Low)

Contact                Score  Last Activity
───────────────────────────────────────────
Sarah Johnson          92     2 hours ago
Michael Chen           87     1 day ago
Emily Rodriguez        76     3 days ago
David Kim              68     1 week ago
...
```

### Quick Filters

- **Hot Leads**: Score 76-100
- **Warm Leads**: Score 51-75
- **Nurture**: Score 26-50
- **Cold**: Score 0-25

---

## Use Scores in Campaigns

### Score-Based Segments

Create targeted campaigns based on lead temperature:

#### Hot Leads (Score 76-100)

```text
Campaign: "Book a Demo"
Strategy: Aggressive CTA, direct sales messaging
Goal: Convert to opportunity
```

#### Warm Leads (Score 51-75)

```text
Campaign: "Feature Deep Dive"
Strategy: Educational content, case studies
Goal: Move to hot status
```

#### Cold Leads (Score 0-50)

```text
Campaign: "Getting Started Guide"
Strategy: Nurture sequence, value demonstration
Goal: Re-engage and warm up
```

---

## Best Practices

### Do's

- ✅ Review your scoring model monthly
- ✅ Adjust thresholds based on conversion data
- ✅ Use scores alongside other signals
- ✅ Set up notifications for high-score leads

### Don'ts

- ❌ Ignore score decay (inactive leads aren't hot)
- ❌ Set scoring rules too complex initially
- ❌ Forget to align with sales on definitions
- ❌ Over-rely on demographics alone

---

## Next Steps

Ready to customize your scoring model?

- **[Advanced Scoring Configuration](./advanced-scoring-configuration)** - Create custom rules
- **[Score Analytics](./score-analytics)** - Track scoring effectiveness
- **[Technical Implementation](./technical-implementation)** - API and database details

---

**[Back to Lead Scoring Overview](./README)**
