---
title: "Daily Operations Checklist"
description: "Morning, midday, and end-of-day operational health checks and maintenance"
last_modified_date: "2025-11-19"
level: "3"
persona: "Database Ops Team, DevOps Engineers"
---

# Daily Operations Checklist

Morning, midday, and end-of-day operational health checks and maintenance procedures.

## Morning Health Check (9:00 AM)

### Quick Check (5 minutes)

- [ ] **PostHog Dashboard Review** - Check database performance metrics
- [ ] **Connection Pool Status** - Verify pool health across all tiers

### Standard Operations (15 minutes)

- [ ] **Backup Verification** - Confirm last successful backups for all databases
- [ ] **Error Log Review** - Check for new errors or warnings

### Comprehensive Review (30 minutes)

- [ ] **Queue Health** - Verify queue processing rates and backlog
- [ ] **Storage Usage** - Check storage consumption and growth trends

## Midday Performance Review (1:00 PM)

### Quick Analysis (10 minutes)

- [ ] **Query Performance** - Review slow queries
- [ ] **Index Usage** - Check index effectiveness

### Standard Review (20 minutes)

- [ ] **Resource Utilization** - Monitor CPU, memory, and disk usage

### Comprehensive Analysis (30 minutes)

- [ ] **Security Events** - Review security alerts

## End-of-Day Review (5:00 PM)

### Basic Tasks (10 minutes)

- [ ] **Daily Reports** - Generate reports following QA Success Measurement Framework
- [ ] **Tomorrow's Preparation** - Prepare with QA Continuous Improvement Framework

### Standard Operations (20 minutes)

- [ ] **Maintenance Tasks** - Complete scheduled activities

### Comprehensive Operations (30 minutes)

- [ ] **Alert Review** - Address alerts using QA Issue Detection & Response

## Performance Monitoring Integration

### PostHog Dashboard Access

```yaml
Dashboard URL: https://app.posthog.com/[PROJECT_ID]
Key Metrics:
  - Database Query Performance
  - Connection Pool Utilization
  - Queue Processing Rates
  - Content Storage Efficiency
  - Analytics Data Freshness

Alert Setup:
  - Query response time > 5 seconds
  - Connection pool usage > 90%
  - Queue backlog > 100 jobs
  - Storage growth > 10% daily

Configuration Notes:
  - Replace [PROJECT_ID] with actual PostHog project ID
  - Configure custom events for database performance tracking
  - Set up alerting rules for critical performance thresholds
```

## Related Documentation

- [Infrastructure Operations Overview](/docs/operations/analytics/operations-management/infra-ops/overview) - Main page
- [Emergency Response](/docs/operations/analytics/operations-management/infra-ops/emergency-response) - Incident procedures
- [Database Tiers](/docs/operations/analytics/operations-management/infra-ops/database-tiers) - Tier operations
