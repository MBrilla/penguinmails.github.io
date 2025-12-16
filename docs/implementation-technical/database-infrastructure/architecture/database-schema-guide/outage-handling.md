---
title: "Database Tier Outage Handling"
description: "Failure behavior and incident response for 5-tier database architecture"
last_modified_date: "2025-11-17"
level: "2"
persona: "Technical Teams, Site Reliability Engineers"
keywords: [outage, incident, failure, recovery, database, resilience]
---

# Database Tier Outage Handling

How the database tiers behave under failures, and how we avoid abusing OLAP or logs for live state.

## Tier Failure Behavior

Each tier has a clear blast radius. This guide defines expected behavior during outages.

### 1. OLTP Outage

**Impact:** Core workflows degraded (auth, campaigns, writes)

**Still Available:**
- Notifications DB (if separate) for previously written notifications/events
- External logging for error/infra signals
- OLAP (read-only) for historical analytics (not live truth)

**Behavior:**
- Jobs depending on OLTP back off and retry
- On recovery, queues drain and derived data (OLAP, notifications) reconverge with OLTP

### 2. Content DB Outage

**Impact:** Access to bodies/attachments limited

**Still Available:**
- OLTP metadata
- Notifications DB for surfaced incidents
- External logging shows related errors

**Behavior:**
- Core metadata flows continue
- Admin_system_events/notifications can highlight content issues without involving OLAP

### 3. OLAP Outage

**Impact:** Analytics dashboards unavailable or stale

**Not Impacted:**
- OLTP operations
- Notifications/incident visibility
- Queue/Jobs execution

**Behavior:**
- System continues to operate normally
- This is why we keep OLAP out of live notification/system-event storage

### 4. Queue / Jobs Outage

**Impact:** Delayed async processing (emails, aggregates, some notifications)

**Still Available:**
- OLTP, Notifications DB, external logging

**Behavior:**
- Jobs resume when restored
- No OLAP write-dependence for correctness

### 5. Notifications DB Outage

**Impact:** In-app notifications and curated admin events temporarily unavailable

**Still Available:**
- OLTP for core functions
- External logging for raw signals

**Behavior:**
- Critical events are still logged externally
- On recovery, we can repopulate curated events from logs or compensations if needed
- Notifications DB issues do not impact OLTP/OLAP schema correctness

### 6. External Logging Outage

**Impact:** Reduced observability

**Still Available:**
- OLTP, Content, OLAP, Notifications, Jobs

**Behavior:**
- Core correctness unaffected
- Admin_system_events/notifications still record curated incidents

## Failure Isolation Principles

| Tier | Blast Radius | Recovery Priority |
|------|--------------|-------------------|
| OLTP | High - Core workflows | P0 - Critical |
| Content DB | Medium - Email display | P1 - High |
| OLAP | Low - Analytics only | P2 - Medium |
| Queue/Jobs | Medium - Async processing | P1 - High |
| Notifications | Low - UI notifications | P2 - Medium |
| External Logging | Low - Observability | P3 - Low |

## Design Implications

### Key Insights

1. **Notifications DB + External Logging** provide runtime visibility and UX without abusing OLAP
2. **OLAP** remains a lean analytical layer, safe to be non-critical in outages
3. Each tier can fail independently without cascading failures

### Incident Response Guidelines

Use this behavior model when designing:
- Failure handling workflows
- Incident response flows
- Recovery procedures
- Monitoring and alerting

### Anti-Patterns to Avoid

- Using OLAP for live state or notifications
- Depending on external logging for correctness
- Making Queue/Jobs dependent on OLAP writes
- Storing core entities in Notifications DB

---

**Last Updated:** November 17, 2025
**Status:** Production Documentation
