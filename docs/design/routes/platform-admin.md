---
title: "Platform Administration"
description: "Internal tools for PenguinMails staff to manage the platform, support users, and monitor health"
last_modified_date: "2025-12-16"
level: "2"
persona: "Technical Teams"
---

# Platform Administration (Internal)

## 1. Purpose & Context (The "Why")

- **Goal**: Internal tools for PenguinMails staff to manage the platform, support users, and monitor health.

- **Feature References**:

  - [Internal Views](/docs/features/analytics/specs/internal-views)

  - [Operations Analytics](/docs/operations/analytics/overview)

- **User Journey**: Support ticket → Admin looks up user → Reviews audit trail to debug.

## 2. UI Patterns & Components (The "How")

- **Core Components**:

  - `AdminTable`: High-density table with raw data views.

  - `JSONViewer`: For viewing raw logs/payloads.

  - `StatusIndicator`: Traffic light system for service health.

- **Analytics Patterns**:

  - `ExecutiveDashboard`: See [Internal Views](/docs/features/analytics/specs/internal-views).

- **Layout**: Admin Context (Distinct visual theme).

## 3. Route Specifications

| Route | Access | Purpose | State/Data Requirements |
|---|---|---|---|
| `/dashboard/admin/plans` | Super Admin | Plan Management | List all subscription plans (active & inactive). Actions: Create, Edit, Toggle Status. |
| `/dashboard/admin/plans/new` | Super Admin | Create Plan | Form to define new plan limits and Stripe mapping. |
| `/dashboard/admin/plans/[id]` | Super Admin | Edit Plan | Update plan details, limits, or status. |
| `/dashboard/users` | Super Admin | Global Users | Searchable list of ALL users. Actions: Ban, Reset Password, **View Audit Trail**. |
| `/dashboard/tenants` | Super Admin | Global Tenants | List of organizations. Plan overrides, Feature flags management. |
| `/dashboard/finance` | Finance | Subscription Overview | Subscription status, Stripe sync health. |
| `/dashboard/system/queues` | Ops | Queue Health Monitor | Custom UI for hybrid PostgreSQL + Redis queue system. Job retry controls. |
| `/dashboard/system/logs` | Ops | System Logs | Log viewer (Elasticsearch/ClickHouse interface) (Future/2026 Spike). |
| `/admin/secrets` | Super Admin | Vault Secrets Management | Centralized secrets management for all tenants. Vault health monitoring, SSH keys, SMTP credentials, API keys, DKIM keys. Bulk rotation and audit log viewer. |

## 4. Detailed View Descriptions

### `/dashboard/admin/plans` - Plan Management

**User Story**: *\"As a product manager, I want to manage subscription plans and pricing without code changes, so I can launch new tiers for Black Friday.\"*

**What You'll Find**:

#### Plans Table

- **Columns**: Name, Slug, Price (Monthly/Yearly), Stripe Product ID, Status (Active/Inactive), Subscribers, Actions.

- **Status Indicators**:
  - **Green**: Active (Publicly purchasable)
  - **Grey**: Inactive (Hidden from new customers, but existing subscribers can renew)

- **Actions**:
  - **\"Create New Plan\" Button**: Navigates to plan creation form
  - **Row Actions**: Edit plan details, Toggle active status

**User Journey Context**: Product launches and pricing strategy management.

**Related Documentation**:

- [Plan Management Feature](/docs/features/admin/plan-management)
- [Billing Schema](/docs/implementation-technical/database-infrastructure/oltp-database/schema-guide)
- [Stripe Integration](/docs/features/payments/stripe-integration)

**Technical Integration**:

- **Stripe Product Sync**: Plans store `stripe_product_id` to generate checkout links
- **Feature Gating**: Plans table defines tenant limits (max emails, max users, etc.)
- **Cache Invalidation**: Updates trigger cache invalidation for public-facing pricing pages

---

### `/dashboard/admin/plans/[id]` - Edit Plan

**User Story**: *\"As an admin, I want to update feature limits for the Pro plan or deactivate a seasonal plan.\"*

**What You'll Find**:

#### Plan Details Form

- **General Info**:
  - Name (e.g., "Pro Plan")
  - Slug (e.g., "pro", used in URLs)
  - Description (marketing copy)

