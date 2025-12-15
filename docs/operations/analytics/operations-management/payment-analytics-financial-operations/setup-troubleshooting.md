---
title: "Setup & Troubleshooting"
description: "Initial setup scenarios, common payment issues, and cross-reference integration for payment operations"
last_modified_date: "2025-12-15"
level: "3"
parent: "Payment Analytics & Financial Operations"
---

# Setup & Troubleshooting

This guide covers initial setup scenarios, common payment issues and solutions, and cross-reference integration.

---

## Initial Setup Scenarios

### First-Time Stripe Connect Setup

```markdown
Onboarding Flow → Stripe OAuth → Business Info → Bank Details → Verification → Success
```

**Detailed Steps:**

1. **Onboarding Context**:
   - **Trigger**: User reaches Step 3 of onboarding
   - **Education**: Modal explaining B2B payment model and fees
   - **CTA**: "Connect Stripe Account" button

2. **Stripe Hosted Flow**:
   - **Redirect**: User sent to Stripe's secure onboarding
   - **Business Info**: Legal name, address, tax ID collection
   - **Banking**: Bank account or debit card setup
   - **Verification**: Identity verification process

3. **Return & Activation**:
   - **Redirect**: Back to PenguinMails with success token
   - **Validation**: Account verification status checked
   - **Feature Unlock**: Billing features activated

### Payment Method Addition

```markdown
Billing Settings → Add Method → Card Form → 3DS Verification → Confirmation → Default Set
```

**Detailed Steps:**

1. **Access Point**:
   - **Page**: Billing dashboard
   - **Element**: "Add Payment Method" button or empty state prompt

2. **Card Collection**:
   - **Form**: Stripe Elements secure card input
   - **Validation**: Real-time card number/format checking
   - **Address**: Billing address collection (AVS compliance)

3. **Verification**:
   - **3D Secure**: Additional authentication when required
   - **Micro-deposits**: Alternative verification for bank accounts
   - **Confirmation**: Success message and method listing

---

## Troubleshooting: Payment & Billing Issues

### "Why was my payment declined?"

#### 1. Insufficient Funds

- **Symptom**: "Payment declined" with bank error code
- **Solution**: Check account balance, add alternative payment method
- **Prevention**: Set up payment failure notifications

#### 2. Card Expired or Invalid

- **Symptom**: "Card expired" or "Invalid card number"
- **Solution**: Update card details in billing settings
- **Prevention**: Update cards before expiration

#### 3. Security Block

- **Symptom**: Bank fraud protection triggered
- **Solution**: Contact bank to whitelist PenguinMails, try different card
- **Prevention**: Use business cards for business subscriptions

### "Why am I being charged the wrong amount?"

#### 1. Prorated Billing

- **Symptom**: Unexpected partial charges on plan changes
- **Solution**: Check billing history for upgrade/downgrade details
- **Explanation**: Proration calculates unused time credit

#### 2. Usage Overages

- **Symptom**: Extra charges beyond base plan
- **Solution**: Review usage metrics in billing dashboard
- **Prevention**: Monitor usage alerts and plan limits

#### 3. Currency Conversion

- **Symptom**: Charges in unexpected currency
- **Solution**: Update billing country in account settings
- **Prevention**: Set correct billing address during setup

### "Why can't I change my payment method?"

#### 1. Account Verification Pending

- **Symptom**: Payment method changes disabled
- **Solution**: Complete Stripe Connect business verification
- **Status**: Check verification status in billing dashboard

#### 2. Outstanding Balance

- **Symptom**: "Clear outstanding balance first"
- **Solution**: Pay any overdue invoices before changing methods
- **Prevention**: Keep payment methods current

#### 3. Failed Payment Retry

- **Symptom**: Changes blocked during retry period
- **Solution**: Wait for retry attempts to complete or update existing method
- **Prevention**: Keep valid payment methods on file

---

## Quick Reference: Common Issues

| Issue | Likely Cause | Quick Fix |
|-------|--------------|-----------|
| Payment declined | Insufficient funds | Add backup payment method |
| Wrong charge amount | Proration | Review billing history |
| Can't add card | 3DS failure | Try different browser/card |
| Invoice missing | Email filter | Check spam/promotions folder |
| Subscription not active | Payment pending | Complete payment method setup |

---

## Cross-Reference Integration (Canonical Alignment)

### Operations & Analytics

- [`docs/operations-analytics/overview`](/docs/operations/analytics) - Global operations analytics framework
- [`docs/operations-analytics/analytics-performance/metrics-kpis`](/docs/operations/analytics/analytics-performance) - Core KPI definitions for revenue, churn, and billing performance
- [`docs/operations-analytics/operations-management/organization-analytics-team-management`](/docs/operations/analytics/operations-management) - Team and organization management analytics
- [`docs/operations-analytics/operations-management/environment-release-management`](/docs/operations/analytics/operations-management) - Environment and release operations impact on billing and reliability

### Business Strategy

- [`docs/business/model/overview`](/docs/business/model) - Canonical business and revenue model
- [`docs/business/value-proposition/overview`](/docs/business/value-proposition) - Value proposition framing for pricing and packaging
- [`docs/business/strategy/overview`](/docs/business/strategy) - Strategic priorities that payment analytics must support

### Technical Architecture

- [`docs/technical/architecture/overview`](/docs/technical/architecture) - High-level system architecture
- [`docs/technical/architecture/detailed-technical/integration-guide`](/docs/technical/architecture/detailed-technical) - Canonical integration patterns, including billing/Stripe
- [`docs/implementation-technical/development-guidelines/api-reference`](/docs/implementation-technical/development-guidelines) - API surface for billing, subscriptions, and webhooks

### Compliance & Security

- [`docs/compliance-security/overview`](/docs/compliance-security) - Compliance posture
- [`docs/compliance-security/enterprise/security-framework`](/docs/compliance-security/enterprise) - Security controls relevant to financial data
- [`docs/compliance-security/international/data-privacy-policy`](/docs/compliance-security/international) - Data handling, retention, and privacy for billing records

*This section is authoritative; legacy content references should be treated as non-canonical historical scaffolding.*

---

## Next Steps

Navigate to specific payment and financial areas:

- **[Payment & Billing Journey](./payment-billing-journey)** - Complete payment flows
- **[Stripe Connect Integration](./stripe-connect-integration)** - Technical implementation
- **[Edge Cases & Recovery](./edge-cases-recovery)** - Error handling and recovery

---

**Keywords**: setup guide, troubleshooting, payment issues, billing problems, Stripe setup, cross-reference
