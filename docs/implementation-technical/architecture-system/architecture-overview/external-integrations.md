---
title: "External Integrations"
last_modified_date: "2025-12-15"
level: "2"
persona: "Technical Leads, Developers"
---

# External Integrations

Hostwinds VPS, MailU SMTP, Stripe billing, and NileDB integration.

## VPS Provider Integration (Hostwinds)

| Aspect | Details |
|--------|---------|
| **Purpose** | Automated VPS provisioning and management |
| **Integration** | REST API with webhooks |
| **Features** | Geographic IP distribution, automatic scaling |

### Provisioning Flow

```text
1. Customer requests new inbox
2. API call to Hostwinds to provision VPS
3. VPS created with specifications
4. SSH access configured
5. Mail server installation triggered
6. DNS records created
7. Inbox ready for use
```

## SMTP Server Integration (MailU)

| Aspect | Details |
|--------|---------|
| **Purpose** | Email sending and receiving |
| **Components** | Postfix, Dovecot, Roundcube |
| **Features** | SPF/DKIM/DMARC, spam filtering |

### Mail Server Stack

```text
┌─────────────────────────────────────┐
│           MailU Stack               │
├─────────────────────────────────────┤
│  Postfix (MTA)      - Email sending │
│  Dovecot (MDA)      - Email storage │
│  Roundcube (Webmail)- Web access    │
│  SpamAssassin       - Spam filter   │
│  OpenDKIM           - DKIM signing  │
└─────────────────────────────────────┘
```

## Billing Integration (Stripe)

| Aspect | Details |
|--------|---------|
| **Purpose** | Subscription and payment management |
| **Integration** | Stripe SDK with webhooks |
| **Features** | Subscriptions, usage billing, invoices |

### Webhook Events

| Event | Action |
|-------|--------|
| `customer.subscription.created` | Provision resources |
| `customer.subscription.updated` | Adjust limits |
| `customer.subscription.deleted` | Suspend access |
| `invoice.payment_succeeded` | Confirm payment |
| `invoice.payment_failed` | Send reminder |

## Database Integration (NileDB)

| Aspect | Details |
|--------|---------|
| **Purpose** | Multi-tenant PostgreSQL with built-in isolation |
| **Integration** | NileDB SDK |
| **Features** | Row-level security, tenant isolation |

### Multi-Tenant Architecture

```typescript
// NileDB provides automatic tenant isolation
import { nile } from '@niledatabase/server';

// Queries automatically scoped to tenant
const campaigns = await nile
  .db('campaigns')
  .select('*')
  .where('status', 'active');
// Automatically includes: WHERE tenant_id = <current_tenant>
```

---

[← Back to Architecture Overview](./README)
