---
title: "Platform Administration Overview"
description: "Overview of internal admin tools for PenguinMails platform management"
last_modified_date: "2025-12-16"
level: "2"
persona: "Technical Teams"
---

# Platform Administration (Internal)

## 1. Purpose & Context

- **Goal**: Internal tools for PenguinMails staff to manage the platform, support users, and monitor health.
- **Feature References**:
  - [Internal Views](/docs/features/analytics/specs/internal-views)
  - [Operations Analytics](/docs/operations/analytics/overview)
- **User Journey**: Support ticket → Admin looks up user → Reviews audit trail to debug.

## 2. UI Patterns & Components

- **Core Components**:
  - `AdminTable`: High-density table with raw data views.
  - `JSONViewer`: For viewing raw logs/payloads.
  - `StatusIndicator`: Traffic light system for service health.
- **Analytics Patterns**:
  - `ExecutiveDashboard`: See [Internal Views](/docs/features/analytics/specs/internal-views).
- **Layout**: Admin Context (Distinct visual theme).

## 3. Route Specifications

| Route | Access | Purpose |
|---|---|---|
| `/dashboard/admin/plans` | Super Admin | Plan Management |
| `/dashboard/admin/plans/new` | Super Admin | Create Plan |
| `/dashboard/admin/plans/[id]` | Super Admin | Edit Plan |
| `/dashboard/users` | Super Admin | Global Users |
| `/dashboard/tenants` | Super Admin | Global Tenants |
| `/dashboard/finance` | Finance | Subscription Overview |
| `/dashboard/system/queues` | Ops | Queue Health Monitor |
| `/dashboard/system/logs` | Ops | System Logs |
| `/admin/secrets` | Super Admin | Vault Secrets Management |

## Related Documentation

- [Plan Management Routes](./platform-admin/plan-management) - Plan creation and editing
- [User Management Routes](./platform-admin/user-management) - Global user administration
- [Tenant Management Routes](./platform-admin/tenant-management) - Organization management
- [Finance Routes](./platform-admin/finance) - Subscription and billing overview
- [System Routes](./platform-admin/system-monitoring) - Queues, logs, infrastructure
- [Secrets Management Routes](./platform-admin/secrets-management) - Vault and credentials
