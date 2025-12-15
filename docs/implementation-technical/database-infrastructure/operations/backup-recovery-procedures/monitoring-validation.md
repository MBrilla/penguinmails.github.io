---
title: "Monitoring & Validation"
last_modified_date: "2025-12-15"
level: "2"
persona: "DevOps, Database Administrators"
---

# Monitoring & Validation

Backup monitoring, testing, and compliance procedures.

## Backup Monitoring Dashboard

### Key Metrics

| Metric | Target | Alert Threshold |
|--------|--------|-----------------|
| Backup success rate | 100% | < 99% |
| Backup duration | < baseline + 20% | > baseline + 50% |
| Storage utilization | < 80% | > 90% |
| Cross-region sync lag | < 1 minute | > 5 minutes |

### Monitoring Queries

```sql
-- Check backup status for last 24 hours
SELECT 
  backup_type,
  database_tier,
  start_time,
  end_time,
  status,
  size_bytes,
  CASE 
    WHEN status = 'completed' THEN 'OK'
    ELSE 'ALERT'
  END as health
FROM backup_logs
WHERE start_time > NOW() - INTERVAL '24 hours'
ORDER BY start_time DESC;

-- Check backup size trends
SELECT 
  database_tier,
  DATE(start_time) as backup_date,
  AVG(size_bytes) as avg_size,
  MAX(size_bytes) as max_size
FROM backup_logs
WHERE start_time > NOW() - INTERVAL '30 days'
GROUP BY database_tier, DATE(start_time)
ORDER BY backup_date DESC;
```

## Backup Validation

### Daily Validation Script

```bash
#!/bin/bash
# Daily Backup Validation
set -e

S3_BUCKET="penguinmails-backups-us-east-1"
VALIDATION_DIR="/validation/$(date +%Y%m%d)"

echo "Starting daily backup validation"

mkdir -p "$VALIDATION_DIR"

# Validate OLTP backup integrity
aws s3 cp "s3://$S3_BUCKET/oltp/$(date +%Y%m%d)/oltp_full_*.dump" "$VALIDATION_DIR/"
pg_restore --list "$VALIDATION_DIR/oltp_full_*.dump" > "$VALIDATION_DIR/oltp_toc.txt"
if [ $? -eq 0 ]; then
  echo "OLTP backup validation: PASSED"
else
  echo "OLTP backup validation: FAILED"
  exit 1
fi

# Validate Content backup integrity
aws s3 cp "s3://$S3_BUCKET/content/$(date +%Y%m%d)/content_metadata.dump" "$VALIDATION_DIR/"
pg_restore --list "$VALIDATION_DIR/content_metadata.dump" > "$VALIDATION_DIR/content_toc.txt"
if [ $? -eq 0 ]; then
  echo "Content backup validation: PASSED"
else
  echo "Content backup validation: FAILED"
  exit 1
fi

echo "Daily backup validation completed successfully"
```

### Weekly Restore Test

```bash
#!/bin/bash
# Weekly Restore Test - Validates full recovery capability
set -e

TEST_DB="penguinmails_restore_test"
BACKUP_DATE="$(date -d 'yesterday' +%Y%m%d)"

echo "Starting weekly restore test"

# Create test database
psql -c "DROP DATABASE IF EXISTS $TEST_DB;"
psql -c "CREATE DATABASE $TEST_DB;"

# Restore latest OLTP backup to test database
pg_restore \
  --dbname=$TEST_DB \
  --verbose \
  "/backups/oltp/$BACKUP_DATE/oltp_full_*.dump"

# Validate critical tables
USERS_COUNT=$(psql -t -d $TEST_DB -c "SELECT COUNT(*) FROM users;")
TENANTS_COUNT=$(psql -t -d $TEST_DB -c "SELECT COUNT(*) FROM tenants;")

echo "Restore test results:"
echo "  Users: $USERS_COUNT"
echo "  Tenants: $TENANTS_COUNT"

# Cleanup test database
psql -c "DROP DATABASE $TEST_DB;"

echo "Weekly restore test completed successfully"
```

## Compliance Requirements

### Audit Trail

All backup operations are logged with:
- Timestamp
- Operator (system or user)
- Operation type
- Source and destination
- Success/failure status
- Error details (if applicable)

### Retention Compliance

| Regulation | Requirement | Implementation |
|------------|-------------|----------------|
| GDPR | Right to erasure | Backup rotation + data anonymization |
| SOC 2 | Backup validation | Weekly restore tests |
| HIPAA | Encryption at rest | AES-256 encryption |
| PCI-DSS | Access controls | IAM policies + audit logs |

### Compliance Reporting

```sql
-- Monthly compliance report
SELECT 
  COUNT(*) FILTER (WHERE status = 'completed') as successful_backups,
  COUNT(*) FILTER (WHERE status = 'failed') as failed_backups,
  COUNT(*) FILTER (WHERE validated = true) as validated_backups,
  MIN(start_time) as oldest_backup_tested,
  MAX(start_time) as newest_backup_tested
FROM backup_logs
WHERE start_time > NOW() - INTERVAL '30 days';
```

---

[← Back to Backup & Recovery](./README)
