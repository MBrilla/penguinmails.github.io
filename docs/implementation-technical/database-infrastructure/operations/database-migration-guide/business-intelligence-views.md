---
title: "Business Intelligence Views"
description: "Step 3: Creating executive-level reporting views for business intelligence"
last_modified_date: "2025-12-15"
level: "3"
parent: "database-migration-guide"
---

# Business Intelligence Views Creation

**Migration Step:** 3 of 5  
**Component:** Executive Reporting Views

## Purpose

Provide executive-level reporting views for real-time business intelligence using modeled infrastructure cost approximations.

---

## Executive Summary View

### Migration SQL

```sql
-- ============================================================================
-- VIEW 1: Executive Business Summary
-- ============================================================================
-- Purpose: Provide C-Suite executives with internal business health metrics
-- using modeled infrastructure cost approximations, NOT direct provider billing.

CREATE OR REPLACE VIEW executive_business_summary AS
SELECT
    s.tenant_id,
    c.name as company_name,
    p.name as plan_name,
    s.current_period_start,
    s.current_period_end,

    -- Revenue and cost metrics
    COALESCE(p.price_monthly, 0) as monthly_subscription_revenue,
    COALESCE(SUM(vi.approximate_cost), 0) as infrastructure_costs,
    COALESCE(SUM(sia.approximate_cost), 0) as email_service_costs,

    -- Business performance indicators
    CASE
        WHEN (COALESCE(SUM(vi.approximate_cost), 0) + COALESCE(SUM(sia.approximate_cost), 0)) > 0
        THEN ROUND((p.price_monthly::decimal ), 0) + COALESCE(SUM(sia.approximate_cost), 0))::decimal), 2)
        ELSE 0
    END as gross_margin_ratio,

    -- Cost efficiency score (1-100)
    CASE
        WHEN p.price_monthly > 0 AND (COALESCE(SUM(vi.approximate_cost), 0) + COALESCE(SUM(sia.approximate_cost), 0)) > 0
        THEN LEAST(100, GREATEST(0, ROUND((p.price_monthly - (COALESCE(SUM(vi.approximate_cost), 0) + COALESCE(SUM(sia.approximate_cost), 0))) )
        ELSE 0
    END as cost_efficiency_score,

    -- Total cost base
    (COALESCE(SUM(vi.approximate_cost), 0) + COALESCE(SUM(sia.approximate_cost), 0)) as total_operational_cost,

    -- Health status for executive dashboard
    CASE
        WHEN p.price_monthly > (COALESCE(SUM(vi.approximate_cost), 0) + COALESCE(SUM(sia.approximate_cost), 0)) * 2 THEN 'Excellent'
        WHEN p.price_monthly > (COALESCE(SUM(vi.approximate_cost), 0) + COALESCE(SUM(sia.approximate_cost), 0)) * 1.5 THEN 'Good'
        WHEN p.price_monthly > (COALESCE(SUM(vi.approximate_cost), 0) + COALESCE(SUM(sia.approximate_cost), 0)) THEN 'Monitor'
        ELSE 'Critical Action Required'
    END as business_health_status,

    CURRENT_DATE as dashboard_date

FROM subscriptions s
JOIN companies c ON c.id = s.tenant_id
JOIN plans p ON p.id = s.plan_id
LEFT JOIN vps_instances vi ON vi.status = 'active'
LEFT JOIN smtp_ip_addresses sia ON sia.vps_instance_id = vi.id
WHERE s.status = 'active'
GROUP BY s.tenant_id, c.name, p.name, p.price_monthly, s.current_period_start, s.current_period_end;
```

### Executive Dashboard Queries

```sql
-- Daily Executive Health Check
SELECT
    company_name,
    plan_name,
    business_health_status,
    cost_efficiency_score,
    gross_margin_ratio,
    monthly_subscription_revenue,
    total_operational_cost,
    (monthly_subscription_revenue - total_operational_cost) as monthly_profit
FROM executive_business_summary
WHERE dashboard_date = CURRENT_DATE
ORDER BY cost_efficiency_score DESC;

-- Cost Optimization Opportunities
SELECT
    company_name,
    business_health_status,
    cost_efficiency_score,
    CASE
        WHEN cost_efficiency_score < 50 THEN 'High Optimization Potential'
        WHEN cost_efficiency_score < 75 THEN 'Moderate Optimization Potential'
        ELSE 'Well Optimized'
    END as optimization_category,
    (monthly_subscription_revenue * 2 - total_operational_cost) as optimization_opportunity
FROM executive_business_summary
WHERE business_health_status != 'Excellent'
ORDER BY optimization_opportunity DESC;
```

