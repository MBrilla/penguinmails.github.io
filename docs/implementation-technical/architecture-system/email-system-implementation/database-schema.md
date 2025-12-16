---
title: "Email System Database Schema"
description: "Database schema definitions and migration strategies for email system"
last_modified_date: "2025-01-09"
level: "2"
persona: "Database Engineers, Backend Developers"
---

# Email System Database Schema

## Natural Database Relationships

```sql
-- Query intent is immediately clear
SELECT m.subject, c.content_text, a.filename
FROM email_messages m
JOIN email_content c ON m.storage_key = c.storage_key
LEFT JOIN attachments a ON c.storage_key = a.parent_storage_key
WHERE m.tenant_id = $1;
```

## Rich Analytics Capabilities

```sql
-- Email size and attachment analysis
SELECT
    m.direction,
    m.status,
    COUNT(*) as email_count,
    AVG(c.raw_size_bytes) as avg_size_bytes,
    COUNT(a.*) as total_attachments
FROM email_messages m
JOIN email_content c ON m.storage_key = c.storage_key
LEFT JOIN attachments a ON c.storage_key = a.parent_storage_key
WHERE m.tenant_id = $1
GROUP BY m.direction, m.status;
```

---

## Data Flow Architecture

### Complete Email Processing Flow

```markdown
1. Event Reception: IMAP/Cronjob/Endpoint → Queue Producer
2. Queue Entry: Redis Queue with job classification
3. Handler Processing: Type-specific processing (inbound/bounce)
4. Database Storage: email_messages (analytics) + email_content (content)
5. Analytics Pipeline: Queue → OLAP analytics aggregation
```

### Cross-Database Relationships

```sql
-- OLTP to Content Database references
email_messages.campaign_id → campaigns.id (OLTP)
email_messages.lead_id → leads.id (OLTP)
email_messages.email_account_id → email_accounts.id (OLTP)

-- Content Database internal references
email_messages.storage_key → email_content.storage_key
email_content.storage_key → attachments.parent_storage_key
```

---

## Migration from Legacy Structure

### Migration Strategy

```sql
-- Step 1: Create new tables with clear names
CREATE TABLE email_messages (
    storage_key VARCHAR(500) PRIMARY KEY REFERENCES email_content(storage_key),
    tenant_id UUID NOT NULL,
    email_account_id UUID,
    campaign_id UUID REFERENCES campaigns(id),
    lead_id UUID REFERENCES leads(id),
    parent_message_id UUID REFERENCES email_messages(storage_key),
    direction VARCHAR(20) CHECK (direction IN ('inbound', 'outbound')),
    message_type VARCHAR(20) CHECK (message_type IN ('email', 'bounce', 'auto_reply')),
    from_email VARCHAR(254),
    to_email VARCHAR(254),
    subject VARCHAR(500),
    status VARCHAR(50),
    processed TIMESTAMP WITH TIME ZONE,
    created TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE TABLE email_content (
    storage_key VARCHAR(500) PRIMARY KEY,
    tenant_id UUID NOT NULL,
    content_text TEXT,
    content_html TEXT,
    headers JSONB,
    raw_size_bytes INTEGER,
    compressed_size_bytes INTEGER,
    content_hash VARCHAR(64),
    compression_algorithm VARCHAR(20),
    created TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    expires TIMESTAMP WITH TIME ZONE,
    retention_days INTEGER DEFAULT 2555
);

-- Step 2: Migrate data
INSERT INTO email_messages
SELECT * FROM content_inbox_message_refs;

INSERT INTO email_content
SELECT * FROM content_objects;

-- Step 3: Update foreign key references
-- Update attachments.parent_storage_key to reference email_content

-- Step 4: Archive old tables
ALTER TABLE content_inbox_message_refs RENAME TO content_inbox_message_refs_archived;
ALTER TABLE content_objects RENAME TO content_objects_archived;
```

### Data Validation

```sql
-- Verify migration integrity
SELECT
    'Total migrated email_messages' as check_type,
    COUNT(*) as count
FROM email_messages
UNION ALL
SELECT
    'Total migrated email_content' as check_type,
    COUNT(*) as count
FROM email_content
UNION ALL
SELECT
    'Messages with content' as check_type,
    COUNT(*) as count
FROM email_messages m
JOIN email_content c ON m.storage_key = c.storage_key
UNION ALL
SELECT
    'Orphaned messages' as check_type,
    COUNT(*) as count
FROM email_messages m
LEFT JOIN email_content c ON m.storage_key = c.storage_key
WHERE c.storage_key IS NULL;
```

---

## Design Benefits Achieved

### Clear Separation

- **OLTP Focus**: campaign_sequence_steps handles operational campaign execution
- **Content Focus**: email_messages handles analytics, email_content handles content
- **Queue Integration**: Perfect fit with 4-step queue architecture
- **Mailu Ready**: Sets foundation for external email system integration

### Developer Experience Enhanced

- **Intuitive Names**: New developers immediately understand the schema
- **Natural Relationships**: Logical foreign key hierarchy
- **Better APIs**: Endpoint design becomes clear and logical
- **Maintainability**: Database structure easier to understand and extend

---

## Related Sections

- [Overview](/docs/implementation-technical/architecture-system/email-system-implementation/overview) - Architecture overview
- [Queue Integration](/docs/implementation-technical/architecture-system/email-system-implementation/queue-integration) - Queue processing
- [Performance Monitoring](/docs/implementation-technical/architecture-system/email-system-implementation/performance-monitoring) - Metrics
