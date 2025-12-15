---
title: "Payments & Billing"
description: "Subscription management, billing, invoicing, and payment processing"
last_modified_date: "2025-12-16"
level: "1"
persona: "All Users"
---

# Payments & Billing

Subscription management, billing, invoicing, and payment processing.

## Purpose

Handle all payment-related operations including subscriptions, billing, and invoices.

---

## MVP Status

**Current State:** Partially Complete (Core features documented, 7 MVP gaps identified)

## What's Available Today

- ✅ Multi-tier subscription plans (Starter, Professional, Business, Enterprise)

- ✅ Stripe Checkout integration for secure payment processing

- ✅ Automatic subscription creation and management

- ✅ Prorated upgrades and downgrades

- ✅ Usage tracking (emails, users, workspaces, domains)

- ✅ Automatic invoice generation via Stripe

- ✅ Payment method updates via Stripe Customer Portal

- ✅ Failed payment retry logic (Days 3, 5, 7, 14)

- ✅ Subscription cancellation with 90-day grace period

- ✅ Basic subscription status monitoring

**Assessment:** Core billing infrastructure is functional, but critical user-facing features are missing. 7 MVP gaps block production readiness.

---

## Missing MVP Features

### Critical Gaps (Q1 2026)

1. **[P0] Plan Upgrade/Downgrade UI Flow** (3-5 days)

   - Plan comparison modal with feature differences

   - Proration preview before confirmation

   - Downgrade validation (usage vs new limits)

   - **Impact:** Core monetization feature - users must self-service plan changes

2. **[P0] Usage Tracking Dashboard** (4-6 days)

   - Real-time progress bars with color-coded alerts

   - Historical usage charts (6-month trends)

   - CSV export functionality

   - **Impact:** Prevents service interruptions, drives upgrade conversions

3. **[P1] Invoice Management System** (3-5 days)

   - Direct PDF download from PenguinMails UI

   - Invoice preview modal and search/filtering

   - Bulk download capability

   - **Impact:** Professional businesses expect self-service invoice management

4. **[P1] Payment Method Management UI** (4-6 days)

   - In-app card updates using Stripe Elements

   - Multiple payment methods support

   - Payment method verification

   - **Impact:** Payment failures are major churn driver - easy updates critical

5. **[P0] Failed Payment Recovery Flow** (2-3 days)

   - Dashboard banner with failure reason

   - One-click "Update Payment & Retry" button

   - Grace period countdown timer

   - **Impact:** Quick recovery prevents service interruption and churn

6. **[P1] Subscription Cancellation Flow** (2-3 days)

   - Cancellation reason survey

   - Retention offer modal (discount, pause option)

   - Data export reminder

   - **Impact:** Graceful cancellation reduces negative sentiment

7. **[P2] Budget Controls** (3-4 days)

   - Budget limit configuration

   - Alert thresholds (80%, 100%)

   - Overage handling options

   - **Impact:** Budget-conscious customers need spending controls

**Total MVP Effort:** 21-36 days (4-7 weeks)

---

## Features in This Category

### Available Now

- **[Subscription Management](/docs/features/payments/subscription-management)** - Plan management and upgrades

- **[Account Lifecycle](/docs/features/payments/account-lifecycle)** - Renewal, suspension, and decommission flows

- **[Stripe Integration](/docs/features/payments/stripe-integration)** - Payment processing via Stripe

- **[Billing Dashboard](/docs/features/payments/billing-dashboard)** - Usage and billing overview

### 🚧 In Progress (Q1 2026)

- **Plan Upgrade/Downgrade UI** - Visual comparison and confirmation workflow

- **Usage Tracking Dashboard** - Real-time monitoring with alerts

- **Invoice Management** - Direct download and bulk operations

- **Payment Method Management** - In-app card updates

- **Failed Payment Recovery** - One-click retry workflow

- **Cancellation Flow** - Feedback collection and retention offers

- **Budget Controls** - Spending limits and alerts

### �� Coming Soon (Q2-Q4 2026)

- **Custom Pricing** - Enterprise contracts and volume discounts (Q2 2026)

- **Multi-Currency Support** - EUR, GBP, CAD, AUD with local payment methods (Q3 2026)

- **Advanced Analytics** - Usage forecasting and cost optimization (Q4 2026)

- **Flexible Payment Options** - ACH, wire transfer, purchase orders (Q2 2026)

- **Revenue Recognition** - Automated ASC 606 compliance (Q3 2026)

- **Subscription Pause** - Temporary pause with reduced rates (Q4 2026)

- **Referral Credits** - Referral program with account credits (Q3 2026)

- **Advanced Dunning** - Smart retry logic and recovery campaigns (Q4 2026)

---

## Admin Capabilities

### Plan Management

- **Create/Edit Plans**: Define limits, pricing, and Stripe mapping via Admin Dashboard.
- **Lifecycle Control**: Activate/Deactivate plans (seasonal offers, legacy tiers).
- **Pricing Strategy**: Manage displayed pricing independent of Stripe charge logic.

