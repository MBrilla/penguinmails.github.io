---
title: Webhooks & Provisioning
description: Stripe webhook handling, subscription status synchronization, and infrastructure provisioning after payment
last_modified_date: 2025-01-10
level: 2
persona: developer
keywords: [webhooks, provisioning, stripe-events, subscription-sync, infrastructure]
---

# Webhooks & Provisioning

Webhook handling is critical for maintaining accurate subscription state between Stripe and PenguinMails. When payment events occur in Stripe, webhooks notify our system to update local records and provision or deprovision resources accordingly.

## Webhook Event Handler

The primary webhook handler validates and routes all Stripe events:

```typescript
import { db } from '@/db/client';
import { subscriptions, organizations } from '@/db/schema';
import { eq } from 'drizzle-orm';
import Stripe from 'stripe';

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY);

export async function handleWebhook(
  payload: string,
  signature: string
): Promise<{ received: boolean }> {
  let event: Stripe.Event;

  // Verify webhook signature
  try {
    event = stripe.webhooks.constructEvent(
      payload,
      signature,
      process.env.STRIPE_WEBHOOK_SIGNING_SECRET
    );
  } catch (err) {
    throw new Error(`Webhook signature verification failed`);
  }

  // Route to appropriate handler
  switch (event.type) {
    case 'checkout.session.completed':
      await handleCheckoutComplete(event.data.object as Stripe.Checkout.Session);
      break;
    case 'invoice.paid':
      await handleInvoicePaid(event.data.object as Stripe.Invoice);
      break;
    case 'invoice.payment_failed':
      await handlePaymentFailed(event.data.object as Stripe.Invoice);
      break;
    case 'customer.subscription.updated':
      await handleSubscriptionUpdated(event.data.object as Stripe.Subscription);
      break;
    case 'customer.subscription.deleted':
      await handleSubscriptionDeleted(event.data.object as Stripe.Subscription);
      break;
    default:
      console.log(`Unhandled event type: ${event.type}`);
  }

  return { received: true };
}
```

## Checkout Complete Handler

When checkout completes, we create the subscription record and provision resources:

```typescript
async function handleCheckoutComplete(session: Stripe.Checkout.Session) {
  const organizationId = session.metadata?.organizationId;
  const subscriptionId = session.subscription as string;

  if (!organizationId || !subscriptionId) {
    throw new Error('Missing metadata in checkout session');
  }

  // Retrieve full subscription details
  const subscription = await stripe.subscriptions.retrieve(subscriptionId);
  const priceId = subscription.items.data[0].price.id;

  // Create subscription record
  await db.insert(subscriptions).values({
    organizationId,
    stripeSubscriptionId: subscriptionId,
    stripeCustomerId: session.customer as string,
    stripePriceId: priceId,
    status: 'active',
    currentPeriodStart: new Date(subscription.current_period_start * 1000),
    currentPeriodEnd: new Date(subscription.current_period_end * 1000),
    createdAt: new Date(),
  });

  // Provision resources based on plan
  await provisionInfrastructure(organizationId, priceId);

  // Send welcome email
  const [org] = await db
    .select()
    .from(organizations)
    .where(eq(organizations.id, organizationId));

  await sendEmail({
    to: org.billingEmail,
    template: 'subscription-welcome',
    data: {
      orgName: org.name,
      planName: getPlanName(priceId),
    },
  });
}
```

## Subscription Status Sync

Subscription updates from Stripe are synchronized to local records:

```typescript
async function handleSubscriptionUpdated(subscription: Stripe.Subscription) {
  const { id, status, current_period_start, current_period_end, items } = subscription;
  
  await db
    .update(subscriptions)
    .set({
      status: mapStripeStatus(status),
      stripePriceId: items.data[0].price.id,
      currentPeriodStart: new Date(current_period_start * 1000),
      currentPeriodEnd: new Date(current_period_end * 1000),
      updatedAt: new Date(),
    })
    .where(eq(subscriptions.stripeSubscriptionId, id));

  // Handle plan changes
  const [sub] = await db
    .select()
    .from(subscriptions)
    .where(eq(subscriptions.stripeSubscriptionId, id));

  if (sub) {
    await updateResourceLimits(sub.organizationId, items.data[0].price.id);
  }
}

async function handleSubscriptionDeleted(subscription: Stripe.Subscription) {
  await db
    .update(subscriptions)
    .set({
      status: 'canceled',
      canceledAt: new Date(),
      updatedAt: new Date(),
    })
    .where(eq(subscriptions.stripeSubscriptionId, subscription.id));

  // Get organization for deprovisioning
  const [sub] = await db
    .select()
    .from(subscriptions)
    .where(eq(subscriptions.stripeSubscriptionId, subscription.id));

  if (sub) {
    await deprovisionInfrastructure(sub.organizationId);
  }
}

function mapStripeStatus(stripeStatus: string): string {
  const statusMap: Record<string, string> = {
    active: 'active',
    past_due: 'past_due',
    canceled: 'canceled',
    unpaid: 'unpaid',
    trialing: 'trialing',
    incomplete: 'incomplete',
  };
  return statusMap[stripeStatus] || 'unknown';
}
```

## Infrastructure Provisioning

After successful payment, PenguinMails provisions email infrastructure based on plan:

