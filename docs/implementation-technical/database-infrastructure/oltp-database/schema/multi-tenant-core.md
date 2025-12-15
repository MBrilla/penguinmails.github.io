---
title: "Multi-Tenant Core Tables"
description: "NileDB-managed authentication and user identity tables for PenguinMails"
last_modified_date: "2025-12-15"
level: "2"
persona: "Engineering Teams"
keywords: "NileDB, multi-tenant, authentication, users, tenants, OLTP"
---

# Multi-Tenant Core Tables

NileDB manages user identity and multi-tenant associations as the foundation of PenguinMails' authentication system. These tables are created and maintained by NileDB's SDK, which handles token generation, session management, and tenant context automatically.

## users Table

The users table stores identity and profile information. NileDB populates this table during OAuth flows (Google, GitHub) or email-based registration.

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

The `deleted` column supports soft-delete patterns, allowing user data recovery within compliance windows. Profile fields like `picture` and `family_name` come from OAuth providers when available.

## tenants Table

Tenants represent organizations or workspaces that contain all business data. NileDB creates tenant records during organization setup and manages the `compute_id` for distributed compute assignment.

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

The `compute_id` field is reserved for NileDB's compute layer, which routes tenant operations to appropriate database shards. Application code does not modify this value.

## tenant_users Table

This junction table maps users to tenants with role assignments. A single user can belong to multiple tenants, each with different role sets.

```sql
CREATE TABLE tenant_users (
    tenant_id UUID REFERENCES tenants(id) ON DELETE CASCADE,
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    created TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    deleted TIMESTAMP WITH TIME ZONE,
    roles TEXT[] DEFAULT '{}',
    email TEXT,
    PRIMARY KEY (tenant_id, user_id)
);
```

The `roles` column is a PostgreSQL ARRAY type, which NileDB requires for its RBAC (Role-Based Access Control) system. Common role values include `owner`, `admin`, `member`, and `readonly`. The `email` field stores the user's email within the tenant context, which may differ from their primary identity email for service accounts or delegated access.

## NileDB Integration Points

NileDB's SDK sets the tenant context for each request using PostgreSQL session variables. All downstream queries automatically filter by the current tenant through Row Level Security policies.

```sql
-- NileDB sets this at the start of each request
SET app.current_tenant_id = 'tenant-uuid-here';
SET app.current_user_id = 'user-uuid-here';
```

Application code never manually queries or joins on `tenant_id` for isolation purposes. Instead, RLS policies attached to each table enforce tenant boundaries automatically. See [Performance and Security](performance-security) for RLS policy implementations.

## Related Documentation

- [Schema Overview](overview)
- [Business Logic Tables](business-logic)
- [Performance and Security](performance-security)
