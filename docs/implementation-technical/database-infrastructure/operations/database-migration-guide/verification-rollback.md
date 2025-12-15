---
title: "Verification & Rollback"
description: "Migration verification, data integrity validation, and rollback procedures"
last_modified_date: "2025-12-15"
level: "3"
parent: "database-migration-guide"
---

# Migration Verification & Rollback

This document covers validation queries and emergency rollback procedures for the database migration.

---

## Data Integrity Validation

### Migration Success Queries

```sql
-- Verify new columns exist
SELECT
    table_name,
    column_name,
    data_type,
    is_nullable,
    column_default
FROM information_schema.columns
WHERE table_name IN ('vps_instances', 'smtp_ip_addresses')
  AND column_name = 'approximate_cost';

-- Expected Results:
-- ├── vps_instances.approximate_cost: DECIMAL(8,2), NULL allowed, DEFAULT 0.00
-- ├── smtp_ip_addresses.approximate_cost: DECIMAL(6,2), NULL allowed, DEFAULT 0.00
-- └── All columns have proper business comments
```

---

## Business Logic Validation

### Test Business View Accuracy

```sql
-- Test business view accuracy
SELECT
    tenant_id,
    plan_name,
    monthly_subscription_revenue,
    total_infrastructure_cost,
    total_email_service_cost,
    total_monthly_cost,
    monthly_profit,
    business_health_status
FROM executive_business_summary
WHERE dashboard_date = CURRENT_DATE
LIMIT 5;
```

### Validate Calculations

```sql
-- Validate calculations
WITH cost_validation AS (
    SELECT
        tenant_id,
        (total_infrastructure_cost + total_email_service_cost) as calculated_total_cost,
        (monthly_subscription_revenue - (total_infrastructure_cost + total_email_service_cost)) as calculated_profit
    FROM executive_business_summary
    WHERE dashboard_date = CURRENT_DATE
)
SELECT
    evs.tenant_id,
    evs.total_operational_cost,
    cv.calculated_total_cost,
    CASE WHEN evs.total_operational_cost = cv.calculated_total_cost THEN 'PASS' ELSE 'FAIL' END as cost_validation,
    evs.monthly_profit,
    cv.calculated_profit,
    CASE WHEN ABS(evs.monthly_profit - cv.calculated_profit) < 0.01 THEN 'PASS' ELSE 'FAIL' END as profit_validation
FROM executive_business_summary evs
JOIN cost_validation cv ON evs.tenant_id = cv.tenant_id
WHERE evs.dashboard_date = CURRENT_DATE;
```

---

## Performance Validation

### Test Query Performance

```sql
-- Test query performance
EXPLAIN ANALYZE
SELECT * FROM executive_business_summary
WHERE dashboard_date = CURRENT_DATE
ORDER BY cost_efficiency_score DESC;
```

### Performance Targets

| Metric | Target |
|--------|--------|
| Query execution time | <500ms for executive summary |
| Index utilization | 100% for filtered queries |
| Memory usage | <100MB for complex aggregations |
| Concurrent query handling | 100+ simultaneous queries |

---

## Rollback Procedures

### Emergency Rollback

⚠️ **Use Only in Emergency**

```sql
-- ============================================================================
-- ROLLBACK PROCEDURES (Use Only in Emergency)
-- ============================================================================

-- Remove business views
DROP VIEW IF EXISTS executive_business_summary;
DROP VIEW IF EXISTS business_cost_allocation;

-- Remove indexes
DROP INDEX IF EXISTS idx_vps_instances_approximate_cost;
DROP INDEX IF EXISTS idx_smtp_ip_addresses_approximate_cost;
DROP INDEX IF EXISTS idx_executive_business_summary_tenant;
DROP INDEX IF EXISTS idx_executive_business_summary_date;

-- Remove constraints
ALTER TABLE vps_instances DROP CONSTRAINT IF EXISTS chk_vps_approximate_cost_positive;
ALTER TABLE smtp_ip_addresses DROP CONSTRAINT IF EXISTS chk_smtp_approximate_cost_positive;

-- Remove columns (LAST RESORT - only if absolutely necessary)
-- ALTER TABLE vps_instances DROP COLUMN IF EXISTS approximate_cost;
-- ALTER TABLE smtp_ip_addresses DROP COLUMN IF EXISTS approximate_cost;
```

### Important Warning

> **Warning:** Only use column removal as absolute last resort, as this will permanently lose business data. Prefer to set `approximate_cost` to `0.00` instead.

### Rollback Decision Matrix

| Issue | Recommended Action |
|-------|-------------------|
| View returns incorrect data | Drop and recreate view |
| Index causing performance issues | Drop specific index |
| Constraint too restrictive | Drop and recreate constraint |
| Column data corruption | Set values to 0.00, do not drop column |
| Complete migration failure | Execute full rollback script |

---

## Related Documentation

- [Overview & Strategy](./overview-strategy) - Migration overview
- [Performance & Security](./performance-security) - Previous migration step
- [README](./README) - Main documentation index