```typescript
import { db } from '@/db/client';
import { emailAccounts, domains } from '@/db/schema';

interface PlanLimits {
  emailAccounts: number;
  domains: number;
  dailySendLimit: number;
  warmupEnabled: boolean;
}

const PLAN_LIMITS: Record<string, PlanLimits> = {
  'price_starter': {
    emailAccounts: 3,
    domains: 1,
    dailySendLimit: 500,
    warmupEnabled: false,
  },
  'price_professional': {
    emailAccounts: 10,
    domains: 5,
    dailySendLimit: 5000,
    warmupEnabled: true,
  },
  'price_enterprise': {
    emailAccounts: 100,
    domains: 25,
    dailySendLimit: 50000,
    warmupEnabled: true,
  },
};

export async function provisionInfrastructure(
  organizationId: string,
  priceId: string
) {
  const limits = PLAN_LIMITS[priceId];
  
  if (!limits) {
    throw new Error(`Unknown price ID: ${priceId}`);
  }

  // Update organization limits
  await db
    .update(organizations)
    .set({
      emailAccountLimit: limits.emailAccounts,
      domainLimit: limits.domains,
      dailySendLimit: limits.dailySendLimit,
      warmupEnabled: limits.warmupEnabled,
      provisionedAt: new Date(),
    })
    .where(eq(organizations.id, organizationId));

  // Queue DNS verification for pending domains
  const pendingDomains = await db
    .select()
    .from(domains)
    .where(and(
      eq(domains.organizationId, organizationId),
      eq(domains.status, 'pending')
    ));

  for (const domain of pendingDomains) {
    await queueDNSVerification(domain.id);
  }

  // Initialize warmup for enabled plans
  if (limits.warmupEnabled) {
    await initializeWarmup(organizationId);
  }
}

export async function updateResourceLimits(
  organizationId: string,
  newPriceId: string
) {
  const limits = PLAN_LIMITS[newPriceId];
  
  if (!limits) return;

  await db
    .update(organizations)
    .set({
      emailAccountLimit: limits.emailAccounts,
      domainLimit: limits.domains,
      dailySendLimit: limits.dailySendLimit,
      warmupEnabled: limits.warmupEnabled,
    })
    .where(eq(organizations.id, organizationId));
}

export async function deprovisionInfrastructure(organizationId: string) {
  // Stop warmup processes
  await stopWarmup(organizationId);

  // Disable sending
  await db
    .update(organizations)
    .set({
      dailySendLimit: 0,
      warmupEnabled: false,
      deprovisionedAt: new Date(),
    })
    .where(eq(organizations.id, organizationId));

  // Mark email accounts as inactive
  await db
    .update(emailAccounts)
    .set({ status: 'inactive' })
    .where(eq(emailAccounts.organizationId, organizationId));
}
```

## Database Schema

The billing-related database schema in Drizzle ORM:

```typescript
import { pgTable, text, timestamp, integer, boolean } from 'drizzle-orm/pg-core';

export const subscriptions = pgTable('subscriptions', {
  id: text('id').primaryKey().default(sql`gen_random_uuid()`),
  organizationId: text('organization_id').notNull().references(() => organizations.id),
  stripeSubscriptionId: text('stripe_subscription_id').notNull().unique(),
  stripeCustomerId: text('stripe_customer_id').notNull(),
  stripePriceId: text('stripe_price_id').notNull(),
  status: text('status').notNull().default('active'),
  currentPeriodStart: timestamp('current_period_start').notNull(),
  currentPeriodEnd: timestamp('current_period_end').notNull(),
  canceledAt: timestamp('canceled_at'),
  cancelReason: text('cancel_reason'),
  paymentFailedAt: timestamp('payment_failed_at'),
  retryCount: integer('retry_count').default(0),
  nextRetryAt: timestamp('next_retry_at'),
  createdAt: timestamp('created_at').defaultNow(),
  updatedAt: timestamp('updated_at').defaultNow(),
});

export const cancellationFeedback = pgTable('cancellation_feedback', {
  id: text('id').primaryKey().default(sql`gen_random_uuid()`),
  subscriptionId: text('subscription_id').notNull(),
  reason: text('reason').notNull(),
  feedback: text('feedback'),
  createdAt: timestamp('created_at').defaultNow(),
});
```

## Webhook Events Reference

Key Stripe webhook events and their handlers:

| Event | Description | Handler Action |
|-------|-------------|----------------|
| `checkout.session.completed` | User completed checkout | Create subscription, provision resources |
| `invoice.paid` | Invoice payment succeeded | Update billing records, send receipt |
| `invoice.payment_failed` | Payment attempt failed | Start dunning, notify user |
| `customer.subscription.created` | New subscription created | Record subscription details |
| `customer.subscription.updated` | Subscription modified | Sync status, update limits |
| `customer.subscription.deleted` | Subscription canceled | Deprovision resources, cleanup |

## Related Documentation

- [Stripe Integration Overview](/docs/features/payments/stripe-integration/overview) - Payment flow and checkout
- [Subscription Management](/docs/features/payments/stripe-integration/subscription-management) - Plan changes and cancellation
- [Configuration & Security](/docs/features/payments/stripe-integration/configuration-security) - Setup and compliance

---

**Document Classification:** Level 2 - Feature Implementation
**Implementation Access:** Developers
