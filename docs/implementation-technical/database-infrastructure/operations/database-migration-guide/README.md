---
title: "Database Migration Guide"
description: "Comprehensive database migration guide for cost tracking and business intelligence views"
last_modified_date: "2025-12-15"
level: "2"
persona: "Documentation Users"
---

# Database Migration Guide

This guide provides comprehensive documentation for implementing cost tracking fields and business intelligence views to support executive-level business intelligence and decision making.

**Document Level:** Level 3 - Technical Implementation  
**Target Audience:** Database Administrators, Backend Engineers, Business Intelligence Engineers  
**Migration Priority:** Critical - Essential for executive cost attribution and reporting

## Topic Guides

| Topic | Description |
|-------|-------------|
| [Overview & Strategy](./overview-strategy) | Business context, objectives, and migration components |
| [VPS Cost Tracking](./vps-cost-tracking) | VPS instances cost tracking enhancement (Step 1) |
| [SMTP IP Cost Tracking](./smtp-ip-cost-tracking) | SMTP IP addresses cost tracking enhancement (Step 2) |
| [Business Intelligence Views](./business-intelligence-views) | Executive summary and cost allocation views (Step 3) |
| [Performance & Security](./performance-security) | Optimization indexes and access control (Steps 4-5) |
| [Verification & Rollback](./verification-rollback) | Data integrity validation and rollback procedures |

## Quick Reference

### Migration Components

1. **VPS Instances Cost Reference:** Add `vps_instances.approximate_cost` for modeled allocations
2. **SMTP IP Cost Reference:** Add `smtp_ip_addresses.approximate_cost` for IP-level cost modeling
3. **Business Intelligence Views:** Executive-level reporting views for heuristic analysis
4. **Performance Optimization:** Indexes for efficient cost-based queries
5. **Security Implementation:** Role-based access with multi-tenant isolation

### Business Justification

- **Cost Transparency (Internal):** Approximate infrastructure cost distribution using controlled heuristics
- **Profitability Analysis:** Support internal LTV, unit economics, and pricing strategy
- **Optimization Insights:** Identify anomalous usage patterns and cost inefficiencies
- **Executive Reporting:** Feed internal dashboards with modeled signals

### Key Constraints

- Values are modeled by Finance/Operations, not pulled from provider APIs in real-time
- Final financial truth remains in Finance systems and provider invoices
- Views are restricted to internal/admin roles and never exposed to tenants as authoritative billing