- **Stripe Mapping**:
  - `stripe_product_id` (Required): Links to Stripe Product
  - **Note**: Prices managed in Stripe, but displayed here for reference (`price_monthly`, `price_yearly`)

- **Feature Limits**:
  - Max Users per tenant
  - Max Domains per tenant
  - Max Campaigns/Month
  - Max Emails/Month
  - Storage Limit (GB)

- **Feature Flags**:
  - API Access enabled
  - Priority Support
  - White Label branding
  - Advanced Analytics

- **Lifecycle Control**:
  - **Is Active Toggle**: Controls visibility in purchase UI (inactive plans hidden from new customers)

**Validation Rules**:

- `slug` must be unique across all plans
- `stripe_product_id` format validation
- Feature limits must be positive integers

**User Journey Context**: Plan configuration and lifecycle management.

**Related Documentation**:

- [Subscription Management](/docs/features/payments/subscription-management)
- [Feature Gating Implementation](/docs/implementation-technical/feature-gating)

**Technical Integration**:

- **Database**: `plans` table in OLTP database
- **Cache Strategy**: Heavily cached for public pages, admin updates invalidate cache tags
- **Webhooks**: Stripe product updates can sync back to PenguinMails

---

### `/dashboard/users` - Global User Management

**User Story**: *"As a support admin, I want to quickly look up any user, view their account details, and review their audit trail to debug issues, so I can resolve tickets faster."*

**What You'll Find**:

#### Search & Filter

- **Global Search Bar** (Prominent):

  - Placeholder: "Search by email, name, or user ID".

  - **Autocomplete**: Shows results as you type.

- **Filters**:

  - Role: All, Owner, Admin, Member.

  - Status: Active, Suspended, Deleted.

  - Plan: Free, Pro, Enterprise.

#### Users Table

- **Columns**: UserID, Name, Email, Plan, Tenant, Status, Created Date, Last Login, Actions.

- **Color Coding**:

  - Suspended users: Red background.

  - Churned users (plan expired): Orange.

- **Row Actions** (Gear icon):

  - **View Details**: Opens modal with full user profile, activity log.

  - **View Audit Trail**: Opens comprehensive audit viewer.

    - **Audit Viewer Modal**: Shows complete user action history with filters.

    - **Searchable Timeline**: Logins, campaign actions, settings changes, API calls.

    - **Export Capability**: Download audit trail as CSV for detailed analysis.

  - **Reset Password**: Sends reset email.

  - **Suspend Account**: Confirmation modal, reason required.
  
  - **Revoke All Sessions**: Immediate token invalidation (Security).

  - **Change Role**: Promote/demote user (e.g., Member ↔ Admin).

  - **Delete Account** (GDPR): Hard delete, irreversible.

#### Audit Trail Viewer

- **Modal or Slide-over** when clicking "View Audit Trail":

  - **Activity Timeline**: Logins, Campaign Creates, Settings Changes, API Calls.

  - **Support Access Log**: Admin views and support ticket references.

  - **Filters**: Date range, action type, IP address.

  - **Search**: Full-text search across all audit entries.

  - **Export**: Download filtered results as CSV.

> [!NOTE]
> **Database Schema**: References audit log table from database documentation (verification needed - see Phase 4).
> **API Endpoint**: Requires executive/platform API endpoint for audit trail retrieval (verification needed - see Phase 4).

**User Journey Context**: Reactive (support-driven). Must be fast and auditable.

**Related Documentation**:

- [Support Processes](/docs/customer-success/support-playbooks/account-lookup)

- [GDPR Compliance](/docs/compliance-security/data-privacy/gdpr)

**Technical Integration**:

- **Audit Trail**: Comprehensive logging of all user actions with retention policy.

- **Search**: Elasticsearch for fast full-text search across millions of users and audit entries.

- **Database Reference**: Audit log table schema (to be verified in Phase 4).

- **API Reference**: Executive/Platform API audit trail endpoint (to be verified in Phase 4).

---

### `/dashboard/tenants` - Tenant (Organization) Management

**User Story**: *"As a platform admin, I want to manage tenant accounts, override plans, and enable beta features, so I can support sales and customer success."*

**What You'll Find**:

#### Tenants Table

- **Columns**: Tenant ID, Company Name, Owner Email, Plan, MRR, Workspaces, Users, Status, Created, Actions.

