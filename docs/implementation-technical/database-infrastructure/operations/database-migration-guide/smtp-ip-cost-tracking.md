---
title: "SMTP IP Addresses Cost Tracking"
description: "Step 2: Adding approximate cost tracking to SMTP IP addresses"
last_modified_date: "2025-12-15"
level: "3"
parent: "database-migration-guide"
---

# SMTP IP Addresses Cost Tracking Enhancement

**Migration Step:** 2 of 5  
**Component:** SMTP IP Cost Reference

## Business Purpose

Support infrastructure and deliverability cost analysis and ROI calculations for Hostwinds-backed resources.

---

## Migration SQL

```sql
-- ============================================================================
-- ENHANCEMENT 2: SMTP IP Addresses Cost Tracking
-- ============================================================================
-- Business Justification: Support deliverability cost analysis and email service
-- ROI calculations for optimization of IP allocation and pricing strategy
-- Expected Impact: 10-15% monthly savings through better IP management

-- Add approximate cost column
ALTER TABLE smtp_ip_addresses
ADD COLUMN IF NOT EXISTS approximate_cost DECIMAL(6,2) DEFAULT 0.00;

-- Add business-focused comments for documentation
COMMENT ON COLUMN smtp_ip_addresses.approximate_cost IS
'Internally maintained estimated monthly cost in USD per SMTP IP address used for deliverability and cost-efficiency analysis.
Not authoritative billing; values are modeled/allocated by Finance & Operations.';

-- Add check constraint to ensure non-negative costs
ALTER TABLE smtp_ip_addresses
ADD CONSTRAINT chk_smtp_approximate_cost_positive
CHECK (approximate_cost >= 0);

-- Create index for efficient cost-based queries
CREATE INDEX IF NOT EXISTS idx_smtp_ip_addresses_approximate_cost
ON smtp_ip_addresses(approximate_cost) WHERE status IN ('active', 'warmed', 'warming');
```

---

## Business Impact

| Benefit | Description |
|---------|-------------|
| **Deliverability ROI** | Calculate cost per successful email delivery |
| **Resource Optimization** | Identify over-provisioned IP addresses |
| **Competitive Analysis** | Benchmark email service costs vs. industry |
| **IP Management** | Optimize IP warmup and allocation strategies |

---

## SMTP IP Approximate Cost Model (Clarified)

### Pricing Baseline

- Hostwinds API does not expose a clean, standalone "per IP" price
- Internal testing confirms an effective baseline of **$4.99/month per dedicated IP** for our current configuration

### Implementation Guidance

| Consideration | Recommendation |
|---------------|----------------|
| **Default Value** | Treat `$4.99` as the default internal constant for `smtp_ip_addresses.approximate_cost` per active dedicated IP |
| **Configuration** | Allow this constant to be configuration-driven (e.g. application setting or migration-config table) so Finance & Operations can update it if Hostwinds pricing changes |
| **Overrides** | For bundled offers or special contracts, Finance may override approximate_cost at the record level |

### Usage Constraints

These IP cost values:
- Are internal modeling inputs for margin and utilization analysis
- Are **not** exposed as line-item customer billing

---

## Column Specification

| Attribute | Value |
|-----------|-------|
| **Column Name** | `approximate_cost` |
| **Data Type** | `DECIMAL(6,2)` |
| **Default** | `0.00` |
| **Constraint** | Non-negative (≥ 0) |
| **Index** | Partial index on active/warmed/warming IPs |

---

## Related Documentation

- [VPS Cost Tracking](./vps-cost-tracking) - Previous migration step
- [Business Intelligence Views](./business-intelligence-views) - Next migration step
- [Verification & Rollback](./verification-rollback) - Validation queries
