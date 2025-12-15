---
title: "Finance Routes"
description: "Admin routes for subscription overview and Stripe integration"
last_modified_date: "2025-12-16"
level: "2"
persona: "Technical Teams"
---

# Finance Routes

## `/dashboard/finance` - Subscription Overview

**User Story**: *"As a finance admin, I want to monitor subscription status and access Stripe Dashboard for revenue analytics."*

### Subscription Summary

- **Basic Metrics**:
  - **Active Subscriptions**: 234.
  - **Plan Distribution**: Free (45), Pro (150), Enterprise (39).
  - **Stripe Dashboard Link**: "View MRR and Revenue Analytics in Stripe" button.
- **Quick Links**:
  - **Open Stripe Dashboard**: Opens Stripe revenue section in new tab.
  - **View All Subscriptions in Stripe**: Direct link to Stripe subscriptions list.

### Stripe Webhook Status

- **Last Webhook**: "5 minutes ago".
- **Webhook Health**:
  - Healthy / Delayed / Error indicator.
  - Last event type: `invoice.paid`, `subscription.updated`, etc.
  - Failed payments count: 3 subscriptions with payment issues.
- **Stripe Links**:
  - "View Payment Details in Stripe" button.
  - "View Failed Payments in Stripe" link.

### Stripe Dashboard Access

- **Quick Actions**:
  - **"Open Stripe Dashboard"**: Opens Stripe in new tab.
  - **"Force Sync Transaction"**: Input Invoice ID to pull latest status from Stripe.
  - **"View MRR Reports"**: Direct link to Stripe revenue section.
  - **"Download Invoices"**: Link to Stripe invoice list.
  - **"Export Revenue Data"**: Link to Stripe data export tools.

> [!NOTE]
> All detailed financial analytics (MRR trends, revenue forecasting, invoice management, payment history) are handled in Stripe Dashboard.

**User Journey Context**: Monthly financial overview and Stripe integration monitoring.

**Related Documentation**:
- [Finance Processes](/docs/business/finance/reporting/mrr-calculation)
- [Stripe Webhooks](/docs/technical/integrations/stripe)

**Technical Integration**:
- **Stripe API**: Read-only access for subscription list and webhook status
- **Subscription Count**: Simple query from `subscriptions` table (`status = 'active'`)
- **No Complex Calculations**: MRR, revenue trends, and analytics handled by Stripe