---

## Business Cost Allocation View

### Migration SQL

```sql
-- ============================================================================
-- VIEW 2: Business Cost Allocation
-- ============================================================================
-- Purpose: Provide detailed, heuristic cost breakdown for executive decision making
-- based on approximate_cost fields and payment data. These outputs are for
-- internal planning and margin analysis only, and do not replace Finance's
-- authoritative billing and accounting systems.

CREATE OR REPLACE VIEW business_cost_allocation AS
SELECT
    s.tenant_id,
    p.name as plan_name,
    p.price_monthly,

    -- Infrastructure costs from enhanced fields
    COALESCE(SUM(vi.approximate_cost), 0) as total_infrastructure_cost,
    COALESCE(SUM(sia.approximate_cost), 0) as total_email_service_cost,

    -- Business efficiency metrics
    CASE
        WHEN SUM(ba.emails_sent) > 0
        THEN ROUND((SUM(py.amount) + COALESCE(SUM(vi.approximate_cost), 0) + COALESCE(SUM(sia.approximate_cost), 0))::decimal )::decimal, 4)
        ELSE 0
    END as cost_per_email_delivered,

    -- Total business cost
    COALESCE(SUM(py.amount), 0) +
    COALESCE(SUM(vi.approximate_cost), 0) +
    COALESCE(SUM(sia.approximate_cost), 0) as total_monthly_cost,

    -- Profitability analysis
    COALESCE(SUM(py.amount), 0) - (COALESCE(SUM(vi.approximate_cost), 0) + COALESCE(SUM(sia.approximate_cost), 0)) as monthly_profit,

    -- Efficiency score (business KPIs)
    CASE
        WHEN (COALESCE(SUM(vi.approximate_cost), 0) + COALESCE(SUM(sia.approximate_cost), 0)) > 0
        THEN ROUND((COALESCE(SUM(py.amount), 0)::decimal ), 0) + COALESCE(SUM(sia.approximate_cost), 0))::decimal), 2)
        ELSE 0
    END as business_efficiency_ratio

FROM subscriptions s
JOIN plans p ON s.plan_id = p.id
LEFT JOIN billing_analytics ba ON ba.subscription_id = s.id
    AND ba.period_start >= CURRENT_DATE - INTERVAL '30 days'
LEFT JOIN vps_instances vi ON vi.status = 'active'
LEFT JOIN smtp_ip_addresses sia ON sia.vps_instance_id = vi.id
LEFT JOIN payments py ON py.subscription_id = s.id
    AND py.billing_period_start >= s.current_period_start
WHERE s.status = 'active'
GROUP BY s.tenant_id, p.name, p.price_monthly;
```

### Cost Analysis Queries

```sql
-- Monthly Cost Analysis by Tenant
SELECT
    plan_name,
    COUNT(*) as tenant_count,
    AVG(total_infrastructure_cost) as avg_infrastructure_cost,
    AVG(total_email_service_cost) as avg_email_service_cost,
    AVG(total_monthly_cost) as avg_total_cost,
    AVG(monthly_profit) as avg_monthly_profit,
    AVG(business_efficiency_ratio) as avg_efficiency_ratio
FROM business_cost_allocation
GROUP BY plan_name
ORDER BY avg_efficiency_ratio DESC;

-- Cost Per Email Analysis
SELECT
    company_name,
    plan_name,
    cost_per_email_delivered,
    total_monthly_cost,
    monthly_profit,
    business_efficiency_ratio,
    CASE
        WHEN cost_per_email_delivered < 0.01 THEN 'Highly Efficient'
        WHEN cost_per_email_delivered < 0.02 THEN 'Efficient'
        WHEN cost_per_email_delivered < 0.05 THEN 'Moderate'
        ELSE 'Needs Optimization'
    END as efficiency_category
FROM business_cost_allocation
WHERE cost_per_email_delivered > 0
ORDER BY cost_per_email_delivered ASC;
```

---

## Related Documentation

- [SMTP IP Cost Tracking](./smtp-ip-cost-tracking) - Previous migration step
- [Performance & Security](./performance-security) - Next migration step
- [Verification & Rollback](./verification-rollback) - Validation queries
