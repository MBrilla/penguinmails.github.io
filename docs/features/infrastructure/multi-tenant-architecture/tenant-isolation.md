---
title: "Tenant Isolation"
description: "Database-level tenant isolation and authentication in PenguinMails"
last_modified_date: "2025-11-24"
level: "2"
persona: "Developers, System Architects"
status: "ACTIVE"
category: "Infrastructure"
---

# Tenant Isolation

## Database-Level Isolation

NileDB provides native multi-tenancy:

```sql
-- Every table has tenant_id
CREATE TABLE campaigns (
  id UUID PRIMARY KEY,
  tenant_id UUID REFERENCES tenants(id), -- Automatic isolation
  workspace_id UUID REFERENCES workspaces(id),
  name VARCHAR(255),
  created_at TIMESTAMP
);

-- NileDB automatically filters by tenant
SELECT * FROM campaigns WHERE name = 'Welcome Series';
-- Becomes: SELECT * FROM campaigns WHERE tenant_id = {current_tenant} AND name = 'Welcome Series';
```

Automatic tenant context:

- All queries automatically scoped to current tenant
- Impossible to accidentally query another tenant's data
- Row-level security enforced at database level
- No application-level filtering needed

## Authentication & Tenant Context

How tenant context is established:

1. **User logs in** with email/password
2. **NileDB authenticates** and identifies tenant
3. **Session includes tenant_id** in JWT token
4. **All API requests** automatically scoped to that tenant
5. **Database queries** filtered by tenant_id

**Session Token:**

```javascript
{
  "user_id": "user_abc123",
  "tenant_id": "tenant_xyz789", // Current tenant
  "email": "user@acme.com",
  "role": "admin",
  "workspaces": ["ws_1", "ws_2"]
}
```

## Data Isolation Guarantees

**What's Isolated:**

- ✅ **Campaigns** - Tenant A cannot see Tenant B's campaigns
- ✅ **Contacts** - Complete contact list separation
- ✅ **Templates** - Email templates not shared
- ✅ **Analytics** - Performance data isolated
- ✅ **Workspaces** - Workspace data tenant-scoped
- ✅ **Domains** - Domain configurations isolated
- ✅ **Users** - User accounts tenant-specific

**What's Shared (Platform-Level):**

- ⚙️ **Application Code** - Same codebase for all tenants
- ⚙️ **Infrastructure** - Shared servers (with isolation)
- ⚙️ **Global Suppression** - Platform-wide spam/abuse blocks

---

## Related Documents

- [Overview](/docs/features/infrastructure/multi-tenant-architecture/overview) - Architecture overview

- [Technical Implementation](/docs/features/infrastructure/multi-tenant-architecture/technical-implementation) - Schema and provisioning

- [Security Scalability](/docs/features/infrastructure/multi-tenant-architecture/security-scalability) - Security and scaling
