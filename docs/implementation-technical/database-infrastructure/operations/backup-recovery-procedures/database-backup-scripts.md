---
title: "Database Backup Scripts"
last_modified_date: "2025-12-15"
level: "2"
persona: "DevOps, Database Administrators"
---

# Database Backup Scripts

OLTP, Content, Queue, and OLAP backup procedures with production-ready scripts.

## OLTP Database Backup

### Daily Full Backup

```bash
#!/bin/bash
# OLTP Daily Full Backup Script
set -e

DB_NAME="penguinmails_oltp"
BACKUP_DIR="/backups/oltp/$(date +%Y%m%d)"
S3_BUCKET="penguinmails-backups-us-east-1"
RETENTION_DAYS=90

echo "Starting OLTP full backup for $(date)"

# Create backup directory
mkdir -p "$BACKUP_DIR"

# Perform full backup with compression
pg_dump \
  --host=oltp-db.penguinmails.com \
  --username=backup_user \
  --format=custom \
  --compress=6 \
  --verbose \
  --file="$BACKUP_DIR/oltp_full_$(date +%H%M%S).dump" \
  "$DB_NAME"

# Upload to S3 with encryption
aws s3 sync "$BACKUP_DIR" "s3://$S3_BUCKET/oltp/$(date +%Y%m%d)/" \
  --server-side-encryption AES256 \
  --storage-class STANDARD_IA

echo "OLTP backup completed successfully"
```

### Hourly Incremental Backup (WAL Archiving)

```bash
#!/bin/bash
# OLTP Incremental Backup (WAL archiving)
set -e

WAL_ARCHIVE_DIR="/backups/oltp/wal_archive/$(date +%Y%m%d)"
S3_BUCKET="penguinmails-backups-us-east-1"

# Create WAL archive directory
mkdir -p "$WAL_ARCHIVE_DIR"

# Archive WAL files ready for archival
find /var/lib/postgresql/wal_archive -name "*.wal" -mmin +55 \
  -exec cp {} "$WAL_ARCHIVE_DIR"/ \;

# Upload WAL files to S3
aws s3 sync "$WAL_ARCHIVE_DIR" "s3://$S3_BUCKET/oltp/wal_archive/$(date +%Y%m%d)/" \
  --server-side-encryption AES256

echo "WAL archive completed: $(date)"
```

## Content Database Backup

### Daily Content Backup with Compression

```bash
#!/bin/bash
# Content Database Backup with Advanced Compression
set -e

DB_NAME="penguinmails_content"
BACKUP_DIR="/backups/content/$(date +%Y%m%d)"
S3_BUCKET="penguinmails-backups-us-east-1"

echo "Starting Content database backup"

mkdir -p "$BACKUP_DIR"

# Backup metadata (schema and small tables)
pg_dump \
  --host=content-db.penguinmails.com \
  --username=backup_user \
  --format=custom \
  --compress=9 \
  --no-owner \
  --exclude-table-data="content_objects" \
  --file="$BACKUP_DIR/content_metadata.dump" \
  "$DB_NAME"

# Backup content objects separately with maximum compression
pg_dump \
  --host=content-db.penguinmails.com \
  --username=backup_user \
  --format=custom \
  --compress=9 \
  --data-only \
  --table=content_objects \
  --file="$BACKUP_DIR/content_objects.dump" \
  "$DB_NAME"

# Upload to S3 with Glacier storage class
aws s3 sync "$BACKUP_DIR" "s3://$S3_BUCKET/content/$(date +%Y%m%d)/" \
  --server-side-encryption AES256 \
  --storage-class GLACIER

echo "Content backup completed"
```

## Queue System Backup

### Continuous Transaction Log Backup

```bash
#!/bin/bash
# Queue System Transaction Log Backup
set -e

DB_NAME="penguinmails_queue"
WAL_DIR="/var/lib/postgresql/queue_wal"
BACKUP_DIR="/backups/queue/wal/$(date +%Y%m%d)"
S3_BUCKET="penguinmails-backups-us-east-1"

echo "Starting Queue system WAL backup"

mkdir -p "$BACKUP_DIR"

# Copy WAL files ready for archival
find "$WAL_DIR" -name "*.wal" -type f -exec cp {} "$BACKUP_DIR"/ \;

# Upload to S3
aws s3 sync "$BACKUP_DIR" "s3://$S3_BUCKET/queue/wal/$(date +%Y%m%d)/" \
  --server-side-encryption AES256

echo "Queue WAL backup completed"
```

## OLAP Database Backup

### Weekly Snapshot Backup

```bash
#!/bin/bash
# OLAP Weekly Snapshot Backup
set -e

DB_NAME="penguinmails_olap"
BACKUP_DIR="/backups/olap/$(date +%Y%m%d)"
S3_BUCKET="penguinmails-backups-us-east-1"

echo "Starting OLAP snapshot backup"

mkdir -p "$BACKUP_DIR"

# Create full OLAP backup
pg_dump \
  --host=olap-db.penguinmails.com \
  --username=backup_user \
  --format=custom \
  --compress=6 \
  --file="$BACKUP_DIR/olap_snapshot.dump" \
  "$DB_NAME"

# Upload to analytics archive
aws s3 sync "$BACKUP_DIR" "s3://$S3_BUCKET/olap/$(date +%Y%m%d)/" \
  --server-side-encryption AES256 \
  --storage-class DEEP_ARCHIVE

echo "OLAP snapshot completed"
```

---

[← Back to Backup & Recovery](./README)
