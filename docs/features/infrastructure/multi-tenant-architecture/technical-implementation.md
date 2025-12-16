---
title: "Technical Implementation"
description: "Multi-tenant database schema, provisioning, and workspace management"
last_modified_date: "2025-11-24"
level: "2"
persona: "Developers, System Architects"
status: "ACTIVE"
category: "Infrastructure"
---

# Technical Implementation

## Tenant Provisioning

**Creating a new tenant:**

```javascript
// POST /api/v1/tenants
{
  "company_name": "Acme Marketing",
  "owner_email": "owner@acme.com",
  "owner_name": "John Doe",
  "plan": "professional"
}

// Backend Process:
// 1. Create tenant record in NileDB
// 2. Create owner user account
// 3. Initialize default workspace
// 4. Set up Stripe customer
// 5. Send welcome email
// 6. Return tenant_id and access token

Response:
{
  "tenant_id": "tenant_abc123",
  "owner_user_id": "user_xyz789",
  "default_workspace_id": "ws_default",
  "access_token": "eyJhbGc...",
  "onboarding_url": "/onboarding"
}
```

## Tenant Database Schema

```sql
-- Tenants Table (managed by NileDB)
CREATE TABLE tenants (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(255) NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  settings JSONB DEFAULT '{}'
);

-- Users Table (tenant-scoped)
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID REFERENCES tenants(id),
  email VARCHAR(255) NOT NULL,
  name VARCHAR(255),
  role VARCHAR(50), -- owner, admin, member
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

  UNIQUE(tenant_id, email) -- Email unique per tenant
);

-- Workspaces Table (tenant-scoped)
CREATE TABLE workspaces (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID REFERENCES tenants(id),
  name VARCHAR(255) NOT NULL,
  slug VARCHAR(255) NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

  UNIQUE(tenant_id, slug)
);

-- Workspace Members (access control)
CREATE TABLE workspace_members (
  workspace_id UUID REFERENCES workspaces(id),
  user_id UUID REFERENCES users(id),
  role VARCHAR(50), -- admin, member, viewer

  PRIMARY KEY(workspace_id, user_id)
);
```

## Tenant-Scoped Queries

**Application code example:**

```javascript
// Get campaigns for current tenant
async function getCampaigns(tenantId) {
  // NileDB automatically adds tenant_id filter
  const campaigns = await db.campaigns.findMany({
    where: {
      // tenant_id automatically added by NileDB
      status: 'active'
    }
  });
  return campaigns;
}

// Create campaign (tenant context from session)
async function createCampaign(req, campaignData) {
  const tenantId = req.user.tenant_id; // From JWT

  const campaign = await db.campaigns.create({
    data: {
      tenant_id: tenantId, // Explicit tenant assignment
      workspace_id: campaignData.workspace_id,
      name: campaignData.name,
      // ...
    }
  });

  return campaign;
}
```

## Workspace Management

### Creating Workspaces

**Workspaces organize work within a tenant:**

```javascript
// POST /api/v1/workspaces
{
  "name": "Client A - Holiday Campaign",
  "slug": "client-a-holiday",
  "description": "Q4 2025 marketing push"
}

Response:
{
  "workspace_id": "ws_abc123",
  "name": "Client A - Holiday Campaign",
  "slug": "client-a-holiday",
  "members": [
    {
      "user_id": "user_xyz",
      "role": "admin" // Creator is auto-admin
    }
  ]
}
```

### Workspace Access Control

**Assign users to workspaces:**

```javascript
// POST /api/v1/workspaces/{workspace_id}/members
{
  "user_id": "user_def456",
  "role": "member" // admin, member, viewer
}
```

**Workspace Roles:**

- **Admin** - Full control over workspace
- **Member** - Create/edit campaigns, manage contacts
- **Viewer** - Read-only access

---

## Related Documents

- [Overview](/docs/features/infrastructure/multi-tenant-architecture/overview) - Architecture overview

- [Tenant Isolation](/docs/features/infrastructure/multi-tenant-architecture/tenant-isolation) - Database-level isolation

- [Security Scalability](/docs/features/infrastructure/multi-tenant-architecture/security-scalability) - Security and scaling
