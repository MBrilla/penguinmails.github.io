---
title: "Core Database Tables"
description: "Core table schemas and relationships"
last_modified_date: "2025-12-04"
level: "3"
keywords: "tables, schema, core entities"
---

# Core Database Tables

## Naming Conventions

- **Core Entities**: No prefix, plural nouns (`users`, `companies`, `campaigns`)

- **Junction Tables**: Singular compound names (`tenant_users`, `campaign_sequence_steps`)

- **Configuration Tables**: Descriptive names (`user_preferences`, `tenant_config`)

- **System Tables**: Prefixed with table type (`system_config`, `feature_flags`)

**Table Name Examples:**

- `users` - User identity and profiles

- `tenants` - Tenant organizations

- `companies` - Tenant workspaces

- `domains` - Email sending domains

- `campaigns` - Campaign definitions

- `leads` - Contact database

- `templates` - Email templates

- `plans` - Subscription plans

- `subscriptions` - Active tenant subscriptions

---

## Overview

The **OLTP (Online Transaction Processing) Database** is PenguinMails' primary operational database designed for fast transactional operations, real-time data access, and primary business logic execution.

### Purpose & Characteristics

- **Primary Focus**: Fast queries, quick inserts, operational metadata

- **Performance**: Optimized for high-frequency operations and small record sizes

- **Architecture**: Normalized for data integrity, indexed for speed

- **Multi-Tenant**: Row Level Security (RLS) for complete tenant isolation

### Performance Strategy

- **Denormalized Fields**: `tenant_id` on operational tables for fast filtering

- **Index Coverage**: Covering indexes for common query patterns

- **Connection Pooling**: Aggressive pooling for high-throughput operations

- **Partitioning**: Consider date-based partitioning for large operational tables

---

## Core Multi-Tenant Infrastructure

### **NileDB-Managed Tables** (Authentication & User Management)

#### **users** - User Identity & Profile

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY,
    created TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    deleted TIMESTAMP WITH TIME ZONE,
    name TEXT,
    family_name TEXT,
    given_name TEXT,
    email TEXT,
    picture TEXT,
    email_verified TIMESTAMP WITH TIME ZONE
);


```

#### **tenants** - Tenant Organization

```sql
CREATE TABLE tenants (
    id UUID PRIMARY KEY,
    name TEXT,
    created TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    deleted TIMESTAMP WITH TIME ZONE,
    compute_id UUID
);


```

#### **tenant_users** - Multi-Tenant User Associations

```sql
CREATE TABLE tenant_users (
    tenant_id UUID REFERENCES tenants(id) ON DELETE CASCADE,
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    created TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    deleted TIMESTAMP WITH TIME ZONE,
    roles TEXT[] DEFAULT '{}' "NileDB-managed ARRAY - mandatory for authentication",
    email TEXT,
    PRIMARY KEY (tenant_id, user_id)
);


```

---

## Business Logic Tables

### **Company & Workspace Management**

#### **companies** - Tenant Workspaces (Marketing Access)

```sql
CREATE TABLE companies (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID REFERENCES tenants(id) ON DELETE CASCADE,
    name VARCHAR(100) NOT NULL,
    workspace_name VARCHAR(255) UNIQUE,
    logo_url TEXT,
    status VARCHAR(20) DEFAULT 'active',
    -- Marketing-accessible fields (no sensitive operational data)
    industry VARCHAR(100), -- Optional enrichment field
    company_size VARCHAR(50), -- Optional enrichment field (Small/Medium)
    created TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);


```

### **Domain Management**

#### **domains** - Email Sending Domains

```sql
CREATE TABLE domains (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL,
    company_id UUID REFERENCES companies(id) ON DELETE CASCADE,
    domain VARCHAR(253) NOT NULL,
    verification_status VARCHAR(50) DEFAULT 'pending',
    dns_records JSONB,
    is_primary BOOLEAN DEFAULT FALSE,
    verified TIMESTAMP WITH TIME ZONE,

    -- DNS Operational Metadata (Added for DNS Storage Integration)
    mailu_domain TEXT,                           -- Mailu's canonical hostname
    dns_ttl_observed INTEGER,                    -- Observed DNS TTL in seconds
    dns_last_verified_at TIMESTAMP WITH TIME ZONE, -- Last successful DNS verification
    dns_verification_error TEXT,                 -- Human-friendly error messages
    dns_verification_attempts INTEGER DEFAULT 0, -- Verification retry counter

    created TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);


