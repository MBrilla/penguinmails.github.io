---
title: "Advanced Subscription Features"
description: "Billing cycles, usage monitoring, API access, and future features"
last_modified_date: "2025-01-15"
level: "3"
persona: "Billing Administrators, Developers"
status: "ACTIVE"
category: "Payments"
---

# Advanced Subscription Features

Advanced subscription management including billing cycles, usage monitoring, and API access.

## Annual vs Monthly Billing

**Save 20% with annual commitment.**

### Switching to Annual Billing

**From Monthly to Annual:**

1. Go to **Settings** → **Billing** → **Switch to Annual**
2. Review annual cost (20% discount applied)
3. Pay annual amount today
4. Monthly billing stopped
5. Next charge in 12 months

**Credit for Remaining Monthly Period:**
- Prorated credit applied to annual payment
- Example: 15 days left on monthly → $24.50 credit → reduces annual payment

**Benefits:**
- 20% cost savings
- Predictable yearly budgeting
- Bonus features (priority support, extra sends)

### Switching to Monthly Billing

**From Annual to Monthly:**
- Changes take effect at end of annual period
- No refunds for unused annual time
- Can switch anytime, effective on anniversary

---

## Usage Monitoring & Overages

**Track usage and handle overages gracefully.**

### Usage Dashboard

**View real-time usage:**
- **Emails Sent:** X / Y included (Z% used)
- **Team Members:** X / Y seats
- **Workspaces:** X / Y workspaces
- **Domains:** X / Y domains

**Usage Alerts:**
- 80% of limit - Warning email
- 95% of limit - Urgent notification
- 100% of limit - Action required

### Overage Handling

**Email Sends:**
- **Auto-Add On:** Automatically purchase extra emails ($10/10k)
- **Manual Approval:** Require approval before overage charged
- **Block Sends:** Stop sending when limit reached (not recommended)

**Team Members:**
- Cannot add beyond limit
- Must upgrade plan or remove members
- Pending invitations don't count until accepted

**Workspaces:**
- Cannot create beyond limit
- Must upgrade or archive existing workspaces

---

## Subscription API

**Manage subscriptions programmatically.**

### Get Current Subscription

```javascript
GET /api/v1/billing/subscription
Authorization: Bearer {api_key}

Response:
{
  "plan": "professional",
  "status": "active",
  "billing_cycle": "monthly",
  "amount": 149.00,
  "currency": "USD",
  "next_billing_date": "2025-12-24",
  "usage": {
    "emails_sent": 23450,
    "emails_included": 50000,
    "users": 3,
    "users_included": 5,
    "workspaces": 7,
    "workspaces_included": 10
  }
}
```

### Update Subscription

```javascript
POST /api/v1/billing/subscription/update
Authorization: Bearer {api_key}

{
  "plan": "business",
  "billing_cycle": "annual"
}

Response:
{
  "subscription_id": "sub_abc123",
  "plan": "business",
  "status": "active",
  "proration_amount": 250.00,
  "next_billing_date": "2026-11-24"
}
```

---

## Multi-Currency Support (Future)

**Coming Q2 2026** - Support for multiple currencies.

**Planned Currencies:**
- USD (US Dollar)
- EUR (Euro)
- GBP (British Pound)
- CAD (Canadian Dollar)
- AUD (Australian Dollar)

---

## One-Time Purchases (Post-MVP)

**Status: TBD (Q3 2026)**

One-time purchases are planned for post-MVP. Potential use cases:
- Custom integration setup fees
- Consulting and training packages
- One-time email credit top-ups
- Data migration services

---

**Related Documents:**
- [Overview](overview.md)
- [Available Plans](plans.md)
- [Managing Subscriptions](management.md)
- [Billing Dashboard](/docs/features/payments/billing-dashboard)
- [Stripe Integration](/docs/features/payments/stripe-integration)
