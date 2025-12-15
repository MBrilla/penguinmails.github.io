---
title: "Performance & Security"
description: "Steps 4-5: Performance optimization and security access control"
last_modified_date: "2025-12-15"
level: "3"
parent: "database-migration-guide"
---

# Performance Optimization & Security

**Migration Steps:** 4 and 5 of 5  
**Components:** Performance Indexes, Role-Based Access Control

---

## Step 4: Performance Optimization

### Index Creation

```sql
-- Performance indexes for executive queries
CREATE INDEX IF NOT EXISTS idx_executive_business_summary_tenant
ON executive_business_summary(tenant_id);

CREATE INDEX IF NOT EXISTS idx_executive_business_summary_date
ON executive_business_summary(dashboard_date);

-- Additional cost optimization indexes
CREATE INDEX IF NOT EXISTS idx_business_cost_allocation_tenant
ON business_cost_allocation(tenant_id);

CREATE INDEX IF NOT EXISTS idx_business_cost_allocation_plan
ON business_cost_allocation(plan_name);

CREATE INDEX IF NOT EXISTS idx_business_cost_allocation_profit
ON business_cost_allocation(monthly_profit) WHERE monthly_profit > 0;
```

### Performance Benefits

| Metric | Target |
|--------|--------|
| **Executive Dashboard** | <3 second load times for business health summaries |
| **Cost Analysis** | <5 second query times for complex cost breakdowns |
| **Real-time Updates** | Immediate availability of cost data for live dashboards |
| **Scalability** | Support 100+ concurrent executive users |

---

## Step 5: Security & Access Control

### Permission Configuration

```sql
-- ============================================================================
-- SECURITY CONFIGURATION: Role-Based Access Control
-- ============================================================================

-- Grant select access to business role
GRANT SELECT ON business_cost_allocation TO business_analysts;
GRANT SELECT ON executive_business_summary TO business_analysts;

-- Grant select access to application service
GRANT SELECT ON business_cost_allocation TO application_service;
GRANT SELECT ON executive_business_summary TO application_service;

-- Grant full access to database administrators
GRANT ALL ON business_cost_allocation TO db_administrators;
GRANT ALL ON executive_business_summary TO db_administrators;

-- Row Level Security for multi-tenant isolation
CREATE POLICY executive_summary_tenant_isolation ON executive_business_summary
    FOR ALL USING (tenant_id = current_setting('app.current_tenant_id')::uuid);

CREATE POLICY cost_allocation_tenant_isolation ON business_cost_allocation
    FOR ALL USING (tenant_id = current_setting('app.current_tenant_id')::uuid);
```

### Access Control Matrix

| Role | business_cost_allocation | executive_business_summary |
|------|--------------------------|----------------------------|
| C-Suite Executives | SELECT | SELECT |
| VPs/Directors | SELECT | SELECT |
| Business Analysts | SELECT | SELECT |
| Database Administrators | ALL | ALL |
| Application Service | SELECT | SELECT |

---

## Security Considerations

### Multi-Tenant Isolation

Row Level Security (RLS) policies ensure that:
- Each tenant can only view their own data
- Cross-tenant data leakage is prevented at the database level
- Application-level security is reinforced by database-level controls

### Data Classification

| Data Type | Classification | Access Level |
|-----------|----------------|--------------|
| Approximate cost values | Internal Only | Admin/Finance roles |
| Business health metrics | Confidential | Executive roles |
| Tenant identifiers | Restricted | Application service |

---

## Related Documentation

- [Business Intelligence Views](./business-intelligence-views) - Previous migration step
- [Verification & Rollback](./verification-rollback) - Validation and emergency procedures
