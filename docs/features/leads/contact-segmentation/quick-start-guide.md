---
title: "Contact Segmentation Quick Start Guide"
description: "Create your first segment and explore common segment types for targeted campaigns"
last_modified_date: "2025-12-15"
level: "1"
parent: "contact-segmentation"
---

# Quick Start Guide

Learn how to create segments and use common segment patterns for effective campaign targeting.

---

## Your First Segment

### Step 1: Create Segment

```text
Contacts → Segments → Create New Segment

Segment Name: Active Subscribers
Description: Contacts who opened emails in last 30 days
Type: ○ Dynamic  ○ Static
```

### Step 2: Define Rules (Dynamic Segment)

```text
Conditions (Match ALL):
  ☑ Last Email Opened: within last 30 days
  ☑ Subscription Status: Active
  ☑ Unsubscribed: No

Preview: 2,847 contacts match
```

### Step 3: Save & Use

```text
[Save Segment]

✓ Segment created: "Active Subscribers"
  2,847 contacts

Quick Actions:
  [Create Campaign] [Export List] [View Contacts]
```

---

## Common Segment Types

### By Engagement Level

**Highly Engaged:**

```text
Conditions:
  - Opened emails: ≥ 5 in last 30 days
  - Clicked links: ≥ 2 in last 30 days
  - Last activity: Within 7 days
```

**Inactive Contacts:**

```text
Conditions:
  - Last opened: More than 90 days ago
  - Subscription status: Active
  - Unsubscribed: No
```

**Never Engaged:**

```text
Conditions:
  - Total opens: 0
  - Total clicks: 0
  - Date added: More than 30 days ago
```

### By Demographics

**Location-Based:**

```text
Conditions:
  - Country: United States
  - State: California
  OR
  - City: San Francisco
```

**Company Size:**

```text
Conditions:
  - Custom Field: company_size
  - Value: 51-200 employees
```

**Industry:**

```text
Conditions:
  - Custom Field: industry
  - Value: SaaS, Technology, Software
```

### By Lifecycle Stage

**New Leads (Last 7 Days):**

```text
Conditions:
  - Date added: Within last 7 days
  - Emails sent: 0
```

**Trial Users:**

```text
Conditions:
  - Custom Field: account_type = "trial"
  - Custom Field: trial_end_date: Within next 7 days
```

**Customers:**

```text
Conditions:
  - Custom Field: account_type = "paid"
  - Custom Field: last_purchase_date: Within last 90 days
```

---

## Static Segments

### When to Use

- One-time campaigns (event invitations, announcements)
- Manual contact selection
- Imported lists from external sources
- A/B test control groups

### Creating Static Segment

```text
1. Select contacts manually from list
2. Or import CSV file
3. Name segment
4. Save as static segment (won't auto-update)
```

---

## Next Steps

- [Advanced Segmentation](./advanced-segmentation) - Complex rule logic and behavioral patterns
- [Segment Analytics](./segment-analytics) - Track segment performance

---

**Related:** [Leads Management](/docs/features/leads/leads-management) | [Lead Scoring](/docs/features/leads/lead-scoring)
