---
title: "Operational Database Tiers"
description: "Queue/Jobs Store, Notifications DB, and External Logging tiers"
last_modified_date: "2025-11-17"
level: "2"
persona: "Technical Teams, Database Architects"
keywords: [queue, jobs, notifications, logging, observability, database]
---

# Operational Database Tiers

Detailed documentation for operational tiers: Queue/Jobs, Notifications, and External Logging.

## 4. Queue / Jobs Store

Asynchronous workflow orchestration tier.

### Purpose

- Email sending orchestration
- Analytics aggregation jobs
- Imports/exports processing
- Background maintenance tasks

### Implementation

Backed by:
- Redis + Postgres, or
- Dedicated Postgres tables

Stores:
- Job metadata (type, payload refs, status, attempts, errors, timestamps)

### Key Principles

- Operational; not a long-term analytics or logging store
- References payloads and entities via IDs/storage_keys rather than duplicating data
- Aggressively pruned; long-lived traces belong in external logging

**Reference:** [Queue System Implementation Guide](/docs/implementation-technical/database-infrastructure/queue-system-implementation-guide)

## 5. Notifications & System Events Database

Operational store for user-facing notifications and admin events.

### Purpose

- In-app user/admin notifications
- Curated admin system events

### Characteristics

- Independent from OLAP
- Optimized for fast reads on login
- Status updates (read/resolved)
- Bounded retention (short)

### Core Tables

#### notifications

User/admin visible notifications (in_app/email):

| Field | Description |
|-------|-------------|
| id | Unique identifier |
| user_id | Target user |
| tenant_id | Tenant scope |
| type | Notification type |
| title | Notification title |
| message | Content body |
| channel | Delivery channel |
| is_read | Read status |
| created_at | Creation timestamp |
| read_at | When read |
| expires_at | Expiration time |
| deleted_at | Soft delete |

**Semantics:** Source of truth for notification center UI. Retention via scheduled cleanup.

#### admin_system_events

Curated high-signal system/admin events:

| Field | Description |
|-------|-------------|
| id | Unique identifier |
| created_at | Event timestamp |
| event_type | Event classification |
| severity | Severity level |
| description | Event description |
| details | Additional context (JSONB) |
| tenant_id | Tenant scope |
| user_id | Associated user |
| is_resolved | Resolution status |
| resolved_at | Resolution timestamp |
| resolved_by | Resolver user |

**Semantics:** Backing store for system/incident dashboards. Not raw logs; mid-term history only.

### Key Principles

- Do not place these in OLAP
- Use Redis only as cache and rate-limiter (never as primary store)
- External logging holds raw/full-fidelity events and metrics

**Reference:** [Notifications Database Schema Guide](/docs/implementation-technical/database-infrastructure/notifications-database-schema-guide)

## 6. External Logging, Analytics, and Observability

Out-of-band tier for high-volume telemetry.

### Purpose

- Clickstream and product analytics
- Job/queue traces
- Infra logs and metrics
- Detailed send/delivery events
- Security/forensic logs

### Implementation

- PostHog or equivalent for product analytics
- Centralized logging (ELK/Loki)
- Metrics/tracing (Prometheus/OpenTelemetry) - Future/2026 Spike

### Key Principles

- Primary sink for raw, high-volume event data
- Feeds aggregations into OLAP when explicitly required
- Not used as primary UX state store
- Not a replacement for OLTP/Notifications DB correctness

**Reference:** [External Analytics Logging](/docs/implementation-technical/database-infrastructure/operations/external-analytics-logging)

## 7. Tier Responsibilities Summary

| Tier | Responsibilities |
|------|------------------|
| OLTP | Core entities and real-time workflows |
| Content DB | Heavy bodies and attachments |
| OLAP | Aggregated analytics and compliance summaries |
| Queue/Jobs | Asynchronous execution orchestration |
| Notifications | User/admin notifications and curated events |
| External Logging | Telemetry, raw events, observability |

### Feature Design Decision Guide

When designing new features, use this guide plus per-tier docs to decide:
- Which tier owns the data
- How it should be retained
- How it flows into analytics/logging without polluting OLTP/OLAP

---

**Last Updated:** November 17, 2025
**Status:** Production Documentation
