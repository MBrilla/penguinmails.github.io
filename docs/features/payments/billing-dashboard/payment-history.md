---
title: "Payment History & Budget Management"
description: "Transaction log, failed payments, refunds, and budget controls"
last_modified_date: "2025-11-24"
level: 2
persona: "Billing Administrators, Finance Teams"
keywords: ["payment-history", "budget-management", "refunds", "billing-api"]
---

# Payment History & Budget Management

Complete payment history, failed payment handling, refunds, and budget controls.

## Transaction Log

```javascript
GET /api/v1/billing/payments
Authorization: Bearer {api_key}

Response:
{
  "payments": [
    {
      "id": "pi_abc123",
      "date": "2025-11-24",
      "amount": 149.00,
      "currency": "USD",
      "status": "succeeded",
      "description": "Professional Plan - Monthly",
      "payment_method": "card_4242",
      "receipt_url": "https://stripe.com/receipts/..."
    }
  ],
  "total_count": 12,
  "page": 1
}
```

**Transaction Details**:
- Payment date and time
- Amount charged
- Payment method used
- Payment status (succeeded, failed, refunded)
- Receipt link
- Associated invoice

## Failed Payment Handling

**What happens when payment fails**:

1. **Immediate Notification**: Email to billing admin, in-app alert, dashboard warning banner
2. **Grace Period** (7 days): Service continues, automatic retry attempts, update payment method anytime
3. **Account Suspension** (Day 7): Email sending disabled, read-only access to data, final retry attempt
4. **Cancellation** (Day 14): Subscription canceled, free tier activated (90-day data retention)

**Resolution**:
- Update Payment Method button prominently displayed
- Retry Payment manual retry option
- Contact support for assistance

## Refunds

**Eligible for Refund**:
- ✅ Double-charged (full refund)
- ✅ Billing error (full refund)
- ✅ Service issue (prorated refund)
- ✅ Within 7 days of charge (goodwill refund)

**Not Eligible**:
- ❌ Simply changed mind after 7 days
- ❌ Service not used (non-usage doesn't qualify)
- ❌ Annual plans mid-term (no partial refunds)

**Request Refund**:
1. Contact support: billing@penguinmails.com
2. Provide invoice number and reason
3. Review by billing team (1-2 business days)
4. Refund processed (5-10 business days to card)

## Budget Management

### Spending Limits

```text
💰 Budget Controls

Monthly Budget: $200.00
Current Month Spend: $159.00
Remaining Budget: $41.00

[●●●●●●●●●●●●●●●●●●●○○○○○○] 79.5%

Alert When:
☑ 80% of budget ($160.00)
☑ 100% of budget ($200.00)

Action at 100%:
● Alert only
○ Block overage charges
○ Auto-upgrade to next tier
```

### Cost Optimization Tips

Dashboard provides recommendations:
- 💡 **Switch to Annual** - Save 20% ($358/year savings)
- 💡 **Downgrade Unused Seats** - Remove 2 inactive users ($30/mo savings)
- 💡 **Archive Old Workspaces** - Reduce workspace count
- 💡 **Optimize Send Frequency** - Reduce emails by segmentation

## Billing Administration

### Billing Contact

```text
Billing Contacts

Primary: billing@acme.com [Edit]

Additional Recipients:
finance@acme.com [Remove]
cfo@acme.com [Remove]

[+ Add Recipient]
```

### Company Information

```text
Company Information

Legal Name: Acme Corporation
Tax ID: 12-3456789
Billing Address:
  123 Main Street
  San Francisco, CA 94105
  United States

[Edit Information]
```

## API Access

```javascript
// Get Current Usage
GET /api/v1/billing/usage

// Get Invoices
GET /api/v1/billing/invoices?limit=10

// Download Invoice PDF
GET /api/v1/billing/invoices/{invoice_id}/pdf

// Get Payment History
GET /api/v1/billing/payments?page=1&per_page=20

// Update Billing Info
PUT /api/v1/billing/company-info
```

## Related Documentation

- **[Overview](overview)** - Dashboard overview
- **[Subscription Management](/docs/features/payments/subscription-management)** - Plan upgrades and downgrades
- **[Stripe Integration](/docs/features/payments/stripe-integration)** - Payment processing

---

**Last Updated**: November 24, 2025
**Access Level**: Billing Admin, Tenant Owner
**Data Retention**: Invoices retained for 7 years
