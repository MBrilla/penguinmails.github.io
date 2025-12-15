---
title: "Recovery Procedures"
last_modified_date: "2025-12-15"
level: "2"
persona: "DevOps, Database Administrators"
---

# Recovery Procedures

Point-in-time recovery and disaster recovery procedures for all database tiers.

## OLTP Recovery

### Point-in-Time Recovery

```bash
#!/bin/bash
# OLTP Point-in-Time Recovery
set -e

BACKUP_DATE="$1"  # Format: YYYYMMDD
TARGET_TIME="$2"  # Format: YYYY-MM-DD HH:MM:SS
S3_BUCKET="penguinmails-backups-us-east-1"
RESTORE_DIR="/restore/oltp"

echo "Starting OLTP recovery to $TARGET_TIME"

# Download backup
mkdir -p "$RESTORE_DIR"
aws s3 sync "s3://$S3_BUCKET/oltp/$BACKUP_DATE/" "$RESTORE_DIR/"

# Stop current database connections
psql -c "SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE datname = 'penguinmails_oltp';"

# Restore from backup
pg_restore \
  --host=oltp-db.penguinmails.com \
  --username=restore_user \
  --dbname=penguinmails_oltp_restore \
  --clean \
  --verbose \
  "$RESTORE_DIR/oltp_full_*.dump"

# Apply WAL logs up to target time
aws s3 sync "s3://$S3_BUCKET/oltp/wal_archive/$BACKUP_DATE/" "$RESTORE_DIR/wal/"

# Configure recovery.conf for point-in-time recovery
cat > /var/lib/postgresql/recovery.conf << EOF
restore_command = 'cp $RESTORE_DIR/wal/%f %p'
recovery_target_time = '$TARGET_TIME'
recovery_target_action = 'promote'
EOF

echo "OLTP recovery initiated - monitor PostgreSQL logs"
```

### Full Database Restore

```bash
#!/bin/bash
# OLTP Full Database Restore
set -e

BACKUP_FILE="$1"
TARGET_DB="penguinmails_oltp"

echo "Starting full OLTP restore from $BACKUP_FILE"

# Create fresh database
psql -c "DROP DATABASE IF EXISTS ${TARGET_DB};"
psql -c "CREATE DATABASE ${TARGET_DB};"

# Restore from backup
pg_restore \
  --host=oltp-db.penguinmails.com \
  --username=restore_user \
  --dbname=$TARGET_DB \
  --verbose \
  "$BACKUP_FILE"

# Verify restore
psql -d $TARGET_DB -c "SELECT COUNT(*) FROM users;"
psql -d $TARGET_DB -c "SELECT COUNT(*) FROM tenants;"

echo "OLTP full restore completed"
```

## Content Database Recovery

### Content Restore Procedure

```bash
#!/bin/bash
# Content Database Recovery
set -e

BACKUP_DATE="$1"
S3_BUCKET="penguinmails-backups-us-east-1"
RESTORE_DIR="/restore/content"

echo "Starting Content database recovery"

# Download backups from Glacier (may take hours)
aws s3 sync "s3://$S3_BUCKET/content/$BACKUP_DATE/" "$RESTORE_DIR/" \
  --request-payer requester

# Restore metadata first
pg_restore \
  --host=content-db.penguinmails.com \
  --username=restore_user \
  --dbname=penguinmails_content \
  --clean \
  "$RESTORE_DIR/content_metadata.dump"

# Restore content objects
pg_restore \
  --host=content-db.penguinmails.com \
  --username=restore_user \
  --dbname=penguinmails_content \
  --data-only \
  "$RESTORE_DIR/content_objects.dump"

echo "Content database recovery completed"
```

## Queue System Recovery

### Queue WAL Replay

```bash
#!/bin/bash
# Queue System Recovery via WAL Replay
set -e

BACKUP_DATE="$1"
S3_BUCKET="penguinmails-backups-us-east-1"

echo "Starting Queue system recovery"

# Download WAL files
aws s3 sync "s3://$S3_BUCKET/queue/wal/$BACKUP_DATE/" "/restore/queue/wal/"

# Replay WAL files to recover queue state
# This provides zero data loss recovery
pg_restore --target-action=promote

echo "Queue system recovery completed"
```

## Disaster Recovery Checklist

### Pre-Recovery

- [ ] Assess scope of data loss
- [ ] Identify most recent valid backup
- [ ] Notify stakeholders of expected downtime
- [ ] Prepare recovery environment

### During Recovery

- [ ] Execute tier-specific recovery procedure
- [ ] Monitor recovery progress
- [ ] Validate data integrity at checkpoints
- [ ] Document any issues encountered

### Post-Recovery

- [ ] Verify all data restored correctly
- [ ] Run application smoke tests
- [ ] Resume normal operations
- [ ] Conduct incident post-mortem
- [ ] Update recovery procedures if needed

---

[← Back to Backup & Recovery](./README)