- **MRR Column**: Monthly Recurring Revenue (sortable).

- **Filters**:

  - Plan: All, Free, Pro, Enterprise.

  - Status: Active, Trial, Churned.

#### Row Actions

- **View Details**: Opens tenant's workspace overview and audit trail in read-only mode.

- **Edit Plan**:

  - **Modal**: Override plan (e.g., upgrade to Enterprise, add custom limits).

  - **"Apply Custom Pricing" Checkbox**: For negotiated deals.

- **Feature Flags**:

  - **Modal**: List of beta features with toggles.

  - Example: "Enable Advanced Analytics", "Allow 100k Emails/month".

- **Billing Override**:

  - Comps (free service), Extended trial, Custom billing cycle.

#### Quick Actions (Top Bar)

- **"Create Test Tenant" Button**: Provisions sandbox account for demos.

**User Journey Context**: Account management and  sales support.

**Related Documentation**:

- [Feature Flags](/docs/technical/feature-management/flags)

- [Sales SLA](/docs/business/sales/contract-management)

**Technical Integration**:

- **Feature Flags**: LaunchDarkly or internal flags table.

- **Plan Overrides**: Stored in `tenant_overrides` table, checked before billing.

---

### `/dashboard/finance` - Subscription Overview

**User Story**: *"As a finance admin, I want to monitor subscription status and access Stripe Dashboard for revenue analytics."*

**What You'll Find**:

#### Subscription Summary (see [Analytics UI Patterns](/docs/design/analytics-ui-patterns#data-cards))

- **Basic Metrics**:

  - **Active Subscriptions**: 234.

  - **Plan Distribution**: Free (45), Pro (150), Enterprise (39).

  - **Stripe Dashboard Link**: "View MRR and Revenue Analytics in Stripe" button.

- **Quick Links**:

  - **Open Stripe Dashboard**: Opens Stripe revenue section in new tab.

  - **View All Subscriptions in Stripe**: Direct link to Stripe subscriptions list.

#### Stripe Webhook Status

- **Last Webhook**: "5 minutes ago".

- **Webhook Health**:

  - Healthy / Delayed / Error indicator.

  - Last event type: `invoice.paid`, `subscription.updated`, etc.

  - Failed payments count: 3 subscriptions with payment issues.

- **Stripe Links**:

  - "View Payment Details in Stripe" button.

  - "View Failed Payments in Stripe" link.

#### Stripe Dashboard Access

- **Quick Actions**:

  - **"Open Stripe Dashboard"**: Opens Stripe in new tab.

  - **"Force Sync Transaction"**: Input Invoice ID to pull latest status from Stripe.

  - **"View MRR Reports"**: Direct link to Stripe revenue section.

  - **"Download Invoices"**: Link to Stripe invoice list.

  - **"Export Revenue Data"**: Link to Stripe data export tools.

> [!NOTE]
> All detailed financial analytics (MRR trends, revenue forecasting, invoice management, payment history) are handled in Stripe Dashboard.

**User Journey Context**: Monthly financial overview and Stripe integration monitoring.

**Related Documentation**:

- [Finance Processes](/docs/business/finance/reporting/mrr-calculation)

- [Stripe Webhooks](/docs/technical/integrations/stripe)

**Technical Integration**:

- **Stripe API**: Read-only access for subscription list and webhook status

- **Subscription Count**: Simple query from `subscriptions` table (`status = 'active'`)

- **No Complex Calculations**: MRR, revenue trends, and analytics handled by Stripe

---

### `/dashboard/system/queues` - Background Job Monitoring

**User Story**: *"As an ops engineer, I want to monitor the hybrid PostgreSQL + Redis job queues for failures and delays, so I can ensure system reliability."*

**What You'll Find**:

#### Queue Dashboard

##### Custom UI for Hybrid Queue System

- **Queue List** (PostgreSQL + Redis monitoring):

  - Columns: Queue Name, Active Jobs, Waiting, Completed (last hour), Failed, Actions.

  - **Queue Names**: `queue:email-sending:high`, `queue:email-sending`, `queue:email-sending:low`, `queue:analytics:daily-aggregate`, `queue:warmup:process`, etc.

