---
title: "Business Logic Tables"
description: "Companies, domains, leads, and template management for PenguinMails OLTP"
last_modified_date: "2025-12-15"
level: "2"
persona: "Engineering Teams"
keywords: "companies, domains, leads, templates, business logic, OLTP"
---

# Business Logic Tables

Business logic tables store the core entities that power PenguinMails' email marketing operations. These tables handle company workspaces, domain configuration, lead management, and template organization.

## companies Table

Companies represent tenant workspaces where marketing teams organize their campaigns. Each tenant can have multiple companies, allowing agencies to manage separate client accounts within a single subscription.

```sql
CREATE TABLE companies (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID REFERENCES tenants(id) ON DELETE CASCADE,
    name VARCHAR(100) NOT NULL,
    workspace_name VARCHAR(255) UNIQUE,
    logo_url TEXT,
    status VARCHAR(20) DEFAULT 'active',
    industry VARCHAR(100),
    company_size VARCHAR(50),
    created TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

The `workspace_name` field provides a unique identifier for URL-based access patterns. Marketing-accessible fields like `industry` and `company_size` support lead enrichment without exposing operational internals.

## domains Table

Email sending domains contain DNS configuration and verification status. The platform verifies SPF, DKIM, DMARC, and MX records to ensure deliverability before allowing email sends.

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
    mailu_domain TEXT,
    dns_ttl_observed INTEGER,
    dns_last_verified_at TIMESTAMP WITH TIME ZONE,
    dns_verification_error TEXT,
    dns_verification_attempts INTEGER DEFAULT 0,
    created TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

### DNS Records Structure

The `dns_records` JSONB field stores per-record DNS configuration with lifecycle management:

```json
{
  "records": [
    {
      "record_type": "SPF",
      "name": "@",
      "value": "v=spf1 include:_spf.mailu.io -all",
      "verification_status": "verified",
      "last_verified_at": "2025-10-20T14:32:00Z",
      "verification_attempts": 1,
      "source": "ui"
    },
    {
      "record_type": "DKIM",
      "name": "default._domainkey",
      "selector": "default",
      "managed_by": "platform",
      "secret_ref": "vault://mail/dkim/domain.com/default",
      "verification_status": "verified"
    }
  ]
}
```

DKIM records include a `secret_ref` field pointing to Vault for private key storage. The `managed_by` field distinguishes between platform-generated keys and those created by Mailu.

## email_accounts Table

Email accounts represent individual sending addresses within a domain. Each account links to SMTP settings and warmup state for deliverability tracking.

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

The `vault_key_path` field stores references to credentials in HashiCorp Vault rather than storing passwords directly. The `daily_count` field tracks sending volume for warmup algorithms.

## leads Table

Leads are contact records that receive campaign emails. The table uses a composite unique constraint on tenant and email to prevent duplicates within each tenant's database.

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

Lead status values include `active`, `unsubscribed`, `bounced`, and `complained`. Status transitions are tracked in the analytics database for compliance reporting.

## templates Table

Templates store email content with variable placeholders for personalization. The Eta templating engine renders these templates during campaign execution.

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

### Template Organization

Templates can be organized using folders and tags:

```sql
CREATE TABLE folders (
    id BIGINT PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
    name TEXT NOT NULL,
    updated TIMESTAMP WITH TIME ZONE,
    created TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE TABLE template_folders (
    id BIGINT PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
    template_id UUID REFERENCES templates(id) ON DELETE CASCADE,
    folder_id BIGINT REFERENCES folders(id) ON DELETE CASCADE,
    UNIQUE(template_id, folder_id)
);

CREATE TABLE tags (
    id BIGINT PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
    value TEXT NOT NULL,
    description TEXT,
    tenant_id UUID,
    is_default BOOLEAN DEFAULT FALSE
);

CREATE TABLE template_tags (
    id BIGINT PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
    template_id UUID REFERENCES templates(id) ON DELETE CASCADE,
    tag_id BIGINT REFERENCES tags(id) ON DELETE CASCADE,
    tenant_id UUID NOT NULL,
    UNIQUE(template_id, tag_id)
);
```

These junction tables use BIGINT identity columns instead of UUIDs for storage efficiency in high-volume tagging scenarios.

### Eta Rendering Storage

For pre-compiled template rendering, the `email_templates` table stores Eta-specific metadata:

```sql
CREATE TABLE email_templates (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL,
    name TEXT NOT NULL,
    subject TEXT NOT NULL,
    body TEXT NOT NULL,
    async BOOLEAN DEFAULT FALSE,
    UNIQUE (tenant_id, name)
);

CREATE TABLE template_dictionaries (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL,
    key TEXT NOT NULL,
    value JSONB NOT NULL
);
```

Template dictionaries provide shared variable values across multiple templates, reducing duplication for common elements like company addresses or social links.

## Related Documentation

- [Schema Overview](overview)
- [Multi-Tenant Core](multi-tenant-core)
- [Campaign Management](campaigns)
