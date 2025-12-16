---
title: Stripe Integration Overview
description: Introduction to PenguinMails Stripe payment integration, payment flow, and core checkout philosophy
last_modified_date: 2025-01-10
level: 2
persona: developer, product-manager
keywords: [stripe, payments, checkout, subscription, billing, integration]
---

# Stripe Integration Overview

PenguinMails uses [Stripe](https://stripe.com) as our exclusive payment processor, handling all subscription billing, payment processing, and invoice management. This integration ensures secure, reliable payment handling while maintaining PCI DSS compliance.

## Why Stripe?

Stripe provides the security, reliability, and features PenguinMails needs.

- **PCI Compliant** - Stripe handles all card data, so we never store sensitive payment information
- **Global Payments** - Support for 135+ currencies and local payment methods
- **Subscription Billing** - Built-in recurring billing with automatic retries
- **Customer Portal** - Self-service subscription management for users
- **Webhook Events** - Real-time notifications for payment events

## Payment Flow

The complete payment journey from plan selection to active subscription demonstrates how Stripe Checkout streamlines conversion.

### Step 1: Plan Selection

Users browse available plans on the pricing page. Each plan displays features, limits, and pricing. When a user clicks "Subscribe" or "Upgrade," we create a Stripe Checkout Session.

### Step 2: Stripe Checkout

The user is redirected to Stripe's hosted checkout page where they enter payment details in Stripe's secure, PCI-compliant environment. Address collection and tax calculation happen automatically.

### Step 3: Payment Processing

Stripe processes the payment, handles 3D Secure authentication if required, and creates the subscription. The webhook event `checkout.session.completed` fires.

### Step 4: Account Provisioning

PenguinMails receives the webhook, updates the user's plan, provisions resources (email accounts, domains), and sends a confirmation email.

## Checkout Session Creation

When users subscribe, PenguinMails creates a Stripe Checkout session with their selected plan:

```typescript
import { db } from '@/db/client';
import { organizations, subscriptions } from '@/db/schema';
import Stripe from 'stripe';

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY);

export async function createCheckoutSession(
  organizationId: string,
  priceId: string,
  userId: string
) {
  // Get organization details from NileDB
  const [org] = await db
    .select()
    .from(organizations)
    .where(eq(organizations.id, organizationId));

  if (!org) {
    throw new Error('Organization not found');
  }

  // Create Stripe customer if needed
  let customerId = org.stripeCustomerId;
  if (!customerId) {
    const customer = await stripe.customers.create({
      email: org.billingEmail,
      name: org.name,
      metadata: {
        organizationId: org.id,
        tenantId: org.tenantId, // NileDB tenant tracking
      },
    });
    customerId = customer.id;

    // Store Stripe customer ID
    await db
      .update(organizations)
      .set({ stripeCustomerId: customerId })
      .where(eq(organizations.id, organizationId));
  }

  // Create checkout session
  const session = await stripe.checkout.sessions.create({
    customer: customerId,
    mode: 'subscription',
    payment_method_types: ['card'],
    line_items: [
      {
        price: priceId,
        quantity: 1,
      },
    ],
    success_url: `${process.env.NEXT_PUBLIC_APP_URL}/billing/success?session_id={CHECKOUT_SESSION_ID}`,
    cancel_url: `${process.env.NEXT_PUBLIC_APP_URL}/billing/canceled`,
    subscription_data: {
      metadata: {
        organizationId: org.id,
        userId: userId,
      },
    },
    allow_promotion_codes: true,
    billing_address_collection: 'required',
    customer_update: {
      address: 'auto',
      name: 'auto',
    },
  });

  return { sessionId: session.id, url: session.url };
}
```

## Checkout Integration Component

The frontend checkout component provides a seamless payment experience:

```typescript
'use client';

import { loadStripe } from '@stripe/stripe-js';
import { useState } from 'react';

const stripePromise = loadStripe(
  process.env.NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY!
);

interface CheckoutButtonProps {
  priceId: string;
  planName: string;
  organizationId: string;
}

export function CheckoutButton({
  priceId,
  planName,
  organizationId,
}: CheckoutButtonProps) {
  const [loading, setLoading] = useState(false);

  const handleCheckout = async () => {
    setLoading(true);

    try {
      const response = await fetch('/api/checkout', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ priceId, organizationId }),
      });

      const { sessionId } = await response.json();

      const stripe = await stripePromise;
      await stripe?.redirectToCheckout({ sessionId });
    } catch (error) {
      console.error('Checkout error:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <button
      onClick={handleCheckout}
      disabled={loading}
      className="btn-primary"
    >
      {loading ? 'Loading...' : `Subscribe to ${planName}`}
    </button>
  );
}
```

## Related Documentation

- [Subscription Management](/docs/features/payments/stripe-integration/subscription-management) - Plan updates and payment methods
- [Webhooks & Provisioning](/docs/features/payments/stripe-integration/webhooks-provisioning) - Event handling and resource provisioning
- [Configuration & Security](/docs/features/payments/stripe-integration/configuration-security) - Environment setup and PCI compliance

---

**Document Classification:** Level 2 - Feature Implementation
**Implementation Access:** Developers, Product Managers