- **Per-Queue Actions**:

  - **View Details**: Shows job list.

  - **Pause Queue**: Emergency stop.

  - **Resume Queue**.

#### Job Details (Click on failed job)

- **JSON Viewer**:

  - Shows job data (e.g., campaign ID, recipient list).

  - Error stack trace.

- **Actions**:

  - **Retry Job**: Re-queues the job.

  - **Delete Job**: Removes from queue.

**User Journey Context**: Reactive troubleshooting. Accessed during incidents.

**Related Documentation**:

- [Queue System Implementation](/docs/technical/architecture/detailed-technical/queue-system-implementation)

- [Incident Response](/docs/operations/incident-management/runbooks)

**Technical Integration**:

- **Hybrid Architecture**: PostgreSQL for durable state + Redis for fast processing.

- **Monitoring**: Real-time queue depth tracking and job status monitoring.

- **Database Tables**: `jobs`, `job_queues`, `job_logs` (reference to be verified in Phase 4).

- **API Endpoint**: Executive/Platform API queue monitoring endpoint (to be verified in Phase 4).

- **Alerting**: PagerDuty integration for critical failures.

---

### `/dashboard/system/infrastructure` - System Health Monitoring

**User Story**: *"As an ops engineer, I want a real-time view of server health and IP reputation, so I can detect and resolve issues before they impact users."*

**What You'll Find**:

#### Service Health Grid (see [Analytics UI Patterns](/docs/design/analytics-ui-patterns))

- **Status Cards** (Traffic light):

  - **API Server**: Healthy (Response time: 120ms).

  - **SMTP Service**: Degraded (Queue backlog: 5,000 emails).

  - **Database (OLTP)**: Healthy.

  - **Database (OLAP)**: Healthy.

#### IP Reputation Monitor

- **Table**: IP Address, Provider, Reputation Score, Blacklists, Daily Volume, Status.

- **Alerts**: "IP 203.0.113.12 listed on Spamhaus (detected 2 hours ago)".

#### Infrastructure Alerts

- **Alert Feed** (Last 24 hours):

  - Timestamp, Severity, Service, Message.

  - Example: "ERROR | SMTP | Queue size exceeded 10k threshold".

**User Journey Context**: Proactive monitoring. Reviewed multiple times daily.

**Related Documentation**:

- [Infrastructure Overview](/docs/technical/infrastructure/server-architecture)

- [Monitoring Setup](/docs/operations/monitoring/prometheus-grafana)

**Technical Integration**:

- **Prometheus + Grafana**: Metrics collection and visualization (Future/2026 Spike).
- **ClickHouse**: Stores historical metrics (Future/2026 Spike).

- **PagerDuty**: Alerts for critical issues.

---

### `/dashboard/system/logs` - System Log Viewer

**User Story**: *"As a developer debugging a production issue, I want to search and filter application logs, so I can trace errors to their source."*

**What You'll Find**:

#### Log Search Interface

- **Search Bar**:

  - Placeholder: "Search logs (e.g., errors, user email, request ID)".

  - Supports regex.

- **Filters** (Sidebar):

  - **Level**: Error, Warning, Info, Debug.

  - **Service**: API, SMTP, Queue, Web.

  - **Time Range**: Last hour, Last 24 hours, Custom.

#### Log Results

- **Table or List View**:

  - Columns: Timestamp, Level, Service, Message, User/Request ID.

  - **Color Coding**: Errors (Red), Warnings (Yellow).

- **Expandable Rows**:

  - Click to see full log entry with metadata (stack trace, context).

#### Export

- **"Export to CSV" Button**: For further analysis.

**User Journey Context**: Debugging production issues. High-stress, time-sensitive.

**Related Documentation**:

- [Logging Standards](/docs/technical/observability/logging)

- [Error Tracking](/docs/operations/monitoring/sentry-integration)

**Technical Integration**:

- **Elasticsearch**: Centralized log aggregation (Future/2026 Spike).
- **Sentry**: Error tracking and alerting (linked from error logs) (Future/2026 Spike).

---

### `/admin/secrets` - Vault Secrets Management

**User Story**: *"As a platform admin, I want to monitor Vault health and manage tenant secrets centrally, so I can ensure secure credential storage and respond quickly to security incidents."*

**What You'll Find**:

#### Vault Health Dashboard

