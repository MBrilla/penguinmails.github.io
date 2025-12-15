---
title: "Backup Architecture"
last_modified_date: "2025-12-15"
level: "2"
persona: "DevOps, Database Administrators"
---

# Backup Architecture

4-tier backup strategy and recovery objectives for enterprise data protection.

## 4-Tier Backup Strategy

| Database Tier | Backup Type | Frequency | Retention | Storage Location |
|---------------|-------------|-----------|-----------|------------------|
| **OLTP** | Full + Incremental | Daily full, hourly incremental | 90 days | Primary + Cross-region |
| **Content** | Full + Compressed | Daily full, 4-hour incremental | 180 days | Primary + Archive |
| **Queue** | Transaction log | Continuous + Daily full | 30 days | Primary + Secondary |
| **OLAP** | Full + Snapshot | Daily full, weekly snapshot | 365 days | Primary + Analytics archive |

## Recovery Objectives

### Recovery Time Objectives (RTO)

| Tier | RTO | Business Impact |
|------|-----|-----------------|
| OLTP | 15 minutes | Critical business operations |
| Content | 30 minutes | Email delivery |
| Queue | 5 minutes | Background processing |
| OLAP | 2 hours | Analytics reporting |

### Recovery Point Objectives (RPO)

| Tier | RPO | Data Loss Tolerance |
|------|-----|---------------------|
| OLTP | 15 minutes | Minimal data loss |
| Content | 1 hour | Acceptable for email content |
| Queue | 0 minutes | Transaction log replay |
| OLAP | 24 hours | Analytics data acceptable loss |

## Storage Architecture

### Primary Storage

- PostgreSQL native backup format
- Compressed with gzip level 6-9
- Encrypted at rest with AES-256

### Cross-Region Replication

- Secondary region for disaster recovery
- Asynchronous replication with < 1 minute lag
- Automatic failover capability

### Archive Storage

- S3 Glacier for long-term retention
- Lifecycle policies for cost optimization
- Compliance retention locks where required

## Backup Scheduling

```yaml
Daily Schedule:
  02:00 UTC: OLTP full backup
  03:00 UTC: Content full backup
  04:00 UTC: OLAP full backup
  05:00 UTC: Backup validation jobs

Hourly Schedule:
  :00: OLTP incremental backup
  :15: Queue transaction log backup
  :30: Content incremental (every 4 hours)
  :45: Backup monitoring check

Weekly Schedule:
  Sunday 06:00 UTC: OLAP snapshot
  Sunday 08:00 UTC: Backup retention cleanup
  Sunday 10:00 UTC: Cross-region sync validation
```

---

[← Back to Backup & Recovery](./README)
