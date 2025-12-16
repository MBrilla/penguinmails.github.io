---
title: "Tier-Specific Pool Configurations"
description: "OLTP, Content, Queue, and OLAP database pool configurations"
last_modified_date: "2025-11-19"
level: 2
persona: "Documentation Users"
keywords: ["oltp-pooling", "content-pooling", "queue-pooling", "olap-pooling"]
---

# Tier-Specific Pool Configurations

Detailed connection pool configurations for each database tier in the 4-tier architecture.

## OLTP Database Pooling

### Primary Pool Configuration

```sql
-- OLTP Primary Pool (Transactional Operations)
INSERT INTO connection_pool_config (
    tier, pool_name, min_connections, max_connections,
    connection_timeout_seconds, idle_timeout_seconds, max_lifetime_seconds
) VALUES (
    'oltp', 'primary_pool', 5, 50, 30, 600, 3600
) ON CONFLICT (tier, pool_name) DO UPDATE SET
    min_connections = EXCLUDED.min_connections,
    max_connections = EXCLUDED.max_connections;
```

**Performance Targets**:
- Query Response Time: <200ms for 95th percentile
- Connection Pool Usage: <80% utilization
- Uptime: 99.9% availability
- Transaction Rate: 1000+ transactions/second

### Read Replica Pool

```sql
-- OLTP Read Replica Pool (Reporting & Analytics)
INSERT INTO connection_pool_config (
    tier, pool_name, min_connections, max_connections,
    connection_timeout_seconds, idle_timeout_seconds
) VALUES (
    'oltp', 'read_replica_pool', 3, 30, 30, 600
);
```

## Content Database Pooling

### Content Operations Pool

```sql
-- Content Database Pool (Email Content Management)
INSERT INTO connection_pool_config (
    tier, pool_name, min_connections, max_connections,
    connection_timeout_seconds, idle_timeout_seconds
) VALUES (
    'content', 'content_pool', 3, 25, 60, 900
);
```

**Performance Targets**:
- Content Retrieval: <1s for email content access
- Storage Efficiency: 60% compression ratio
- Retention Management: Automated lifecycle policies

### Content Archival Pool

```sql
-- Content Archival Pool (Long-running operations)
INSERT INTO connection_pool_config (
    tier, pool_name, min_connections, max_connections,
    connection_timeout_seconds, idle_timeout_seconds
) VALUES (
    'content', 'archival_pool', 1, 10, 120, 1800
);
```

## Queue System Pooling

### Queue Processing Pool

```sql
-- Queue System Pool (Background Jobs)
INSERT INTO connection_pool_config (
    tier, pool_name, min_connections, max_connections,
    connection_timeout_seconds, idle_timeout_seconds
) VALUES (
    'queue', 'queue_pool', 5, 40, 15, 300
);
```

**Performance Targets**:
- Queue Processing: <20s average processing time
- Throughput: 2000+ jobs/minute capacity
- Failure Rate: <1% job failure rate

### Worker Processing Pool

```sql
-- Queue Worker Pool (High-concurrency processing)
INSERT INTO connection_pool_config (
    tier, pool_name, min_connections, max_connections,
    connection_timeout_seconds, idle_timeout_seconds
) VALUES (
    'queue', 'worker_pool', 2, 20, 20, 300
);
```

## OLAP Analytics Pooling

### Analytics Processing Pool

```sql
-- OLAP Analytics Pool (Complex queries)
INSERT INTO connection_pool_config (
    tier, pool_name, min_connections, max_connections,
    connection_timeout_seconds, idle_timeout_seconds
) VALUES (
    'olap', 'analytics_pool', 3, 15, 120, 1800
);
```

**Performance Targets**:
- Query Response: <5s for complex analytics queries
- Data Freshness: <1 hour delay for real-time dashboards
- Report Generation: <30s for standard reports

### Reporting Pool

```sql
-- OLAP Reporting Pool (Dashboard queries)
INSERT INTO connection_pool_config (
    tier, pool_name, min_connections, max_connections,
    connection_timeout_seconds, idle_timeout_seconds
) VALUES (
    'olap', 'reporting_pool', 2, 25, 90, 1200
);
```

## Related Documentation

- **[Overview](overview)** - Strategy overview
- **[Auto-Scaling](auto-scaling)** - Dynamic scaling rules