```

#### **DNS Records JSON Structure** (dns_records JSONB)

The `dns_records` JSONB field stores per-record DNS configuration with lifecycle management:

```json
{
  "records": [
    {
      "record_type": "SPF",
      "name": "@",
      "value": "v=spf1 include:_spf.mailu.io include:_spf.sendgrid.net -all",
      "ttl": 3600,
      "verification_status": "verified",
      "last_verified_at": "2025-10-20T14:32:00Z",
      "verification_attempts": 1,
      "verification_error": null,
      "source": "ui"
    },
    {
      "record_type": "DKIM",
      "name": "default._domainkey",
      "selector": "default",
      "value": "v=DKIM1; k=rsa; p=MIGfMA0GCSqGSI....",
      "public_key": "MIGfMA0GCSqGSI....",
      "managed_by": "platform",              -- "platform" or "mailu"
      "secret_ref": "vault://mail/dkim/domain.com/default", -- Vault reference
      "needs_deployment": false,             -- Flag for Mailu sync jobs
      "ttl": 3600,
      "verification_status": "verified",
      "last_verified_at": "2025-10-20T14:32:00Z",
      "verification_attempts": 1,
      "verification_error": null,
      "source": "platform"
    },
    {
      "record_type": "DMARC",
      "name": "_dmarc",
      "value": "v=DMARC1; p=quarantine; rua=mailto:dmarc@domain.com",
      "policy": "quarantine",
      "ttl": 3600,
      "verification_status": "verified",
      "last_verified_at": "2025-10-20T14:32:00Z",
      "verification_attempts": 1,
      "verification_error": null,
      "source": "ui"
    },
    {
      "record_type": "MX",
      "name": "@",
      "value": "mail.penguinmails.io.",
      "priority": 10,
      "ttl": 3600,
      "verification_status": "verified",
      "last_verified_at": "2025-10-20T14:40:00Z",
      "mx_hosts": ["203.0.113.5"],
      "verification_attempts": 1,
      "verification_error": null,
      "source": "ui"
    }
  ]
}


```

**DNS Record Field Meanings**:

- `record_type`: DNS record type (SPF, DKIM, DMARC, MX, A, AAAA, CNAME)

- `name`: DNS record name (@ for root, subdomain for others)

- `value`: Expected DNS record value

- `verification_status`: Current verification state (pending, verified, failed, error)

- `last_verified_at`: Timestamp of last successful verification

- `verification_attempts`: Number of verification attempts

- `verification_error`: Error message from last failed verification

- `managed_by`: Who manages the record ("platform" or "mailu")

- `secret_ref`: Vault reference for DKIM private keys (platform-managed only)

- `needs_deployment`: Flag for Mailu sync jobs to deploy DKIM keys

- `source`: Source of record creation ("ui", "platform", "api")

#### **email_accounts** - Email Account Configuration

```sql
CREATE TABLE email_accounts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL,
    domain_id UUID REFERENCES domains(id) ON DELETE CASCADE,
    email VARCHAR(255) NOT NULL,
    provider VARCHAR(50) DEFAULT 'mailu',
    vault_key_path VARCHAR(500),
    imap_settings JSONB,
    smtp_settings JSONB,
    status VARCHAR(50) DEFAULT 'active',
    last_warmed TIMESTAMP WITH TIME ZONE,
    daily_count INTEGER DEFAULT 0,
    created TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);


```

### **Lead Management**

#### **leads** - Contact Database

```sql
CREATE TABLE leads (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL,
    email VARCHAR(255) NOT NULL,
    name VARCHAR(200),
    company VARCHAR(200),
    status VARCHAR(20) DEFAULT 'active',
    updated TIMESTAMP WITH TIME ZONE,

    UNIQUE(tenant_id, email)
);


```

### **Template Management**

#### **templates** - Email Templates

```sql
CREATE TABLE templates (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL,
    name VARCHAR(100) NOT NULL,
    subject VARCHAR(255),
    content TEXT,
    is_started BOOLEAN DEFAULT FALSE,
    updated TIMESTAMP WITH TIME ZONE,
    created TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);


```

#### **Template Organization**

```sql
-- Template folders for organization (UUID → BIGINT optimization)
CREATE TABLE folders (
    id BIGINT PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
    name TEXT NOT NULL,
    updated TIMESTAMP WITH TIME ZONE,
    created TIMESTAMP WITH TIME ZONE DEFAULT NOW()
