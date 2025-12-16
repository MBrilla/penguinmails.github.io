---
title: "Multi-Tenant Architecture Overview"
description: "Multi-tenant platform architecture and tenant isolation in PenguinMails"
last_modified_date: "2025-11-24"
level: "2"
persona: "Developers, System Architects"
status: "ACTIVE"
category: "Infrastructure"
---

# Multi-Tenant Architecture Overview

**Enterprise-grade multi-tenancy with complete data isolation and workspace management.**

## Overview

PenguinMails is built on a **multi-tenant architecture** powered by NileDB, providing complete tenant isolation, scalable workspace management, and secure data partitioning - all built into the platform from day one.

### What is Multi-Tenancy?

**Multi-tenancy** means multiple customers (tenants) share the same application infrastructure while maintaining complete data isolation and security.

**Benefits:**

- **Complete Isolation** - Tenant data is fully separated

- **Scalability** - Add unlimited tenants without infrastructure changes

- **Cost Efficiency** - Shared infrastructure reduces costs

- **Security** - Database-level tenant isolation

- **Performance** - Per-tenant query optimization

## Tenant Hierarchy

```text
Platform (PenguinMails)
  ├── Tenant (Company/Organization)
  │   ├── Users (Team Members)
  │   ├── Workspaces (Projects/Clients)
  │   ├── Subscription (Billing)
  │   └── Settings (Company-wide)
  │       ├── Workspace 1
  │       │   ├── Campaigns
  │       │   ├── Domains
  │       │   ├── Templates
  │       │   └── Contacts
  │       └── Workspace 2
  │           └── ...
```

### Tenant

Tenant = Company/Organization.

- Highest level of data isolation
- Maps to one paying customer
- Has one subscription
- Can have multiple users and workspaces
- Complete separation from other tenants

### Users

Users = Team Members within a Tenant.

- Belong to exactly one tenant
- Have roles (Owner, Admin, Member)
- Can access multiple workspaces within their tenant
- Cannot see data from other tenants

User roles:

- **Tenant Owner** - Full control, billing access
- **Admin** - Manage users, workspaces, settings
- **Member** - Access assigned workspaces only

### Workspaces

Workspaces = Projects, Clients, or Teams.

- Organize work within a tenant
- Optional sub-isolation for campaigns, contacts, domains
- Multiple users can collaborate in one workspace

---

## Related Documents

- [Tenant Isolation](/docs/features/infrastructure/multi-tenant-architecture/tenant-isolation) - Database-level isolation

- [Technical Implementation](/docs/features/infrastructure/multi-tenant-architecture/technical-implementation) - Schema and provisioning

- [Security Scalability](/docs/features/infrastructure/multi-tenant-architecture/security-scalability) - Security and scaling

- [NileDB Documentation](/docs/implementation-technical/database-infrastructure/niledb) - Database multi-tenancy
