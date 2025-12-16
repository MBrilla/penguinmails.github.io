---
title: "OLAP Analytics Tables"
description: "Core analytics table definitions for billing, campaigns, mailboxes, leads, and warmups"
last_modified_date: "2025-11-17"
level: "3"
persona: "Database Teams"
keywords: [olap, billing-analytics, campaign-analytics, mailbox-analytics, lead-analytics]
---

# OLAP Analytics Tables

Core analytics table definitions for the PenguinMails OLAP warehouse.

## Billing Analytics

```sql
CREATE TABLE billing_analytics (
    id BIGINT PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
    tenant_id TEXT NOT NULL,
    subscription_id TEXT,
    emails_sent INTEGER DEFAULT 0,
    mailboxes_used INTEGER DEFAULT 0,
    domains_used INTEGER DEFAULT 0,
    campaigns_used INTEGER DEFAULT 0,
    leads_used INTEGER DEFAULT 0,
    warmups_active INTEGER DEFAULT 0,
    period_start TIMESTAMPTZ NOT NULL,
    period_end TIMESTAMPTZ NOT NULL,
    updated TIMESTAMPTZ DEFAULT NOW()
);

CREATE UNIQUE INDEX idx_billing_analytics_tenant_period
    ON billing_analytics(tenant_id, period_start, period_end);
```

**Purpose:** Aggregated usage per tenant per period. Drives billing, quotas, and revenue analytics.

**Use Cases:**
- Billing usage summaries
- Dashboard widgets (resource counts)
- Executive reports (tenant growth)

**NOT for:**
- Detailed campaign metrics → Use `campaign_analytics`
- Email deliverability insights → Use `mailbox_analytics`
- Lead engagement scoring → Use `lead_analytics`

## Campaign Analytics

```sql
CREATE TABLE campaign_analytics (
    id BIGINT PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
    campaign_id TEXT NOT NULL,
    company_id TEXT NOT NULL,
    sent INTEGER DEFAULT 0,
    delivered INTEGER DEFAULT 0,
    opened_tracked INTEGER DEFAULT 0,
    clicked_tracked INTEGER DEFAULT 0,
    replied INTEGER DEFAULT 0,
    bounced INTEGER DEFAULT 0,
    unsubscribed INTEGER DEFAULT 0,
    spam_complaints INTEGER DEFAULT 0,
    status TEXT,
    completed_leads INTEGER DEFAULT 0,
    billing_id BIGINT REFERENCES billing_analytics(id),
    updated TIMESTAMPTZ DEFAULT NOW()
);
```

**Purpose:** Aggregated per-campaign performance for reporting and optimization.

## Mailbox Analytics

```sql
CREATE TABLE mailbox_analytics (
    id BIGINT PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
    mailbox_id TEXT NOT NULL,
    company_id TEXT NOT NULL,
    sent INTEGER DEFAULT 0,
    delivered INTEGER DEFAULT 0,
    opened_tracked INTEGER DEFAULT 0,
    clicked_tracked INTEGER DEFAULT 0,
    replied INTEGER DEFAULT 0,
    bounced INTEGER DEFAULT 0,
    unsubscribed INTEGER DEFAULT 0,
    spam_complaints INTEGER DEFAULT 0,
    warmup_status TEXT,
    health_score INTEGER DEFAULT 0,
    current_volume INTEGER DEFAULT 0,
    billing_id BIGINT REFERENCES billing_analytics(id),
    campaign_status TEXT,
    updated TIMESTAMPTZ DEFAULT NOW()
);
```

**Purpose:** Mailbox-level deliverability, health, and warmup analytics.

## Lead Analytics

```sql
CREATE TABLE lead_analytics (
    id BIGINT PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
    lead_id TEXT NOT NULL,
    campaign_id TEXT NOT NULL,
    sent INTEGER DEFAULT 0,
    delivered INTEGER DEFAULT 0,
    opened_tracked INTEGER DEFAULT 0,
    clicked_tracked INTEGER DEFAULT 0,
    replied INTEGER DEFAULT 0,
    bounced INTEGER DEFAULT 0,
    unsubscribed INTEGER DEFAULT 0,
    spam_complaints INTEGER DEFAULT 0,
    status TEXT,
    billing_id BIGINT REFERENCES billing_analytics(id),
    updated TIMESTAMPTZ DEFAULT NOW()
);
```

**Purpose:** Per-lead engagement summaries to support scoring and segmentation.

## Warmup Analytics

```sql
CREATE TABLE warmup_analytics (
    id BIGINT PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
    mailbox_id TEXT NOT NULL,
    company_id TEXT NOT NULL,
    sent INTEGER DEFAULT 0,
    delivered INTEGER DEFAULT 0,
    opened_tracked INTEGER DEFAULT 0,
    clicked_tracked INTEGER DEFAULT 0,
    replied INTEGER DEFAULT 0,
    bounced INTEGER DEFAULT 0,
    unsubscribed INTEGER DEFAULT 0,
    spam_complaints INTEGER DEFAULT 0,
    health_score INTEGER DEFAULT 0,
    progress_percentage INTEGER DEFAULT 0,
    billing_id BIGINT REFERENCES billing_analytics(id),
    updated TIMESTAMPTZ DEFAULT NOW()
);
```

**Purpose:** Warmup performance and reputation metrics.

## Sequence Step Analytics

```sql
CREATE TABLE sequence_step_analytics (
    id BIGINT PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
    step_id TEXT NOT NULL,
    campaign_id TEXT NOT NULL,
    company_id TEXT NOT NULL,
    sent INTEGER DEFAULT 0,
    delivered INTEGER DEFAULT 0,
    opened_tracked INTEGER DEFAULT 0,
    clicked_tracked INTEGER DEFAULT 0,
    replied INTEGER DEFAULT 0,
    bounced INTEGER DEFAULT 0,
    unsubscribed INTEGER DEFAULT 0,
    spam_complaints INTEGER DEFAULT 0,
    billing_id BIGINT REFERENCES billing_analytics(id),
    updated TIMESTAMPTZ DEFAULT NOW()
);
```

**Purpose:** Step-level performance to support optimization and A/B testing.

## Table Relationships

| Parent | Child | Relationship |
|--------|-------|--------------|
| billing_analytics | campaign_analytics | Per-tenant period hub |
| billing_analytics | mailbox_analytics | Per-tenant period hub |
| billing_analytics | lead_analytics | Per-tenant period hub |
| billing_analytics | warmup_analytics | Per-tenant period hub |
| billing_analytics | sequence_step_analytics | Per-tenant period hub |
| campaign_analytics | sequence_step_analytics | Per-campaign breakdown |
| mailbox_analytics | warmup_analytics | Per-mailbox warmup |

**Note:** IDs like tenant_id, campaign_id, company_id, mailbox_id, lead_id are logical references to OLTP, denormalized for warehouse flexibility. No foreign keys to operational schemas.

---

**Last Updated:** November 17, 2025