- **Status Cards** (Traffic light):

  - **Vault Seal Status**: Unsealed (Green) / Sealed (Red).

  - **Active Node**: vault-node-1 (Primary).

  - **Storage Backend**: PostgreSQL - Healthy.

  - **Last Backup**: 2 hours ago (Green) / 25 hours ago (Yellow warning).

  - **Backup Health**: Healthy / Warning / Critical.

- **Quick Actions**:

  - **"Force Backup Now" Button**: Manually trigger Vault snapshot.

  - **"View Backup History" Link**: Shows last 30 backups with restore points.

#### Tenant Secrets Overview

- **Search Bar**:

  - Placeholder: "Search by tenant ID, tenant name, or VPS IP".

  - Autocomplete with tenant suggestions.

- **Filters**:

  - **Secret Type**: All, SSH Keys, SMTP Credentials, API Keys, DKIM Keys.

  - **Status**: Active, Expired, Revoked.

  - **Rotation Due**: Next 7 days, Next 30 days, Overdue.

- **Tenants Table**:

  - **Columns**: Tenant ID, Tenant Name, VPS IP, SSH Key Status, SMTP Status, API Keys Count, DKIM Keys Count, Last Rotation, Actions.

  - **Color Coding**:

    - Expired secrets: Red background.

    - Rotation due soon: Yellow background.

    - Healthy: Green indicator.

#### Secret Details View (Click tenant row to expand)

- **SSH Keys Section**:

  - **Admin SSH Key**: Status (Active/Expired), Last Rotated, Next Rotation, Key Fingerprint.

  - **Tenant SSH Key**: Status, Last Rotated, Next Rotation, Key Fingerprint, Last Downloaded.

  - **Actions**: "Rotate Now", "View Rotation History".

- **SMTP Credentials Section**:

  - **Admin Username**: Display masked username (e.g., `admin@*****.com`).

  - **Webmail URL**: Display full URL (e.g., `https://mail.example.com`).

  - **Last Rotated**: Timestamp (e.g., "2025-07-15 10:00:00 UTC").

  - **Next Rotation**: 180 days from last rotation (e.g., "2026-01-15 10:00:00 UTC").

  - **Rotation Status**: Color-coded indicator (Green: Healthy, Yellow: Due soon, Red: Overdue).

  - **Actions**:

    - **"View Credentials" Button**: Requires re-authentication (password + 2FA).

      - Opens secure modal with time-limited access (15 minutes).

      - Password initially masked (click eye icon to reveal).

      - Copy to clipboard button with confirmation.

      - Countdown timer showing expiration.

      - Auto-hide credentials after 15 minutes.

    - **"Rotate Now" Button**: Manual rotation with reason input.

    - **"Emergency Reset" Button**: For security incidents (requires incident ID).

    - **"View Audit Trail" Link**: Shows all credential access events.

- **API Keys Section**:

  - **Table**: Key ID, Permissions, Rate Limit, Created, Last Used, Request Count, Status.

  - **Actions**: "Revoke Key", "View Usage Analytics".

- **DKIM Keys Section**:

  - **Table**: Domain, Selector, Last Rotated, Next Rotation (365 days), Status.

  - **Actions**: "Rotate Now", "View DNS Records".

#### Rotation Management

- **Bulk Rotation Panel**:

  - **"Rotate All SSH Keys" Button**: Triggers rotation for all tenants (confirmation required).

  - **"Rotate All SMTP Credentials" Button**: Bulk SMTP rotation.

  - **"Rotate Overdue Secrets" Button**: Rotates only secrets past rotation date.

- **Rotation Schedule Configuration**:

  - **SSH Keys**: 90 days (default), Custom (30-365 days).

  - **SMTP Credentials**: 180 days (default), Custom.

  - **DKIM Keys**: 365 days (default), Custom.

- **Grace Period Settings**:

  - **SSH Keys**: 24 hours (both old and new keys valid).

  - **DKIM Keys**: 48 hours (DNS propagation time).

#### SMTP Credentials Viewing Modal

**Triggered by**: Clicking "View Credentials" button in SMTP Credentials Section.

**Security Requirements**:

- **Re-authentication Required**: Admin must enter password + 2FA code.

- **Time-Limited Access**: Credentials visible for 15 minutes only.

- **Audit Logging**: All credential views logged with timestamp, user, and IP address.

