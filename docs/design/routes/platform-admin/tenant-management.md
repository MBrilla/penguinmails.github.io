---
title: "Tenant Management Routes"
description: "Admin routes for organization and tenant administration"
last_modified_date: "2025-12-16"
level: "2"
persona: "Technical Teams"
---

# Tenant Management Routes

## `/dashboard/tenants` - Tenant (Organization) Management

**User Story**: *"As a platform admin, I want to manage tenant accounts, override plans, and enable beta features, so I can support sales and customer success."*

### Tenants Table

- **Columns**: Tenant ID, Company Name, Owner Email, Plan, MRR, Workspaces, Users, Status, Created, Actions.
- **MRR Column**: Monthly Recurring Revenue (sortable).
- **Filters**:
  - Plan: All, Free, Pro, Enterprise.
  - Status: Active, Trial, Churned.

### Row Actions

- **View Details**: Opens tenant's workspace overview and audit trail in read-only mode.
- **Edit Plan**:
  - **Modal**: Override plan (e.g., upgrade to Enterprise, add custom limits).
  - **"Apply Custom Pricing" Checkbox**: For negotiated deals.
- **Feature Flags**:
  - **Modal**: List of beta features with toggles.
  - Example: "Enable Advanced Analytics", "Allow 100k Emails/month".
- **Billing Override**:
  - Comps (free service), Extended trial, Custom billing cycle.

### Quick Actions (Top Bar)

- **"Create Test Tenant" Button**: Provisions sandbox account for demos.

**User Journey Context**: Account management and sales support.

**Related Documentation**:
- [Feature Flags](/docs/technical/feature-management/flags)
- [Sales SLA](/docs/business/sales/contract-management)

**Technical Integration**:
- **Feature Flags**: LaunchDarkly or internal flags table.
- **Plan Overrides**: Stored in `tenant_overrides` table, checked before billing.
