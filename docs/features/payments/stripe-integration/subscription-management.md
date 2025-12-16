---
title: Subscription Management
description: Managing subscriptions, updating payment methods, handling failed payments, and customer portal integration
last_modified_date: 2025-01-10
level: 2
persona: developer, product-manager
keywords: [subscription, billing, payment-methods, upgrade, downgrade, stripe-portal]
---

# Subscription Management

Subscription management enables users to control their billing, update payment methods, and change plans without support intervention. PenguinMails leverages Stripe's Customer Portal for self-service capabilities while providing custom controls for plan-specific features.

## Customer Portal Redirect

The Customer Portal redirect enables self-service subscription management through Stripe's hosted interface:

```typescript
import { db } from '@/db/client';
import { organizations } from '@/db/schema';
import Stripe from 'stripe';

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY);

export async function createPortalSession(organizationId: string) {
  // Get organization with Stripe customer ID
  const [org] = await db
    .select()
    .from(organizations)
    .where(eq(organizations.id, organizationId));

  if (!org?.stripeCustomerId) {
    throw new Error('No billing account found');
  }

  const session = await stripe.billingPortal.sessions.create({
    customer: org.stripeCustomerId,
    return_url: `${process.env.NEXT_PUBLIC_APP_URL}/settings/billing`,
  });

  return { url: session.url };
}
```

## Customer Portal Features

The Stripe Customer Portal provides comprehensive self-service options.

### Invoice Management

- **View Invoices** - Access complete invoice history with downloadable PDFs
- **Download Receipts** - Generate receipts for expense reporting
- **Payment History** - Review all successful and failed payments

### Payment Method Management

- **Add Cards** - Store multiple payment methods for backup
- **Update Default** - Change primary payment method
- **Remove Cards** - Delete outdated payment methods

### Subscription Controls

- **Upgrade Plans** - Move to higher tier with prorated billing
- **Downgrade Plans** - Switch to lower tier at next billing cycle
- **Cancel Subscription** - Initiate cancellation with feedback collection

## Update Payment Method

Payment method updates allow users to add, update, or remove their payment methods:

```typescript
import Stripe from 'stripe';

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY);

export async function updatePaymentMethod(
  customerId: string,
  paymentMethodId: string
) {
  // Attach payment method to customer
  await stripe.paymentMethods.attach(paymentMethodId, {
    customer: customerId,
  });

  // Set as default payment method
  await stripe.customers.update(customerId, {
    invoice_settings: {
      default_payment_method: paymentMethodId,
    },
  });

  return { success: true };
}
```

## Handle Failed Payments

Failed payment handling implements automatic retry logic and grace periods to reduce involuntary churn:

```typescript
import { db } from '@/db/client';
import { subscriptions, organizations } from '@/db/schema';
import { eq } from 'drizzle-orm';

interface FailedPaymentEvent {
  customerId: string;
  subscriptionId: string;
  invoiceId: string;
  attemptCount: number;
  nextRetryDate: Date | null;
}

export async function handleFailedPayment(event: FailedPaymentEvent) {
  const { customerId, subscriptionId, attemptCount, nextRetryDate } = event;

  // Get organization by Stripe customer ID
  const [org] = await db
    .select()
    .from(organizations)
    .where(eq(organizations.stripeCustomerId, customerId));

  if (!org) {
    console.error('Organization not found for customer:', customerId);
    return;
  }

  // Update subscription status
  await db
    .update(subscriptions)
    .set({
      status: 'past_due',
      paymentFailedAt: new Date(),
      retryCount: attemptCount,
      nextRetryAt: nextRetryDate,
    })
    .where(eq(subscriptions.stripeSubscriptionId, subscriptionId));

  // Send notification based on attempt number
  if (attemptCount === 1) {
    await sendEmail({
      to: org.billingEmail,
      template: 'payment-failed-first',
      data: {
        orgName: org.name,
        nextRetry: nextRetryDate,
        updatePaymentUrl: `${process.env.NEXT_PUBLIC_APP_URL}/settings/billing`,
      },
    });
  } else if (attemptCount === 3) {
    await sendEmail({
      to: org.billingEmail,
      template: 'payment-failed-urgent',
      data: {
        orgName: org.name,
        daysRemaining: 7,
        updatePaymentUrl: `${process.env.NEXT_PUBLIC_APP_URL}/settings/billing`,
      },
    });
  }
}
```

## Dunning Management

Dunning management handles subscription recovery through strategic communication.

### Retry Schedule

Stripe automatically retries failed payments on a configurable schedule:

| Attempt | Timing | Action |
|---------|--------|--------|
| 1 | Initial failure | Email notification sent |
| 2 | 3 days later | Reminder email |
| 3 | 5 days later | Urgent notification |
| 4 | 7 days later | Final warning |
| Final | 14 days | Subscription canceled |

### Grace Period

During the grace period, users maintain full access but see billing alerts.

- **Past Due Banner** - Visible in dashboard header
- **Feature Warnings** - Shown before using premium features
- **Email Reminders** - Escalating urgency over time

## Plan Changes

Plan changes handle upgrades and downgrades with proper proration:

```typescript
import Stripe from 'stripe';

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY);

export async function changePlan(
  subscriptionId: string,
  newPriceId: string,
  immediate: boolean = false
) {
  const subscription = await stripe.subscriptions.retrieve(subscriptionId);
  
  const updateParams: Stripe.SubscriptionUpdateParams = {
    items: [
      {
        id: subscription.items.data[0].id,
        price: newPriceId,
      },
    ],
    proration_behavior: immediate ? 'create_prorations' : 'none',
  };

  // For downgrades, apply at period end
  if (!immediate) {
    updateParams.cancel_at_period_end = false;
    updateParams.proration_behavior = 'none';
  }

  const updated = await stripe.subscriptions.update(subscriptionId, updateParams);

  return {
    subscriptionId: updated.id,
    status: updated.status,
    currentPeriodEnd: new Date(updated.current_period_end * 1000),
  };
}
```

## Cancellation Flow

The cancellation flow collects feedback and offers retention options:

```typescript
import { db } from '@/db/client';
import { subscriptions, cancellationFeedback } from '@/db/schema';
import Stripe from 'stripe';

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY);

interface CancellationRequest {
  subscriptionId: string;
  reason: string;
  feedback?: string;
  immediate: boolean;
}

export async function cancelSubscription(request: CancellationRequest) {
  const { subscriptionId, reason, feedback, immediate } = request;

  // Store cancellation feedback
  await db.insert(cancellationFeedback).values({
    subscriptionId,
    reason,
    feedback,
    createdAt: new Date(),
  });

  if (immediate) {
    // Cancel immediately
    await stripe.subscriptions.cancel(subscriptionId);
  } else {
    // Cancel at period end
    await stripe.subscriptions.update(subscriptionId, {
      cancel_at_period_end: true,
    });
  }

  // Update local subscription status
  await db
    .update(subscriptions)
    .set({
      status: immediate ? 'canceled' : 'canceling',
      canceledAt: new Date(),
      cancelReason: reason,
    })
    .where(eq(subscriptions.stripeSubscriptionId, subscriptionId));

  return { success: true };
}
```

## Related Documentation

- [Stripe Integration Overview](/docs/features/payments/stripe-integration/overview) - Payment flow and checkout
- [Webhooks & Provisioning](/docs/features/payments/stripe-integration/webhooks-provisioning) - Event handling
- [Configuration & Security](/docs/features/payments/stripe-integration/configuration-security) - Setup and compliance

---

**Document Classification:** Level 2 - Feature Implementation
**Implementation Access:** Developers, Product Managers
