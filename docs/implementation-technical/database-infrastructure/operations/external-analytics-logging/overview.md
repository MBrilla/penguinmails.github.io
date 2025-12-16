---
title: "External Analytics and Logging Integration"
description: "Separation of concerns for high-volume events, observability, and OLAP boundaries"
last_modified_date: "2025-11-19"
level: "2"
persona: "Technical Teams"
keywords: [analytics, logging, observability, posthog, olap, telemetry]
---

# External Analytics and Logging Integration

How high-volume events, observability data, and exploratory analytics are handled outside the OLAP warehouse.

## Related Documentation

- [Integration Patterns](/docs/implementation-technical/database-infrastructure/operations/external-analytics-logging/integration-patterns)
- [OLAP Schema Guide](/docs/implementation-technical/database-infrastructure/olap-database/schema-guide/overview)

## Purpose

- Keep the OLAP warehouse focused on durable, contractual, and compliance-relevant aggregates
- Route noisy, granular, or purely operational data to specialized external systems
- Avoid coupling OLAP schema to infra configuration or runtime telemetry

## Separation of Concerns

### 1. OLAP Warehouse

**Stores:**
- Aggregated business metrics (per period, per entity)
- Discrete, important state transitions
- Compliance- and contract-relevant records

**Characteristics:**
- Low-to-moderate volume
- Long-term retention with governance
- Directly supports billing, tenant usage, dashboards, audits

### 2. External Analytics (PostHog or equivalent)

**Primary sink for:**
- High-volume clickstream events
- Feature usage and funnels
- Admin UI interactions and navigation behavior
- Experimentation, retention, and UX telemetry

**Characteristics:**
- Event-centric
- Flexible schema
- Tunable retention and sampling
- Feeds aggregated metrics back into OLAP when needed

### 3. Observability / Logging / Security Tooling

**Includes:** APM, metrics, logs, SIEM

**Primary sink for:**
- Infrastructure metrics (CPU, memory, connection pools)
- Application performance metrics
- Error logs, stack traces, retries, timeouts
- Detailed security logs for forensic use

**OLAP relationship:**
- May receive highly aggregated summaries (daily uptime, incident counts)
- Does not mirror full logs

## What Must Go to External Systems

### Admin UI and Low-Risk Behavioral Events

| Event Type | Handling |
|------------|----------|
| Page views in admin panel | Product analytics (admin_ui_viewed) |
| Non-sensitive filter changes | Product analytics |
| Navigation clicks, toggles | Product analytics |
| Session keep-alives | Product analytics |

**NOT** in `admin_audit_log` or other OLAP tables.

### Performance Metrics and Query Telemetry

| Event Type | Handling |
|------------|----------|
| query_duration_ms | Observability stack |
| query_complexity_score | Observability stack |
| Connection pool behavior | Observability stack |

Optionally mirror as aggregated daily summaries in OLAP for capacity planning.

### Infrastructure and System Events

| Event Type | Handling |
|------------|----------|
| CPU/memory/disk usage | Logging/metrics stack |
| Connection pool utilization | Logging/metrics stack |
| Autoscaling events | Logging/metrics stack |
| Health checks | Logging/metrics stack |

OLAP may contain coarse `system_incident` summaries, not every alert.

## OLAP-Aware vs External-Only Models

### admin_audit_log

**OLAP (keep):**
- Role/permission changes
- Tenant-wide configuration changes
- Billing and subscription modifications
- Data export requests affecting PII
- Security-sensitive operations

**External (log only):**
- Low-risk admin actions, navigation
- Detailed timing and performance metrics
- UI-level interactions

### admin_system_events

**OLAP (summarized):**
- Notable events for admin dashboards
- System-wide milestones

**External:**
- All granular alerts, heartbeats, retries
- Any signal used only for SRE/debugging

### Analytics Infra Tables

Tables like `analytics_connection_pools`, `analytics_pool_metrics`, `analytics_rate_limits`:

- **Manage via:** Application config, environment variables, infra-as-code
- **Collect metrics via:** Observability stack (Prometheus, OpenTelemetry)
- **OLAP:** At most, aggregated capacity/uptime summaries

## Design Rules Checklist

When designing new tracking, ask:

**Is this required for:**
- Billing/invoicing?
- SLAs or contractual proof?
- Regulatory or enterprise audits?
- Promised customer-facing analytics?

**If yes →** Model an aggregate in OLAP and/or store in `admin_audit_log`

**If no, route to:**
- Behavioral/experimental/UX → Product analytics (PostHog)
- Infra/ops/telemetry → Observability/logging
- Runtime configuration → Config/OLTP
- User-facing notifications → Notifications DB

---

**Last Updated:** November 19, 2025
