---
title: "Security and Scalability"
description: "Multi-tenant security features, tenant settings, and scalability architecture"
last_modified_date: "2025-11-24"
level: "2"
persona: "Developers, System Architects"
status: "ACTIVE"
category: "Infrastructure"
---

# Security and Scalability

## Multi-Tenant Security

### Security Features

1. **Row-Level Security (RLS)**
   - Database enforces tenant isolation
   - Impossible to bypass tenant filters
   - Automatic on all queries

2. **API Middleware**
   - Validates tenant_id from JWT
   - Ensures user belongs to tenant
   - Rejects cross-tenant requests

3. **Tenant Switching Prevention**
   - Users cannot switch tenants
   - Must log out and log in to different account
   - No shared user accounts across tenants

4. **Audit Logging**
   - All tenant actions logged
   - Cross-tenant access attempts flagged
   - Compliance and security monitoring

### Security Validation

**Request flow:**

```text
User Request
  ↓
Extract JWT token
  ↓
Verify signature
  ↓
Extract tenant_id from token
  ↓
Validate user belongs to tenant
  ↓
Set tenant context for database
  ↓
Execute query (auto-filtered)
  ↓
Return results (tenant-scoped only)
```

## Tenant Settings

### Per-Tenant Configuration

**Customizable settings:**

```javascript
{
  "tenant_id": "tenant_abc123",
  "settings": {
    "company_name": "Acme Marketing",
    "company_address": "123 Main St, SF, CA",
    "timezone": "America/Los_Angeles",
    "date_format": "MM/DD/YYYY",
    "email_sender_name": "Acme Team",
    "default_reply_to": "hello@acme.com",
    "branding": {
      "logo_url": "https://...",
      "primary_color": "#007bff",
      "custom_domain": "mail.acme.com"
    },
    "limits": {
      "max_workspaces": 10,
      "max_users": 5,
      "email_sends_per_month": 50000
    }
  }
}
```

### Tenant-Specific Features

**Feature flags per tenant:**

- **Beta Features** - Early access for specific tenants
- **Custom Branding** - Enterprise tenant white-labeling
- **API Rate Limits** - Per-tenant throttling
- **Data Retention** - Custom retention policies

## Scalability

### Horizontal Scaling

**Multi-tenancy enables:**

- Add unlimited tenants without code changes
- Database sharding (future) - partition tenants across DBs
- Per-tenant performance optimization
- Isolated tenant upgrades/migrations

**Current Architecture:**

- All tenants share one NileDB instance
- Automatic query optimization per tenant
- Connection pooling shared across tenants

**Future Scaling (if needed):**

- Shard large tenants to dedicated databases
- Geo-distributed tenants (US/EU data centers)
- Tenant-specific resource allocation

## Best Practices

### For Tenants (Customers)

1. **Use Workspaces** - Organize by client, project, or team
2. **Assign Roles Carefully** - Give minimum necessary permissions
3. **Regular Audits** - Review workspace access quarterly
4. **Naming Conventions** - Use consistent workspace naming

### For Developers

1. **Always Use Tenant Context** - Never query without tenant_id
2. **Validate Tenant Access** - Check tenant membership on every request
3. **Test Isolation** - Verify tenant data separation in tests
4. **Audit Logs** - Log all tenant-scoped operations
5. **Never Hard-Code tenant_id** - Always from session/token

---

**Technology:** NileDB Multi-Tenant PostgreSQL
**Isolation Level:** Database Row-Level Security (RLS)

## Related Documents

- [Overview](/docs/features/infrastructure/multi-tenant-architecture/overview) - Architecture overview

- [Tenant Isolation](/docs/features/infrastructure/multi-tenant-architecture/tenant-isolation) - Database-level isolation

- [Technical Implementation](/docs/features/infrastructure/multi-tenant-architecture/technical-implementation) - Schema and provisioning

- [Free Mailbox Creation](/docs/features/infrastructure/free-mailbox-creation/overview) - Infrastructure provisioning

- [Authentication](/docs/implementation-technical/security/authentication) - Tenant-aware auth
