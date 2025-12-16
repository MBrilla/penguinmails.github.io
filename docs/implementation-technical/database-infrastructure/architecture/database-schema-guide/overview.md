---
title: "5-Tier Database Architecture Overview"
description: "Overview of PenguinMails canonical 5-tier database architecture model"
last_modified_date: "2025-11-17"
level: "2"
persona: "Technical Teams, Database Architects"
keywords: [database, architecture, 5-tier, oltp, olap, content, queue]
---

# 5-Tier Database Architecture Overview

This guide defines the canonical database tiering for PenguinMails. We operate a 5-tier model with external logging as an out-of-band component.

## Architecture Summary

| Tier | Purpose | Characteristics |
|------|---------|-----------------|
| OLTP | Operational Core | Strong consistency, multi-tenant RLS |
| Content DB | Heavy Content | Email bodies, attachments, archival |
| OLAP | Analytics Warehouse | Aggregation-focused, reporting |
| Queue/Jobs | Async Workflows | Job metadata, orchestration |
| Notifications | System Events | User notifications, admin events |

External logging/observability complements these tiers for telemetry and raw events.

## Related Documentation

- [Data Tiers: OLTP & Content](/docs/implementation-technical/database-infrastructure/architecture/database-schema-guide/data-tiers)
- [Operational Tiers](/docs/implementation-technical/database-infrastructure/architecture/database-schema-guide/operational-tiers)
- [Outage Handling](/docs/implementation-technical/database-infrastructure/architecture/database-schema-guide/outage-handling)

## 1. OLTP - Operational Core

Primary system of record for core entities.

### Purpose

- Tenants, users, organizations
- Campaigns, leads, mailboxes, domains
- Billing entities and subscriptions
- Operational configurations

### Characteristics

- Strong consistency, transactional semantics
- Multi-tenant isolation via RLS and clear ownership
- Optimized for OLTP access patterns

### Key Principles

- Store only what is required for correctness and core workflows
- No heavy content blobs
- No high-volume logs or analytics aggregates

**Reference:** [OLTP Database Schema](/docs/implementation-technical/database-infrastructure/architecture/oltp-database/)

## 2. Content Database - Heavy Content Storage

Dedicated tier for large content objects.

### Purpose

- Email bodies and message content
- Attachments and large binary objects
- Archival content

### Characteristics

- Opaque `storage_key` references from OLTP
- Optimized for storage efficiency, retention, compression/dedup

### Key Principles

- No core business entities
- No generalized logging, metrics, or infra configuration
- No analytics aggregates as primary concern

### Core Schema

- `content_objects`
- `attachments`

**Reference:** [Content Database Guide](/docs/implementation-technical/database-infrastructure/content-database/)

## 3. OLAP Analytics Warehouse

Query-optimized analytics for business intelligence.

### Purpose

- Billing/usage analytics
- Campaign and sequence performance
- Mailbox/warmup health
- Lead engagement
- Compliance-relevant admin actions

### Characteristics

- Aggregation-focused
- Partitioned and indexed for reporting
- Feeds BI tools and customer-facing analytics

### Key Principles

- Only business-critical aggregates and compliance summaries
- No live notifications
- No operational system events
- No infra metrics, rate-limits, or raw logs

### Canonical Tables

| Table | Purpose |
|-------|---------|
| billing_analytics | Usage and billing data |
| campaign_analytics | Campaign performance |
| mailbox_analytics | Mailbox health metrics |
| lead_analytics | Lead engagement data |
| warmup_analytics | Warmup progression |
| sequence_step_analytics | Sequence performance |
| admin_audit_log | Compliance-focused audit |

**Reference:** [OLAP Database Guide](/docs/implementation-technical/database-infrastructure/olap-database/)

---

**Last Updated:** November 17, 2025
**Status:** Production Documentation
