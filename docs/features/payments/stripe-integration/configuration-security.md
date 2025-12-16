---
title: Configuration & Security
description: Stripe environment configuration, setup steps, PCI DSS compliance, and SCA requirements
last_modified_date: 2025-01-10
level: 2
persona: developer, operations
keywords: [stripe-config, environment-variables, pci-compliance, sca, security, webhooks]
---

# Configuration & Security

Proper configuration is essential for secure Stripe integration. This document covers environment variables, setup procedures, PCI DSS compliance, and Strong Customer Authentication (SCA) requirements.

## Environment Configuration

### Required Environment Variables

PenguinMails requires the following Stripe environment variables:

```bash
# Stripe API Keys
STRIPE_SECRET_KEY=sk_live_xxxxx...  # or sk_test_xxxxx for testing
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_live_xxxxx...  # or pk_test_xxxxx

# Webhook Configuration
STRIPE_WEBHOOK_SIGNING_SECRET=whsec_xxxxx...

# Application URLs
NEXT_PUBLIC_APP_URL=https://app.penguinmails.com  # or http://localhost:3000 for development
```

## Setup Steps

### 1. Create Stripe Account

- Sign up at [stripe.com](https://stripe.com)
- Complete business verification (required for live mode)
- Enable Customer Portal in Stripe Dashboard → Settings → Customer Portal

### 2. Retrieve API Keys

- Navigate to Stripe Dashboard → Developers → API keys
- Copy "Secret key" to `STRIPE_SECRET_KEY`
- Copy "Publishable key" to `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY`

### 3. Configure Webhooks

- Go to Stripe Dashboard → Developers → Webhooks
- Click "Add endpoint"
- Endpoint URL: `https://api.penguinmails.com/webhooks/stripe`
- Select events to listen for:
  - `checkout.session.completed`
  - `invoice.paid`
  - `invoice.payment_failed`
  - `customer.subscription.created`
  - `customer.subscription.updated`
  - `customer.subscription.deleted`
- Copy "Signing secret" to `STRIPE_WEBHOOK_SIGNING_SECRET`

### 4. Test Configuration

```bash
# Install Stripe CLI for local testing
brew install stripe/stripe-cli/stripe

# Login to Stripe
stripe login

# Forward webhooks to local development
stripe listen --forward-to localhost:3000/api/webhooks/stripe

# Test webhook delivery
stripe trigger checkout.session.completed
```

## API Endpoints

### Get Current Subscription

```markdown
GET /api/v1/billing/subscription
Authorization: Bearer {api_key}

Response:
{
  "subscription": {
    "id": "sub_xyz789",
    "plan": "Professional",
    "status": "active",
    "current_period_end": "2025-12-24T00:00:00Z",
    "cancel_at_period_end": false
  },
  "usage": {
    "email_accounts": { "used": 5, "limit": 10 },
    "domains": { "used": 2, "limit": 5 },
    "emails_sent_today": 1250
  }
}
```

### Get Payment History

```markdown
GET /api/v1/billing/payments
Authorization: Bearer {api_key}

Response:
{
  "payments": [
    {
      "id": "pi_abc123",
      "amount": 149.00,
      "currency": "USD",
      "status": "succeeded",
      "description": "Professional Plan - Monthly",
      "created": "2025-11-24T10:30:00Z",
      "receipt_url": "https://stripe.com/receipts/..."
    }
  ]
}
```

## PCI Compliance

PenguinMails is PCI DSS compliant by using Stripe for all payment processing.

### Compliance Measures

- **No card data stored** - Stripe handles all card information
- **Tokenization** - Only store Stripe tokens, never raw card data
- **Secure transmission** - TLS 1.3 encryption for all API communication
- **Annual audits** - Stripe maintains PCI DSS Level 1 compliance

### Data Handling

PenguinMails stores only:

| Data Type | Stored | Purpose |
|-----------|--------|---------|
| Stripe Customer ID | Yes | Link account to Stripe customer |
| Stripe Subscription ID | Yes | Track subscription status |
| Last 4 digits | Yes | Display payment method to user |
| Card brand | Yes | Display payment method to user |
| Full card number | No | Never stored or transmitted |
| CVV | No | Never stored or transmitted |
| Expiration date | No | Retrieved from Stripe when needed |

## Strong Customer Authentication (SCA)

European regulation compliance for card payments is handled automatically by Stripe Checkout.

### SCA Features

- **3D Secure** - Built into Stripe Checkout flow
- **Automatic handling** - Stripe manages SCA requirements based on card issuer
- **Fallback methods** - Alternative authentication methods if 3D Secure unavailable
- **Exemption handling** - Stripe applies exemptions where applicable

### Implementation

SCA is automatically enforced through Stripe Checkout:

```typescript
// Stripe Checkout handles SCA automatically
const session = await stripe.checkout.sessions.create({
  mode: 'subscription',
  payment_method_types: ['card'],
  // SCA handled by Stripe based on customer location and card issuer
  ...
});
```

## Security Best Practices

### API Key Security

- Store keys in environment variables, never in code
- Use test keys (`sk_test_*`) for development
- Rotate keys periodically (Stripe Dashboard → Developers → API keys)
- Restrict API key permissions when possible

### Webhook Security

- Always verify webhook signatures
- Use HTTPS endpoints only
- Implement idempotency for event handling
- Log all webhook events for debugging

```typescript
// Webhook signature verification
try {
  event = stripe.webhooks.constructEvent(
    payload,
    signature,
    process.env.STRIPE_WEBHOOK_SIGNING_SECRET
  );
} catch (err) {
  console.error('Webhook signature verification failed:', err.message);
  throw new Error('Invalid webhook signature');
}
```

### Error Handling

```typescript
try {
  await stripe.subscriptions.create({...});
} catch (err) {
  if (err instanceof Stripe.errors.StripeCardError) {
    // Card was declined
    return { error: 'Your card was declined. Please try another card.' };
  } else if (err instanceof Stripe.errors.StripeRateLimitError) {
    // Too many requests
    return { error: 'Too many requests. Please try again later.' };
  } else if (err instanceof Stripe.errors.StripeInvalidRequestError) {
    // Invalid parameters
    console.error('Invalid Stripe request:', err.message);
    return { error: 'An error occurred. Please try again.' };
  }
  throw err;
}
```

## Related Documentation

- [Stripe Integration Overview](/docs/features/payments/stripe-integration/overview) - Payment flow and checkout
- [Subscription Management](/docs/features/payments/stripe-integration/subscription-management) - Plan changes and cancellation
- [Webhooks & Provisioning](/docs/features/payments/stripe-integration/webhooks-provisioning) - Event handling

---

**Last Updated:** November 24, 2025
**Stripe API Version:** 2023-10-16
**Compliance:** PCI DSS Level 1, SCA compliant

*All payments are processed securely by Stripe. PenguinMails never stores credit card information.*

---

**Document Classification:** Level 2 - Feature Implementation
**Implementation Access:** Developers, Operations
