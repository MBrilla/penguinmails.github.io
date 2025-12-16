---
title: "Database Tier Operations"
description: "OLTP, Content, Queue, and OLAP database tier operations and performance targets"
last_modified_date: "2025-11-19"
level: "3"
persona: "Database Ops Team, DevOps Engineers"
---

# Database Tier Operations

OLTP, Content, Queue, and OLAP database tier operations, monitoring, and performance targets.

## OLTP Database (Primary Operations)

### Connection Pool Management

```sql
-- Check OLTP pool status
SELECT
    pool_name,
    min_connections,
    max_connections,
    active_connections,
    idle_connections,
    connection_usage_rate,
    pending_acquires
FROM connection_pool_config cpc
JOIN connection_pool_metrics cpm ON cpc.id = cpm.pool_config_id
WHERE cpc.tier = 'oltp'
AND cpm.collected_at >= NOW() - INTERVAL '5 minutes';
```

### Performance Targets

| Metric | Target |
|--------|--------|
| Query Response Time | <200ms for 95th percentile |
| Connection Pool Usage | <80% utilization |
| Uptime | 99.9% availability |
| Transaction Rate | 1000+ transactions/second |

## Content Database (Email Management)

### Content Lifecycle Operations

```sql
-- Check content retention status
SELECT
    DATE(created) as content_date,
    COUNT(*) as total_messages,
    AVG(LENGTH(content)) as avg_content_size,
    MIN(created) as earliest,
    MAX(created) as latest
FROM email_messages
WHERE created >= NOW() - INTERVAL '30 days'
GROUP BY DATE(created)
ORDER BY content_date DESC;
```

### Performance Targets

| Metric | Target |
|--------|--------|
| Content Retrieval | <1s for email content access |
| Storage Efficiency | 60% compression ratio |
| Retention Management | Automated lifecycle policies |
| Backup Frequency | Every 6 hours with point-in-time recovery |

## Queue System (Background Processing)

### Queue Health Monitoring

```sql
-- Check queue performance by priority
SELECT
    priority,
    COUNT(*) as job_count,
    AVG(EXTRACT(EPOCH FROM (completed - created))) as avg_processing_time_seconds,
    MIN(created) as oldest_job
FROM jobs
WHERE status = 'completed'
AND created >= NOW() - INTERVAL '1 hour'
GROUP BY priority
ORDER BY
    CASE priority
        WHEN 'critical' THEN 1
        WHEN 'high' THEN 2
        WHEN 'normal' THEN 3
        WHEN 'low' THEN 4
        ELSE 5
    END;
```

### Performance Targets

| Metric | Target |
|--------|--------|
| Queue Processing | <20s average processing time |
| Throughput | 2000+ jobs/minute capacity |
| Failure Rate | <1% job failure rate |
| Backlog Management | <100 job backlog threshold |

## OLAP Analytics (Business Intelligence)

### Performance Targets

| Metric | Target |
|--------|--------|
| Query Response | <5s for complex analytics queries |
| Data Freshness | <1 hour delay for real-time dashboards |
| Storage Growth | Controlled growth with automated archival |
| Report Generation | <30s for standard reports |

## Related Documentation

- [Infrastructure Operations Overview](/docs/operations/analytics/operations-management/infra-ops/overview) - Main page
- [Emergency Response](/docs/operations/analytics/operations-management/infra-ops/emergency-response) - Incident procedures
- [Daily Operations](/docs/operations/analytics/operations-management/infra-ops/daily-operations) - Health checks
- [Scaling & Integration](/docs/operations/analytics/operations-management/infra-ops/scaling-integration) - Business integration
