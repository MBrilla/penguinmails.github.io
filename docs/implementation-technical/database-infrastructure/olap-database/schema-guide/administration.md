---
title: "OLAP Administration & Governance"
description: "Audit logging, access control, and schema governance for OLAP warehouse"
last_modified_date: "2025-11-17"
level: "3"
persona: "Database Teams"
keywords: [olap, audit-log, security, governance, rls, compliance]
---

# OLAP Administration & Governance

Audit logging, access control, scope management, and schema governance for the OLAP warehouse.

## Administrative Audit (Compliance-Focused)

### admin_audit_log

```sql
CREATE TABLE admin_audit_log (
    id BIGINT PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
    creation_time TIMESTAMPTZ DEFAULT NOW(),
    admin_user_id TEXT NOT NULL,
    action TEXT NOT NULL,
    resource_type TEXT NOT NULL,
    resource_id TEXT NOT NULL,
    tenant_id TEXT NOT NULL,
    old_values JSONB,
    new_values JSONB,
    ip_address TEXT,
    user_agent TEXT,
    notes TEXT,
    metadata JSONB,
    data_classification VARCHAR(20),
    retention_category VARCHAR(20)
);
```

**Purpose:** OLAP-resident, compliance-scope audit log for high-risk actions:
- Permission/role changes
- Billing/subscription changes
- Tenant-wide configuration changes
- Sensitive export approvals

**Key Constraints - Do NOT Store:**
- Raw performance metrics
- Low-risk UI events
- Full request/response payloads

Those go to external logging (see external-analytics-logging).

## Removed / Out-of-Scope for OLAP

The following are intentionally NOT present in OLAP:

| Entity | Reason | Location |
|--------|--------|----------|
| admin_system_events | Live/operational events | Notifications DB |
| notifications | User-facing notifications | Notifications DB |
| analytics_connection_pools | Infra telemetry | External logging |
| analytics_rate_limits | Config/security telemetry | External logging |
| transactional_emails | Operational behavior | Jobs + external logging |

This keeps the OLAP schema lean, focused, and maintainable.

## Security, RLS, and Access

Apply RLS and access controls to all OLAP tables:

| Access Level | Permissions | Use Case |
|--------------|-------------|----------|
| Admin | All secrets across tenants | Troubleshooting, disaster recovery |
| Tenant | Own tenant data only | Self-service analytics |
| System | Read secrets for automation | Aggregation jobs, reporting |

### Key Controls

- Enforce tenant isolation where OLAP is exposed to tenants
- Restrict admin_audit_log to authorized roles and necessary scopes
- Apply row-level security (RLS) for multi-tenant queries

## Roadmap (Admin Analytics)

We explicitly defer any OLAP-specific modeling of admin/system events.

**If future needs arise (not implemented now), consider:**
- Aggregated views:
  - incidents_by_tenant_by_month
  - mean_time_to_resolve_by_severity
  - quota_breach_counts_over_time

**These would be:**
- Derived from the Notifications DB (`admin_system_events`) and/or external logs
- Implemented as clearly named aggregate tables/views
- Still respecting OLAP's "aggregated and lean" constraints

**Currently:**
- No such OLAP tables are defined
- All admin/system event analytics are future/roadmap only

## Summary

OLAP is now clearly constrained to:

| Category | Status |
|----------|--------|
| Business-critical aggregates | ✅ Included |
| Compliance-focused summaries | ✅ Included (admin_audit_log) |
| Live notifications or system events | ❌ Excluded |
| Infra/config/telemetry storage | ❌ Excluded |

Notifications/system events, logs, and jobs:
- Live in their respective tiers
- Feed OLAP only via intentional, aggregate pipelines when justified

Use this guide as the authoritative reference for what belongs in the OLAP warehouse.

## Related Documentation

- [OLAP Schema Overview](/docs/implementation-technical/database-infrastructure/olap-database/schema-guide/overview)
- [Analytics Tables](/docs/implementation-technical/database-infrastructure/olap-database/schema-guide/analytics-tables)
- [Notifications Database Schema Guide](/docs/implementation-technical/database-infrastructure/notifications-database-schema-guide)
- [External Analytics Logging](/docs/implementation-technical/database-infrastructure/operations/external-analytics-logging)
- [OLAP ER Diagram](/docs/implementation-technical/database-infrastructure/olap-database/mermaid-er)

---

**Last Updated:** November 17, 2025
