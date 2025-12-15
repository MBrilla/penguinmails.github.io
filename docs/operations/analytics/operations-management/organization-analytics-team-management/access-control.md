---
title: "Access Control"
last_modified_date: "2025-12-15"
level: "2"
persona: "Security, Administrators"
---

# Access Control

Role-based permissions, security analytics, and compliance.

## Role Hierarchy

### Standard Roles

| Role | Description | Key Permissions |
|------|-------------|-----------------|
| **Owner** | Full company access | All permissions, billing, delete company |
| **Admin** | Team management | Invite/remove members, settings, campaigns |
| **Member** | Standard user | Create campaigns, view own analytics |

### Staff Roles (Internal)

| Role | Description | Access Scope |
|------|-------------|--------------|
| **Super Admin** | Platform administration | All tenants, all operations |
| **Admin** | Support escalation | Limited tenant access |
| **Support** | Customer assistance | Read-only tenant data |
| **QA** | Quality assurance | Test environments only |

## Permission Analytics

```typescript
interface AccessControlAnalytics {
  companyId: string;
  permissions: {
    totalPermissions: number;
    usedPermissions: number;
    unusedPermissions: number;
    permissionEfficiency: number;
  };
  roleAnalytics: {
    [roleType: string]: {
      count: number;
      averageTenure: number;
      activityScore: number;
      securityScore: number;
    };
  };
  securityMetrics: {
    unauthorizedAttempts: number;
    permissionEscalations: number;
    failedAccessAttempts: number;
    roleChangeFrequency: number;
  };
}
```

## Security Metrics

| Metric | Description | Target |
|--------|-------------|--------|
| Access Violation Rate | Unauthorized attempts | < 0.1% |
| Permission Utilization | Active use of permissions | > 70% |
| Role Change Audit | Changes per month | Documented |
| Multi-tenant Isolation | Cross-company leakage | 0 |

## Audit Trail Requirements

All staff operations touching sensitive data must be fully auditable:

| Field | Description |
|-------|-------------|
| Who | User ID and role |
| What | Operation performed |
| When | Timestamp |
| Which tenant | Tenant ID affected |
| Which resource | Resource ID modified |
| Result | Success/failure status |

## Compliance Controls

### Least Privilege Principle

- Staff roles follow clearly defined hierarchy
- Explicit permission sets per role
- No implicit permission inheritance
- Regular access reviews

### Audit Requirements

```sql
-- Example audit log query
SELECT 
  user_id,
  action,
  resource_type,
  resource_id,
  tenant_id,
  timestamp,
  result
FROM audit_logs
WHERE tenant_id = :tenant_id
  AND timestamp > NOW() - INTERVAL '30 days'
ORDER BY timestamp DESC;
```

---

[← Back to Organization Analytics](./README)
