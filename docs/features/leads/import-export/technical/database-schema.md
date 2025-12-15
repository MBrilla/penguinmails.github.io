---
title: "Database Schema"
description: "PostgreSQL database schema for import and export job tracking"
last_modified_date: "2025-01-20"
level: 3
persona: Developers
keywords: database schema, PostgreSQL, import jobs, export jobs, rollback
---

# Import/Export Database Schema

The import/export system uses PostgreSQL tables to track job progress, store error details, and enable rollback functionality.

## Import Jobs Table

The import_jobs table stores the primary job configuration and results.

```sql
CREATE TABLE import_jobs (
  id UUID PRIMARY KEY,
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  workspace_id UUID REFERENCES workspaces(id),

  -- File information
  file_name VARCHAR(255),
  file_size BIGINT,
  file_url TEXT,  -- S3 URL
  file_type VARCHAR(50),  -- csv, xlsx, json

  -- Import configuration
  field_mapping JSONB,
  import_options JSONB,  -- duplicate strategy, tags, etc.

  -- Status
  status VARCHAR(50),  -- pending, processing, completed, failed, rolled_back

  -- Results
  total_rows INTEGER,
  imported_count INTEGER DEFAULT 0,
  updated_count INTEGER DEFAULT 0,
  created_count INTEGER DEFAULT 0,
  skipped_count INTEGER DEFAULT 0,
  error_count INTEGER DEFAULT 0,

  -- Error details
  errors JSONB,  -- Array of row errors

  -- Progress tracking
  processed_rows INTEGER DEFAULT 0,
  started_at TIMESTAMP,
  completed_at TIMESTAMP,

  -- User tracking
  created_by UUID REFERENCES users(id),
  created_at TIMESTAMP DEFAULT NOW(),

  -- Rollback support
  can_rollback BOOLEAN DEFAULT TRUE,
  rolled_back_at TIMESTAMP
);

CREATE INDEX idx_import_jobs_tenant ON import_jobs(tenant_id);
CREATE INDEX idx_import_jobs_status ON import_jobs(status);
CREATE INDEX idx_import_jobs_created ON import_jobs(created_at);
```

## Import Events Table

The import_events table stores change snapshots for rollback functionality.

```sql
CREATE TABLE import_events (
  id UUID PRIMARY KEY,
  import_job_id UUID NOT NULL REFERENCES import_jobs(id),
  contact_id UUID NOT NULL REFERENCES contacts(id),

  -- Event type
  event_type VARCHAR(50),  -- created, updated, skipped

  -- Snapshot of old data (for rollback)
  previous_data JSONB,
  new_data JSONB,

  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_import_events_job ON import_events(import_job_id);
CREATE INDEX idx_import_events_contact ON import_events(contact_id);
```

## Export Jobs Table

The export_jobs table tracks export operations and file storage.

```sql
CREATE TABLE export_jobs (
  id UUID PRIMARY KEY,
  tenant_id UUID NOT NULL REFERENCES tenants(id),

  -- Export configuration
  export_type VARCHAR(50),  -- full, filtered, segment
  filters JSONB,
  selected_fields JSONB,

  -- Output
  file_format VARCHAR(50),  -- csv, xlsx, json
  file_url TEXT,  -- S3 URL
  file_size BIGINT,

  -- Status
  status VARCHAR(50),  -- pending, processing, completed, failed

  -- Results
  total_contacts INTEGER,

  -- Progress
  processed_contacts INTEGER DEFAULT 0,
  started_at TIMESTAMP,
  completed_at TIMESTAMP,
  expires_at TIMESTAMP,  -- Auto-delete after 7 days

  created_by UUID REFERENCES users(id),
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_export_jobs_tenant ON export_jobs(tenant_id);
CREATE INDEX idx_export_jobs_status ON export_jobs(status);
```

## Related Documentation

- [Import Service](./import-service.md) - Processing logic
- [Export Service](./export-service.md) - Export generation
- [Background Jobs & API](./background-jobs-api.md) - Queue processing