**Modal Components**:

1. **Re-authentication Step**:

   - **Password Input**: "Enter your password to continue".

   - **2FA Code Input**: "Enter your 2FA code".

   - **"Verify" Button**: Validates credentials.

   - **Error Handling**: "Invalid credentials. Please try again."

2. **Credentials Display** (After successful re-authentication):

   - **Header**: "SMTP Admin Credentials - Tenant: {tenant_name}".

   - **Expiration Timer**: "Credentials expire in 14:32" (countdown).

   - **Warning Banner**: "⚠️ These credentials provide full access to the tenant's email infrastructure. Handle with care."

   - **Credential Fields**:

     - **Username**: `admin@example.com` (plain text, copyable).

     - **Password**: `••••••••••••••••` (masked by default).

       - **Eye Icon Button**: Click to reveal password.

       - **Copy Button**: Copy password to clipboard (shows "Copied!" confirmation).

     - **Webmail URL**: `https://mail.example.com` (clickable link, opens in new tab).

   - **Metadata**:

     - **Created**: "2025-01-15 10:00:00 UTC".

     - **Last Rotated**: "2025-07-15 10:00:00 UTC".

     - **Next Rotation**: "2026-01-15 10:00:00 UTC".

   - **Actions**:

     - **"Close" Button**: Closes modal immediately.

     - **"Rotate Now" Button**: Triggers manual rotation (confirmation required).

     - **"Emergency Reset" Button**: For security incidents (requires incident ID).

3. **Auto-Hide Behavior**:

   - After 15 minutes, modal automatically closes.

   - Display warning at 1 minute remaining: "Credentials will expire in 1 minute".

   - After expiration, show message: "Credentials expired. Please re-authenticate to view again."

**Implementation Notes**:

- **API Endpoint**: `GET /api/v1/platform-admin/tenants/{tenant_id}/smtp-credentials`.

- **Re-auth Token**: Generated on successful re-authentication, valid for 15 minutes.

- **Audit Event**: `smtp_credentials_viewed` logged with tenant_id, admin_user_id, timestamp, ip_address.

#### Audit Log Viewer

- **Real-time Audit Stream**:

  - **Columns**: Timestamp, Action, Tenant, User, Secret Type, IP Address, Status.

  - **Actions**: read, write, delete, rotate, download.

  - **Auto-refresh**: Every 5 seconds.

- **Filters**:

  - **Tenant**: Select from dropdown.

  - **Secret Type**: SSH, SMTP, API Keys, DKIM.

  - **Action**: Read, Write, Delete, Rotate, Download.

  - **Date Range**: Last hour, Last 24 hours, Last 7 days, Custom.

  - **User**: Filter by admin user who performed action.

- **Suspicious Activity Alerts**:

  - **Alert Banner**: "⚠️ 5 failed authentication attempts detected in last 5 minutes".

  - **Alert Types**:

    - Multiple failed authentication attempts (>5 in 5 minutes).

    - Access to secrets outside normal hours (2am-6am).

    - Bulk secret reads (>100 secrets in 1 minute).

    - Secret deletion without prior approval.

- **Export Audit Logs**:

  - **"Export to CSV" Button**: Download filtered audit logs.

  - **"Export to JSON" Button**: For programmatic analysis.

**User Journey Context**: Proactive security monitoring and reactive incident response.

**Related Documentation**:

- [Vault Integration Architecture](/docs/design/routes/infrastructure-ssh-access) - Detailed route specification

- [Vault SSH Management Feature](/docs/features/infrastructure/vault-ssh-management) - Feature documentation

- [Security Compliance](/docs/compliance-security/enterprise/infrastructure-security)

- [Incident Response](/docs/operations/incident-management/runbooks)

**Technical Integration**:

- **HashiCorp Vault**: Centralized secrets storage with KV v2 engine.

- **Vault API**: All secret operations via Vault REST API.

- **Access Control**: Role-based policies (admin full access, tenant read-only).

- **Audit Logging**: All Vault operations logged to audit device (file + syslog).

- **Backup System**: Automated daily snapshots to S3 with AES-256-GCM encryption.

- **Monitoring**: Prometheus metrics for Vault health, PagerDuty alerts for critical issues.

- **Database Reference**: Vault uses PostgreSQL storage backend for HA.

