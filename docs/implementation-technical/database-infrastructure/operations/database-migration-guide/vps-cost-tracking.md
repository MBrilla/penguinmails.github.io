---
title: "VPS Instances Cost Tracking"
description: "Step 1: Adding approximate cost tracking to VPS instances"
last_modified_date: "2025-12-15"
level: "3"
parent: "database-migration-guide"
---

# VPS Instances Cost Tracking Enhancement

**Migration Step:** 1 of 5  
**Component:** VPS Instances Cost Reference

## Business Purpose

Provide an internal, approximate reference for infrastructure cost attribution to support executive profitability analysis, recognizing that:

- PenguinMails operates shared NileDB-backed infrastructure (e.g. flat fee + storage tiers + per-GB overages)
- Underlying providers (such as NileDB) may not expose precise per-tenant metering APIs
- Per-tenant values stored here are modeled by Finance/Operations, not pulled as exact real-time provider charges

---

## Migration SQL

```sql
-- ============================================================================
-- ENHANCEMENT 1: VPS Instances Cost Tracking
-- ============================================================================
-- Business Justification (Clarified):
-- - Internal-only reference for Hostwinds VPS costs, maintained by Finance & Operations.
-- - Supports directional margin and efficiency analysis per tenant.
-- - Not an authoritative or itemized billing source for customers.
-- - Final truth for spend remains Hostwinds invoices and Finance systems.

-- Add approximate cost column
ALTER TABLE vps_instances
ADD COLUMN IF NOT EXISTS approximate_cost DECIMAL(8,2) DEFAULT 0.00;

-- Add business-focused comments for documentation
COMMENT ON COLUMN vps_instances.approximate_cost IS
'Estimated monthly cost in USD for this Hostwinds VPS instance, derived from Hostwinds pricing APIs
(get_price_list, get_billingcycle_prices, upgrade_instance) using the active plan/specs and billing cycle.
Primary intent: track Hostwinds-related infrastructure spend per instance for internal ROI/margin analysis.
Not authoritative customer billing; final invoices remain in Finance systems.';

-- Add check constraint to ensure non-negative costs
ALTER TABLE vps_instances
ADD CONSTRAINT chk_vps_approximate_cost_positive
CHECK (approximate_cost >= 0);

-- Create index for efficient cost-based queries
CREATE INDEX IF NOT EXISTS idx_vps_instances_approximate_cost
ON vps_instances(approximate_cost) WHERE status = 'active';
```

---

## Business Impact

### Benefits (Clarified)

| Benefit | Description |
|---------|-------------|
| **Internal Visibility** | CFOs and Finance can see directional infrastructure cost allocations per tenant based on Hostwinds-modeled values |
| **Budget Planning** | Finance teams can forecast costs using approximations aligned to Hostwinds pricing, not NileDB metering |
| **Margin Analysis** | Product managers can estimate customer/unit economics using these internal references |
| **Cost Optimization** | Identify over-provisioned resources and right-sizing opportunities |

### Source of Truth Reminder

All analyses must be reconciled to official provider invoices and Finance ledgers.

---

## Cost Attribution Examples

```markdown
Tenant A: 3 VPS instances × $150 = $450/month infrastructure cost
Tenant B: 2 VPS instances × $200 = $400/month infrastructure cost
Tenant C: 1 VPS instance × $100 = $100/month infrastructure cost
```

---

## Column Specification

| Attribute | Value |
|-----------|-------|
| **Column Name** | `approximate_cost` |
| **Data Type** | `DECIMAL(8,2)` |
| **Default** | `0.00` |
| **Constraint** | Non-negative (≥ 0) |
| **Index** | Partial index on active instances |

---

## Related Documentation

- [SMTP IP Cost Tracking](./smtp-ip-cost-tracking) - Next migration step
- [Business Intelligence Views](./business-intelligence-views) - Views consuming this data
- [Verification & Rollback](./verification-rollback) - Validation queries
