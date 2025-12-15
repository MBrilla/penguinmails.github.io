---
title: "OLTP Schema Overview"
description: "Complete OLTP database schema documentation for PenguinMails"
last_modified_date: "2025-12-15"
level: "2"
persona: "Engineering Teams"
keywords: "OLTP, database, schema, multi-tenant, NileDB, PostgreSQL"
---

# OLTP Schema Overview

The OLTP (Online Transaction Processing) Database serves as PenguinMails' primary operational database. It handles fast transactional operations, real-time data access, and core business logic execution with sub-500ms query performance targets.

## Architecture Characteristics

The OLTP layer optimizes for high-frequency operations with small record sizes. Tables are normalized for data integrity and indexed for speed. Row Level Security (RLS) provides complete tenant isolation at the database level.

### Performance Strategy

- **Denormalized Fields**: `tenant_id` on operational tables enables fast filtering
- **Covering Indexes**: Common query patterns have dedicated index coverage
- **Connection Pooling**: Aggressive pooling supports high-throughput operations
- **Date Partitioning**: Large operational tables may use date-based partitioning

### Table Naming Conventions

| Category | Convention | Examples |
|----------|-----------|----------|
| Core Entities | Plural nouns without prefix | `users`, `companies`, `campaigns` |
| Junction Tables | Singular compound names | `tenant_users`, `campaign_sequence_steps` |
| Configuration | Descriptive names | `user_preferences`, `tenant_config` |
| System Tables | Prefixed with table type | `system_config`, `feature_flags` |

## Schema Sections

### [Multi-Tenant Core](multi-tenant-core)

NileDB-managed authentication tables form the foundation of PenguinMails' multi-tenant architecture. These tables handle user identity, tenant organization, and role-based access control.

Tables: `users`, `tenants`, `tenant_users`

### [Business Logic](business-logic)

Core business entities power email marketing operations. Companies organize workspaces, domains configure email sending, leads store contacts, and templates hold email content.

Tables: `companies`, `domains`, `email_accounts`, `leads`, `templates`, `folders`, `tags`, `email_templates`, `template_dictionaries`

### [Campaign Management](campaigns)

Campaign orchestration tables define multi-step email sequences with conditional logic and scheduling. The execution engine processes these definitions to send emails at the right time.

Tables: `campaigns`, `campaign_sequence_steps`

### [Billing and Subscriptions](billing)

Payment architecture follows a Stripe-first approach. OLTP stores minimal references for access control while Stripe handles the full payment lifecycle.

Tables: `plans`, `subscriptions`, `subscription_addons`, `payments`

### [Infrastructure Management](infrastructure)

Infrastructure tables manage VPS instances, SMTP IP addresses, staff permissions, and system configuration. These support email delivery operations and platform administration.

Tables: `vps_instances`, `smtp_ip_addresses`, `domain_ip_assignments`, `staff_members`, `staff_roles`, `permissions`, `staff_role_permissions`, `system_config`, `feature_flags`, `user_preferences`, `tenant_config`, `tenant_policies`

### [Performance and Security](performance-security)

Index design and RLS policies optimize query performance while enforcing tenant isolation. External analytics integration patterns handle monitoring and observability.

Coverage: Critical indexes, RLS policies, external analytics integration, performance targets

## Four-Tier Database Architecture

PenguinMails uses a four-tier approach to separate concerns:

**OLTP (This Database)**: Fast transactional operations for real-time business logic including users, campaigns, leads, and configuration.

**Content Database**: Heavy email storage with retention policies and compression for message bodies and attachments.

**Analytics (OLAP)**: Aggregated metrics optimized for dashboard queries and business intelligence.

**Queue Layer**: Asynchronous job processing with Redis and PostgreSQL hybrid storage for reliable execution.

## Related Documentation

- [Database Infrastructure Overview](/docs/implementation-technical/database-infrastructure)
- [Connection Pooling Strategy](/docs/implementation-technical/database-infrastructure/architecture/connection-pooling-strategy)
- [Backup and Recovery](/docs/implementation-technical/database-infrastructure/operations/backup-recovery-procedures)
- [Architecture System](/docs/implementation-technical/architecture-system/architecture-overview)
