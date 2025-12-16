---
title: "External Analytics Integration Patterns"
description: "PostHog integration, queue-driven ETL, and event taxonomy for external analytics"
last_modified_date: "2025-11-19"
level: "3"
persona: "Technical Teams"
keywords: [posthog, etl, events, integration, analytics, pipeline]
---

# External Analytics Integration Patterns

PostHog integration, queue-driven ETL, and event taxonomy patterns for external analytics.

## PostHog Integration (Product Analytics)

Use a thin abstraction to send high-volume behavioral and UX events to PostHog, keeping OLAP free of clickstream:

```typescript
// analytics/posthogClient.ts
import { PostHog } from 'posthog-node';

export const posthog = new PostHog(process.env.POSTHOG_API_KEY!, {
  host: process.env.POSTHOG_HOST || 'https://app.posthog.com',
});

// Example: track email opened event
export async function trackEmailOpened(props: {
  userId: string;
  tenantId: string;
  campaignId?: string;
  mailboxId?: string;
  emailId: string;
}) {
  await posthog.capture({
    distinctId: props.userId,
    event: 'email_opened',
    properties: {
      tenant_id: props.tenantId,
      campaign_id: props.campaignId,
      mailbox_id: props.mailboxId,
      email_id: props.emailId,
    },
  });
}
```

**Key Rules:**
- Do not mirror these raw events into OLAP
- Only derive aggregates into canonical OLAP tables if they are contractual, compliance-relevant, or explicitly needed for durable analytics

## Queue-Driven ETL into Lean OLAP

Use jobs/queues to periodically aggregate from OLTP + external analytics into the canonical OLAP tables:

```typescript
// analytics/etlPipeline.ts
import { sql } from 'drizzle-orm';
import { db } from '../db';

export async function aggregateDailyCampaignAnalytics(
  tenantId: string, 
  date: string
) {
  const metrics = await db.execute(sql`
    SELECT
      campaign_id,
      COUNT(*) AS sent,
      COUNT(*) FILTER (WHERE status = 'delivered') AS delivered,
      COUNT(*) FILTER (WHERE opened IS NOT NULL) AS opened_tracked,
      COUNT(*) FILTER (WHERE clicked IS NOT NULL) AS clicked_tracked,
      COUNT(*) FILTER (WHERE replied IS NOT NULL) AS replied,
      COUNT(*) FILTER (WHERE bounce_type IS NOT NULL) AS bounced,
      COUNT(*) FILTER (WHERE unsubscribed IS NOT NULL) AS unsubscribed,
      COUNT(*) FILTER (WHERE complaint IS NOT NULL) AS spam_complaints
    FROM emails
    WHERE tenant_id = ${tenantId}
      AND DATE(sent_at) = ${date}
    GROUP BY campaign_id
  `);

  for (const row of metrics) {
    await db.execute(sql`
      INSERT INTO campaign_analytics (
        campaign_id, company_id, sent, delivered,
        opened_tracked, clicked_tracked, replied,
        bounced, unsubscribed, spam_complaints, updated
      )
      VALUES (
        ${row.campaign_id}, ${row.company_id}, ${row.sent},
        ${row.delivered}, ${row.opened_tracked}, ${row.clicked_tracked},
        ${row.replied}, ${row.bounced}, ${row.unsubscribed},
        ${row.spam_complaints}, NOW()
      )
      ON CONFLICT (campaign_id) DO UPDATE
      SET sent = EXCLUDED.sent, delivered = EXCLUDED.delivered,
          opened_tracked = EXCLUDED.opened_tracked,
          clicked_tracked = EXCLUDED.clicked_tracked,
          replied = EXCLUDED.replied, bounced = EXCLUDED.bounced,
          unsubscribed = EXCLUDED.unsubscribed,
          spam_complaints = EXCLUDED.spam_complaints, updated = NOW();
    `);
  }
}
```

**Key Rules:**
- Jobs and ETL workers live in the Queue/Jobs tier
- Emit traces/metrics to external logging
- OLAP receives only stable aggregates

**Target OLAP Tables:**
- billing_analytics
- campaign_analytics
- mailbox_analytics
- lead_analytics
- warmup_analytics
- sequence_step_analytics
- admin_audit_log (compliance-focused)

## Event Taxonomy

Standardized event categories for external-only tracking:

### Email/Campaign Events

| Event | Description |
|-------|-------------|
| email_sent | Email dispatched to provider |
| email_delivered | Confirmed delivery to recipient |
| email_opened | Recipient opened email |
| email_clicked | Recipient clicked link |
| email_bounced | Email bounced |
| email_replied | Recipient replied |

### Security Events

| Event | Description |
|-------|-------------|
| security_login_* | Login attempts and outcomes |
| security_suspicious_activity | Potential threat detected |
| security_data_access | Sensitive data access |

### Infrastructure/Ops Events

| Event | Description |
|-------|-------------|
| infra_connection_pool_metrics | Connection pool state |
| infra_error_rate | Application error rates |
| infra_latency | Request latency metrics |

### Business Events

| Event | Description |
|-------|-------------|
| business_signup | New customer signup |
| business_upgrade | Plan upgrade |
| business_churn | Customer churn |

**All these events:**
- Are tracked in external analytics/logging platforms
- Only surface in OLAP as coarse aggregates if they meet the design rules

## Related Documentation

- [External Analytics Overview](/docs/implementation-technical/database-infrastructure/operations/external-analytics-logging/overview)
- [OLAP Schema Guide](/docs/implementation-technical/database-infrastructure/olap-database/schema-guide/overview)
- [Queue System Implementation](/docs/implementation-technical/database-infrastructure/queue-system-implementation-guide)

---

**Last Updated:** November 19, 2025