### Subscription Oversight

- **View Tenant Subscriptions**: Monitor status, plan, and MRR.
- **Cancel/Refund**: Admin controls for support cases.
- **Manual Overrides**: Grant complimentary access or extended trials.

---

## Integration Philosophy

### Stripe Checkout (Hosted)

We use Stripe Checkout for all payment flows to ensure:

- **PCI Compliance**: No card data touches our servers.
- **Conversion**: Mobile-optimized, trusted UI.
- **Maintenance**: Zero custom form code.

### Dashboard-First

Users manage their own billing via Stripe Customer Portal:

- Update cards
- Download invoices
- Cancel subscriptions

### Webhooks as Truth

Database state is strictly driven by Stripe webhooks (`checkout.session.completed`, `customer.subscription.updated`), ensuring perfect sync with actual payment status.

---

## Roadmap Summary

### Q1 2026: MVP Completion

**Focus:** Complete critical user-facing billing features

- Plan upgrade/downgrade UI with visual comparison

- Real-time usage tracking dashboard

- Invoice management with direct download

- In-app payment method updates

- Failed payment recovery workflow

- Subscription cancellation with retention offers

- Budget controls and spending limits

**Effort:** 4-7 weeks
**Priority:** P0-P2 (MVP blockers)

### Q2 2026: Enterprise Features

**Focus:** Custom pricing and enterprise billing

- Custom pricing configuration per tenant

- Volume discount tiers

- Custom billing cycles (quarterly, annual)

- Custom payment terms (Net 30, Net 60)

- Contract management with renewals

- Flexible payment options (ACH, wire transfer, PO)

**Effort:** 4-5 weeks
**Priority:** P1-P2 (Enterprise expansion)

### Q3 2026: International & Advanced

**Focus:** Multi-currency and advanced analytics

- Multi-currency support (EUR, GBP, CAD, AUD)

- Local payment methods (SEPA, iDEAL)

- Currency-specific invoicing with local tax

- Revenue recognition automation (ASC 606)

- Referral credits and promotional discounts

- Advanced usage analytics and forecasting

**Effort:** 10-12 weeks
**Priority:** P1-P2 (International expansion)

### Q4 2026: Optimization & Retention

**Focus:** Subscription flexibility and retention

- Subscription pause with reduced rates

- Advanced dunning management

- Smart retry logic based on failure reason

- Multi-channel dunning (email, SMS, in-app)

- Payment recovery campaigns

- Predictive churn prevention

**Effort:** 8-10 weeks
**Priority:** P2 (Retention optimization)

---

## Third-Party Dependencies

### Stripe (Payment Processing)

**Status:** Fully integrated
**Cost:** 2.9% + $0.30 per transaction (~$462/month at MVP)
**Replacement Plan:** Not recommended (PCI compliance complexity)

## Current Usage

- Stripe Checkout for payment collection

- Subscription management and billing

- Customer Portal for self-service

- Webhook integration for real-time sync

- Invoice generation and delivery

### Loop.so (Transactional Emails)

**Status:** Active
**Cost:** $29/month (up to 50,000 emails)
**Replacement Plan:** In-house SMTP (Q3 2026)

## Current Usage

- Payment failure notifications

- Invoice delivery emails

- Usage alert emails

- Subscription change confirmations

- Cancellation emails

**Migration Target:** Q3 2026 (Post-MVP)
**Rationale:** Cost-effective for MVP, replace at scale (10,000+ customers)

---

## Cross-References

### Route Specifications

- **[Settings Routes](/docs/design/routes/settings)** - `/dashboard/settings/billing` UI specification

- **[Platform Admin Routes](/docs/design/routes/platform-admin)** - `/dashboard/finance` for finance team

### Implementation Tasks

- **Epic 4: Billing, Plans & Subscriptions** - Internal task reference for implementation sequencing

  - Milestone 1: Plans & Subscriptions Schema

  - Milestone 2: Stripe Checkout Integration

  - Milestone 3: Subscription State Synchronization

### API Documentation

- **[Subscriptions API](/docs/implementation-technical/api/platform-api/subscriptions)** - Subscription management endpoints

- **[Billing API](/docs/implementation-technical/api/platform-api/billing)** - Billing and payment endpoints

### User Journeys

- **Startup Founder Onboarding** - Includes plan selection (internal journey reference)

- **Agency Owner Onboarding** - Multi-workspace billing (internal journey reference)

### Business Documentation

- **[Pricing Strategy](/docs/business/strategy/pricing-strategy)** - Plan pricing and positioning

- **[Financial Model](/docs/business/financial-analysis/financial-model)** - Revenue projections

### Detailed Roadmap

- **[Billing & Subscriptions Roadmap](/docs/features/payments/roadmap)** - Complete feature timeline with quarterly breakdown

---

[← Back to All Features](https://github.com/penguinmails/penguinmails.github.io/blob/main)
