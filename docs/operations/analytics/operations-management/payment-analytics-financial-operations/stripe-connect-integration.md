---
title: "Stripe Connect Integration & Operations"
description: "Technical architecture for Stripe Connect onboarding, account management, and webhook integration"
last_modified_date: "2025-12-15"
level: "3"
parent: "Payment Analytics & Financial Operations"
---

# Stripe Connect Integration & Operations

This guide describes the complete Stripe Connect onboarding and billing integration for PenguinMails. The system follows an OLTP-first architecture with NileDB for secure financial data storage and Stripe for payment processing.

---

## Technical Architecture Overview

### User Flow Architecture

#### 1. User Signup and Authentication

**Entry Point**: `/signup`

- User signs up using NileDB authentication
- Creates tenant user account automatically
- Redirects to dashboard on successful signup

**Key Components**:
- `app/signup/` - Signup pages
- `lib/niledb/auth.ts` - NileDB authentication utilities
- `components/auth/` - Authentication components

#### 2. Dashboard Entry

**Entry Point**: `/dashboard`

- User lands in main dashboard after signup
- Same user account available in admin panel as tenant user
- Sidebar includes "Setup Guide" link to onboarding

**Key Components**:
- `components/layout/DashboardSidebar.tsx` - Navigation with onboarding link
- `app/dashboard/` - Dashboard pages

#### 3. Onboarding Experience

**Entry Point**: `/dashboard/onboarding`

- Multi-step onboarding flow for new users
- Includes billing setup as one of the steps
- Uses context-based state management

#### 4. Stripe Connect Setup Step

**New Onboarding Step**: "Connect Payment Processor"

This step should be added to the onboarding flow with strategic positioning and enterprise-grade security.

---

## Stripe Connect Integration

### 1. Account Creation Flow

```typescript
// API Endpoint: POST /api/stripe/connect
export async function createStripeConnectAccount(companyId: string, billingEmail?: string) {
  try {
    // 1. Check if account already exists in tenant_config
    const existingAccount = await nile.db.query(
      `SELECT stripe_account_id FROM tenant_config
       WHERE tenant_id = CURRENT_TENANT_ID()`,
      []
    );

    if (existingAccount.rows.length > 0 && existingAccount.rows[0].stripe_account_id) {
      return { accountId: existingAccount.rows[0].stripe_account_id };
    }

    // 2. Create Stripe Connect Express account
    const account = await stripe.accounts.create({
      type: 'express',
      country: 'US', // Default, should be configurable
      email: billingEmail,
      capabilities: {
        card_payments: { requested: true },
        transfers: { requested: true },
      },
    });

    // 3. Store account ID in tenant_config
    await nile.db.query(
      `UPDATE tenant_config SET stripe_account_id = $1, updated = CURRENT_TIMESTAMP
       WHERE tenant_id = CURRENT_TENANT_ID()`,
      [account.id]
    );

    return { accountId: account.id };
  } catch (error) {
    console.error('Failed to create Stripe Connect account:', error);
    throw error;
  }
}
```

### 2. Onboarding Link Generation

```typescript
// API Endpoint: GET /api/stripe/connect/onboarding-link
export async function getStripeOnboardingLink(accountId: string) {
  try {
    const accountLink = await stripe.accountLinks.create({
      account: accountId,
      refresh_url: `${process.env.NEXT_PUBLIC_APP_URL}/dashboard/settings/billing`,
      return_url: `${process.env.NEXT_PUBLIC_APP_URL}/dashboard/settings/billing`,
      type: 'account_onboarding',
    });

    return { url: accountLink.url };
  } catch (error) {
    console.error('Failed to create onboarding link:', error);
    throw error;
  }
}
```

---

## Database Schema

**Minimal Schema Changes**: Add to existing `tenant_config` table

```sql
-- Add to existing tenant_config table for tenant-level Stripe accounts
ALTER TABLE tenant_config ADD COLUMN stripe_account_id VARCHAR(255) UNIQUE;

-- Platform Stripe account stored in system_config (single row, key='stripe_account_id')
-- This is separate from tenant customer accounts in tenant_config.stripe_customer_id

-- Existing tables cover all other billing needs:
-- - subscriptions: subscription management
-- - payments: payment history and billing
-- - plans: plan definitions and limits
-- - subscription_addons: additional features
-- - Usage calculated at runtime from campaigns, emails, companies tables
```

---

## Settings/Billing Integration

### Status Checking Logic

```typescript
// API Endpoint: GET /api/stripe/connect/status
export async function getStripeConnectStatus() {
  try {
    // Get stored account info from tenant_config
    const accountInfo = await nile.db.query(
      `SELECT stripe_account_id FROM tenant_config
       WHERE tenant_id = CURRENT_TENANT_ID()`,
      []
    );

    if (!accountInfo.rows.length || !accountInfo.rows[0].stripe_account_id) {
      return { connected: false, status: 'not_connected' };
    }

    const stripeAccountId = accountInfo.rows[0].stripe_account_id;

    // Retrieve account status from Stripe
    const stripeAccount = await stripe.accounts.retrieve(stripeAccountId);

    return {
      connected: true,
      status: stripeAccount.details_submitted ? 'complete' : 'pending',
      chargesEnabled: stripeAccount.charges_enabled,
      payoutsEnabled: stripeAccount.payouts_enabled,
      requirements: stripeAccount.requirements,
    };
  } catch (error) {
    console.error('Failed to get Stripe Connect status:', error);
    throw error;
  }
}
```

---

## Webhook Integration

### 1. Webhook Endpoint

**Endpoint**: `POST /api/webhooks/stripe`

Handles account updates, capability changes, and other Stripe events for real-time status synchronization.

### 2. Event Handlers

- **Account Updates**: Process account information changes
- **Capability Updates**: Handle payment capability status changes
- **Payment Events**: Track payment processing and failures

---

## Next Steps

- **[Payment & Billing Journey](./payment-billing-journey)** - User-facing payment flows
- **[Financial Analytics Framework](./financial-analytics-framework)** - Revenue and cost tracking
- **[Edge Cases & Recovery](./edge-cases-recovery)** - Error handling and recovery

---

**Keywords**: Stripe Connect, API integration, webhook, account creation, billing integration, NileDB
