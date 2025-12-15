---
title: "Platform Administration"
description: "Internal operational tools for PenguinMails staff"
last_modified_date: "2025-12-16"
level: "2"
persona: "Internal Staff"
---

# Platform Administration (Internal)

**Target Users**: PenguinMails internal staff (super_admin, admin, support, qa)  
**Customer-Facing**: No - These are internal operational tools

## Overview

The PenguinMails admin panel provides operational tools for internal staff to manage the platform, monitor system health, support customers, and make data-driven executive decisions.

**Key Insight**: These features progressively expose more information for executive decision-making. Currently, PenguinMails executives use third-party tools (spreadsheets, external dashboards) for comprehensive reporting. Future enhancements will bring this data into the admin panel to answer questions like "how much money did we make last year?"

## Current Capabilities (MVP - Q1 2026)

- **[Plan Management](/docs/features/admin/plan-management/)** - Configure subscription plans and pricing
- **[User Management](/docs/features/admin/user-management/)** - Global user lookup, audit trails, and account actions
- **[Tenant Management](/docs/features/admin/tenant-management/)** - Manage customer organizations, plan overrides, and feature flags
- **[Finance](/docs/features/admin/finance/)** - Subscription monitoring and Stripe Dashboard access
- **[System Monitoring](/docs/features/admin/system-monitoring/)** - Infrastructure health, queue monitoring, and log viewer
  - [Queue Monitoring](/docs/features/admin/system-monitoring/queue-monitoring) - Background job management
  - [Infrastructure Monitoring](/docs/features/admin/system-monitoring/infrastructure-monitoring) - Service health and IP reputation
  - [Log Viewer](/docs/features/admin/system-monitoring/log-viewer) - Centralized log search
- **[Vault Management](/docs/features/admin/vault-management/)** - Secure secrets management (SSH keys, SMTP credentials, API keys, DKIM keys)

## Future Enhancements (Roadmap)

These features will progressively expose more information for PenguinMails staff to make better decisions:

### Q1 2026 (Critical MVP Features)

**Platform Infrastructure Monitoring Dashboard** (10-15 days)

- Real-time SMTP health monitoring
- Server health tracking
- Queue monitoring and alerts
- Infrastructure health alerts

**Payment History & Financial Management** (10-15 days)

- Complete payment transaction history
- Invoice management
- Failed payment tracking
- Refund management
- Subscription analytics
- Revenue reporting (MRR, ARR, churn analysis)

**Admin Platform Features** (7-10 days)

- User audit trail
- Support access tracking
- Tenant plan management
- Feature flags management
- Stripe webhooks handling

### Q2 2026

## Enhanced Financial Reporting

- Comprehensive revenue analytics
- Answer executive questions like "how much money did we make last year?"
- CFO-ready financial dashboards
- Revenue forecasting

## User Analytics

- Tenant user behavior tracking
- Feature adoption metrics
- Usage patterns and trends

## Operational Dashboards

- Real-time platform metrics
- Performance KPIs
- System capacity planning

### Q3 2026

## Compliance Automation

- Automated compliance reporting (SOC 2, GDPR, HIPAA)
- Control effectiveness tracking
- Evidence collection for audits

### Q4 2026 - Q1 2027 (Enterprise Features)

See **[Enterprise Features Roadmap](/docs/features/admin/enterprise-features/overview)** for advanced admin capabilities:

- Advanced audit logs and compliance reporting
- Real-time monitoring and SLA tracking
- Team collaboration features for internal staff
- Dedicated support tools and SLA tiers

## Role-Based Access

| Role | Access Level | Capabilities |
|------|--------------|--------------|
| **super_admin** | Platform-wide, all tenants | Full access to all admin features, system configuration |
| **admin** | Platform-wide, all tenants | Manage tenants, plans, limited system access |
| **support** | Read-only, no PII | View tenant info, help troubleshoot, no sensitive data access |
| **qa** | Sandbox only | Test features without affecting production data |

## Related Documentation

### Route Specifications

- [Platform Admin Routes](/docs/design/routes/platform-admin) - Complete admin panel UI specifications
- **Specific Route Sections**:
  - [Plan Management](/docs/design/routes/platform-admin#dashboard-admin-plans) - Plan configuration UI
  - [User Management](/docs/design/routes/platform-admin#dashboard-users) - Global user lookup and audit trails
  - [Tenant Management](/docs/design/routes/platform-admin#dashboard-tenants) - Organization management UI
  - [Finance Dashboard](/docs/design/routes/platform-admin#dashboard-finance) - Subscription overview UI
  - [Queue Monitoring](/docs/design/routes/platform-admin#dashboard-system-queues) - Background job monitoring
  - [Infrastructure Health](/docs/design/routes/platform-admin#dashboard-system-infrastructure) - Service health monitoring
  - [Log Viewer](/docs/design/routes/platform-admin#dashboard-system-logs) - System log access
  - [Vault Secrets](/docs/design/routes/platform-admin#admin-secrets) - Secrets management UI

### API Documentation

- **Platform Admin Endpoints**:
  - [Admin Operations API](/docs/implementation-technical/api/platform-api/admin) - User and tenant management
  - [Plans API](/docs/implementation-technical/api/platform-api/plans) - Subscription plan configuration
  - [Finance API](/docs/implementation-technical/api/platform-api/finance) - Subscription counts and Stripe sync status
  - [System Monitoring API](/docs/implementation-technical/api/platform-api/system-monitoring) - Queue, infrastructure, and logs
  - [Vault Management API](/docs/implementation-technical/api/platform-api/vault) - Secrets management and rotation
- **Related APIs**:
  - [Subscriptions API](/docs/implementation-technical/api/platform-api/subscriptions) - Billing and subscription data
  - [Analytics API](/docs/implementation-technical/api/platform-api/analytics) - System analytics

### Operations

- [Support Role Guide](/docs/operations/support/role-guide) - Support and QA role documentation
- [Internal Analytics](/docs/features/analytics/internal-analytics/) - Operational reporting for PenguinMails staff

### Comparison: Admin vs. Tenant Settings

**Admin Panel** (`/dashboard/admin/*`):

- **Who**: PenguinMails internal staff
- **Scope**: Platform-wide, all tenants
- **Purpose**: Operate and monitor the platform

**Tenant Settings** (`/dashboard/settings/*`):

- **Who**: Tenant administrators (owner, manager)
- **Scope**: Single tenant organization
- **Purpose**: Configure tenant account and billing

---

[← Back to All Features]()
