---
title: "Infrastructure Management Tables"
description: "VPS instances, SMTP IPs, staff permissions, and system configuration for PenguinMails OLTP"
last_modified_date: "2025-12-15"
level: "2"
persona: "Engineering Teams"
keywords: "VPS, SMTP, IP addresses, infrastructure, staff, permissions, configuration, OLTP"
---

# Infrastructure Management Tables

Infrastructure tables manage the physical and logical resources that power email delivery. These include VPS instances, SMTP IP addresses, staff permissions, and system configuration.

## vps_instances Table

VPS instances host Mailu email servers and SMTP infrastructure. Each instance runs on Hostwinds and can host multiple SMTP IP addresses.

```sql
CREATE TABLE vps_instances (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    hostwinds_instance_id VARCHAR(255) UNIQUE,
    ip_address VARCHAR(45),
    region VARCHAR(50),
    status VARCHAR(50) CHECK (status IN ('active', 'provisioning', 'scheduled_decommission', 'decommissioned')),
    monthly_cost DECIMAL(10,2),
    approximate_cost DECIMAL(8,2),
    hostwinds_billing_day INTEGER,
    current_billing_period_start TIMESTAMP WITH TIME ZONE,
    current_billing_period_end TIMESTAMP WITH TIME ZONE,
    decommission_scheduled_for TIMESTAMP WITH TIME ZONE,
    created TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

The `approximate_cost` field enables cost attribution per tenant for profitability analysis. Business leaders use this data to track infrastructure costs and optimize resource allocation.

Status transitions follow a lifecycle: `provisioning` during initial setup, `active` for operational servers, `scheduled_decommission` when replacement is planned, and `decommissioned` for retired instances.

## smtp_ip_addresses Table

SMTP IP addresses are the individual sending IPs within a VPS instance. Each IP has its own reputation state and warmup status.

```sql
CREATE TABLE smtp_ip_addresses (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    vps_instance_id UUID REFERENCES vps_instances(id) ON DELETE CASCADE,
    ip_address VARCHAR(45) NOT NULL,
    status VARCHAR(50) CHECK (status IN ('available', 'assigned', 'warming', 'warmed', 'degraded', 'burned', 'quarantined')),
    reputation_state VARCHAR(50) CHECK (reputation_state IN ('good', 'fair', 'poor', 'critical')),
    approximate_cost DECIMAL(6,2),
    last_reputation_check TIMESTAMP WITH TIME ZONE,
    assigned_count INTEGER DEFAULT 0,
    provider_blacklist_status JSONB,
    created TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

### IP Status Values

- **available**: Ready for assignment to domains
- **assigned**: Currently allocated to a domain
- **warming**: Gradually building reputation through controlled sending
- **warmed**: Fully established reputation, ready for production volume
- **degraded**: Reputation issues detected, reduced sending recommended
- **burned**: Severely damaged reputation, not usable
- **quarantined**: Temporarily removed from rotation for investigation

The `provider_blacklist_status` JSONB field tracks listing status across major providers (Gmail, Outlook, Yahoo) and blacklist services.

## domain_ip_assignments Table

This junction table maps domains to SMTP IPs with assignment lifecycle tracking.

```sql
CREATE TABLE domain_ip_assignments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    domain_id UUID REFERENCES domains(id) ON DELETE CASCADE,
    smtp_ip_address_id UUID REFERENCES smtp_ip_addresses(id) ON DELETE CASCADE,
    status VARCHAR(50) CHECK (status IN ('active', 'warming', 'scheduled', 'expired')),
    assigned TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    expires TIMESTAMP WITH TIME ZONE,
    warmup_state VARCHAR(50) CHECK (warmup_state IN ('unwarmed', 'warming', 'warmed')),
    last_sent TIMESTAMP WITH TIME ZONE,
    created TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

Assignments can expire to support IP rotation strategies. The `warmup_state` field tracks domain-specific warmup progress separate from the IP's overall reputation.

## Staff and Permissions

Staff tables manage internal user permissions for platform administration.

### staff_members Table

```sql
CREATE TABLE staff_members (
    user_id UUID PRIMARY KEY REFERENCES users(id),
    role_id INTEGER REFERENCES staff_roles(id),
    notes TEXT,
    created TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

### staff_roles Table

```sql
CREATE TABLE staff_roles (
    id INTEGER PRIMARY KEY,
    name VARCHAR(50) UNIQUE,
    description TEXT,
    created TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

### permissions Table

```sql
CREATE TABLE permissions (
    id INTEGER PRIMARY KEY,
    name VARCHAR(100) UNIQUE,
    description TEXT,
    category VARCHAR(50) DEFAULT 'general',
    created TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

### staff_role_permissions Table

```sql
CREATE TABLE staff_role_permissions (
    role_id INTEGER REFERENCES staff_roles(id),
    permission_id INTEGER REFERENCES permissions(id),
    created TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    PRIMARY KEY (role_id, permission_id)
);
```

Staff permissions use integer IDs for efficiency. The junction table allows flexible role-permission combinations for administrative access control.

## System Configuration

### system_config Table

System-wide configuration stores platform settings that apply globally.

```sql
CREATE TABLE system_config (
    key VARCHAR(255) PRIMARY KEY,
    value JSONB,
    description VARCHAR(500),
    category VARCHAR(50),
    is_sensitive BOOLEAN DEFAULT FALSE,
    updated_by UUID REFERENCES users(id),
    updated TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

The `is_sensitive` flag indicates values that should be masked in logs and admin interfaces. Platform Stripe account configuration uses key `stripe_account_id`.

### feature_flags Table

```sql
CREATE TABLE feature_flags (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    key VARCHAR(100) UNIQUE NOT NULL,
    description TEXT,
    is_enabled BOOLEAN DEFAULT FALSE,
    rollout_percentage INTEGER DEFAULT 0,
    updated_by UUID REFERENCES users(id),
    updated TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

Feature flags support gradual rollouts through `rollout_percentage`. A flag with 25% rollout enables the feature for approximately one quarter of users.

### user_preferences Table

```sql
CREATE TABLE user_preferences (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id),
    theme VARCHAR(50) DEFAULT 'light',
    language VARCHAR(10) DEFAULT 'en',
    timezone VARCHAR(100) DEFAULT 'UTC',
    email_notifications BOOLEAN DEFAULT TRUE,
    push_notifications BOOLEAN DEFAULT TRUE,
    weekly_reports BOOLEAN DEFAULT FALSE,
    marketing_emails BOOLEAN DEFAULT FALSE,
    created TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

### tenant_config Table

```sql
CREATE TABLE tenant_config (
    tenant_id UUID PRIMARY KEY REFERENCES tenants(id),
    stripe_customer_id VARCHAR(255),
    billing_email VARCHAR(255),
    billing_address JSONB,
    notify_on_billing_changes BOOLEAN DEFAULT FALSE,
    notify_on_system_updates BOOLEAN DEFAULT FALSE,
    notify_on_security_alerts BOOLEAN DEFAULT FALSE,
    theme_primary_color VARCHAR(7),
    theme_logo_url VARCHAR(500),
    theme_favicon_url VARCHAR(500),
    ui_sidebar_default_collapsed BOOLEAN DEFAULT FALSE,
    ui_date_format VARCHAR(20),
    ui_timezone VARCHAR(50),
    created TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

Tenant configuration supports white-label customization through theme fields. The `stripe_customer_id` links to Stripe for billing operations.

### tenant_policies Table

```sql
CREATE TABLE tenant_policies (
    tenant_id UUID PRIMARY KEY REFERENCES tenants(id),
    password_min_length INTEGER DEFAULT 8,
    password_require_uppercase BOOLEAN DEFAULT FALSE,
    password_require_numbers BOOLEAN DEFAULT FALSE,
    password_require_symbols BOOLEAN DEFAULT FALSE,
    session_timeout_hours INTEGER DEFAULT 8,
    max_login_attempts INTEGER DEFAULT 5,
    two_factor_required BOOLEAN DEFAULT FALSE,
    company_default_status VARCHAR(50) DEFAULT 'active',
    company_allow_member_invites BOOLEAN DEFAULT TRUE,
    company_auto_approve_members BOOLEAN DEFAULT FALSE,
    company_require_email_verification BOOLEAN DEFAULT TRUE,
    created TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

Tenant policies enable enterprise customers to enforce security requirements on their users. These settings integrate with NileDB's authentication layer.

## Related Documentation

- [Schema Overview](overview)
- [Billing and Subscriptions](billing)
- [Performance and Security](performance-security)