- **API Endpoints**:

  - `GET /api/v1/admin/vault/health` - Vault health status.

  - `GET /api/v1/admin/secrets/tenants` - List all tenant secrets.

  - `GET /api/v1/admin/secrets/tenant/{tenant_id}` - Tenant secret details.

  - `POST /api/v1/admin/secrets/rotate-all` - Bulk rotation.

  - `GET /api/v1/admin/vault/audit-logs` - Retrieve audit logs.

---

### Admin-Specific UI Theme (Future / Post-MVP)

**User Story**: *"As an admin, I want a visually distinct interface, so I always know when I'm in admin mode and avoid accidental changes."*

**Visual Differences**:

- **Color Scheme**: Dark orange accent (vs. blue for users).

- **Sidebar Header**: "Admin Panel" label.

- **Context Indicator**: Banner showing current admin context (e.g., "Viewing as Admin").

**Related Documentation**:

- [Design Tokens](/docs/design/design-tokens)

## 5. Related API Endpoints

| Route | Related API | Description |
|---|---|---|
| `/dashboard/admin/plans` | [Plans API](/docs/implementation-technical/api/platform-api/plans) | `GET /api/v1/platform/admin/plans` (List), `POST` (Create), `PUT /{id}` (Update). |
| `/users` | [Admin API](/docs/implementation-technical/api/platform-api/admin) | `GET /api/v1/platform/admin/users` (List), `GET .../audit` (Audit Trail). |
| `/tenants` | [Admin API](/docs/implementation-technical/api/platform-api/admin) | `GET /api/v1/platform/admin/tenants` (List), `PATCH` (Overrides). |
| `/finance` | [Subscriptions API](/docs/implementation-technical/api/platform-api/subscriptions) | `GET /api/v1/platform/subscriptions/mrr` (Revenue stats). |
| `/system/queues` | [Queue API](/docs/implementation-technical/api/queue/jobs) | `GET /api/v1/queue/stats` (Monitor), `POST .../retry` (Actions). |
| `/admin/secrets` | Vault Admin API | `GET /api/v1/admin/vault/health` (Health), `GET /api/v1/admin/secrets/tenants` (List), `POST /api/v1/admin/secrets/rotate-all` (Bulk rotation), `GET /api/v1/admin/vault/audit-logs` (Audit). |

## 6. Data Strategy

- **Fetching Method**:

  - **Tables (Users/Tenants)**: Server Components with searchParams.

  - **Real-time Dashboards (Queues/Logs)**: Client Components with polling (every 5s) or WebSocket.

- **Caching**:

  - **Plan Data**: Heavily cached for public-facing pages. Admin updates invalidate cache tags (`plans-list`, `plan-{slug}`).

  - **Aggregated Metrics (MRR)**: Cached for 1 hour.

  - **Logs**: No caching (live stream).

- **Pagination**:

  - **Offset-based** for admin tables (need "Jump to page X").

- **Access Control**:

  - **Strict RBAC**:
    - Access guarded by `staff_members` table (distinct from tenant users).
    - Checks on every data fetch (Super Admin vs Support vs Finance).

## 7. Edge Cases & Error Handling

- **Tenant Not Found**: Accessing `/tenants/[id]` with invalid ID → 404 Page.

- **Insufficient Permissions**: Support role accessing Finance dashboard → 403 Forbidden.

- **Queue Action Failures**: If "Retry All Failed Jobs" partially fails, show "Retried 45/50. 5 jobs still stuck."

- **Log Stream Overflow**: If logs exceed buffer, show "Logs truncated. Download full logs?" with export link.

## 8. Component Architecture

### Page Components

- **`AdminDataTable`** (Server/Client)

  - Generic table with sorting, filtering, and bulk actions.

  - Props: `data: T[]`, `columns: ColumnDef<T>[]`.

- **`QueueMonitor`** (Client)

  - Features: Real-time polling of job counts, "Retry" actions.

  - Sub-components: `QueueCard`, `JobList`.

- **`LogViewer`** (Client)

  - Features: Virtualized scroll, regex search, "Follow tail" mode.

### Shared Components

- **`JSONViewer`**: For inspecting job payloads and audit logs.

- **`StatusDot`**: Real-time health indicator (Green/Red/Blinking).
