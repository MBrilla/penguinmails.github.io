---
title: "Agency Client Workspaces"
description: "Create and configure client workspaces for agencies managing multiple clients"
last_modified_date: "2025-12-16"
level: "2"
persona: "Technical Teams"
---

# Agency Client Workspaces

## 1. Purpose & Context (The "Why")

- **Goal**: Create and configure client workspaces. Crucial for Agencies managing multiple clients.

- **Feature References**:

  - [Agency Features](/docs/business/user-personas/agency-owners)

- **User Journey**: Agency signs up → Creates Workspace for "Client A" → Invites Client A users.

## 2. UI Patterns & Components (The "How")

- **Core Components**:

  - `CardGrid`: Visual list of workspaces with health status.

  - `Modal`: For "Create Workspace" flow.

  - `Tabs`: For Workspace Settings (General, Members, Billing).

- **Analytics Patterns**:

  - `DataCard`: "Client Health" summary on the card (e.g., " 3 Active Campaigns").

- **Layout**: Global Context.

## 3. Route Specifications

| Route | Access | Purpose | State/Data Requirements |
|---|---|---|---|
| `/dashboard/workspaces` | Tenant/Agency | Workspace List | **Server Component**. List of all accessible workspaces. Agency view shows aggregated stats (Total Revenue, Active Clients). |
| `/dashboard/workspaces/new` | Tenant/Agency | Create Workspace | **Client Component**. Form: Name, Logo, Plan selection (if separate billing). |
| `/dashboard/workspaces/[slug]/settings` | Admin | Workspace Config | Settings for the specific workspace. Rename, Delete, Transfer Ownership. |

## 4. Detailed View Descriptions

### `/dashboard/workspaces` - Workspace Command Center

**User Story**: *"As an agency owner, I want a bird's-eye view of all my client workspaces with health indicators, so I can proactively manage accounts."*

**What You'll Find**:

**For Agencies (Multi-Workspace)**:

- **Header Stats Row** (see [Analytics UI Patterns](/docs/design/analytics-ui-patterns#data-cards)):

  - **DataCards**: Total Clients, Active Campaigns (across all), Total Emails Sent (MTD), Aggregated Reply Rate.

- **Workspace Grid**:

  - **Each Workspace Card Shows**:

    - **Client Logo** (uploaded or initials fallback).

    - **Workspace Name**.

    - **Health Score** (0-100):

      - ✅ 90-100: Green "Healthy".

      - âš ï¸ 70-89: Yellow "Needs Attention".

      - ❌ <70: Red "Critical".

    - **Quick Stats**: "3 Active Campaigns", "1,200 Leads", "42% Open Rate".

    - **"View Workspace" Button**: Navigates to `/dashboard/workspaces/[slug]`.

- **Actions**:

  - **"+ New Workspace" Button** (top-right).

  - **Sort Dropdown**: By Name, Health Score, Campaign Count.

**For Single-Tenant Users**:

- **Simplified View**: Shows single workspace card or skips directly to workspace dashboard.

**User Journey Context**: Agency dashboard. Daily check-in for client health monitoring.

**Related Documentation**:

- [Agency Owner Persona](/docs/business/user-personas/agency-owners)

- [Multi-tenancy Architecture](/docs/technical/architecture/multi-tenancy)

**Technical Integration**:

- **Aggregated Metrics**: Fetched from OLAP database with 1-hour cache.

- **Health Score Calculation**: Based on campaign activity, deliverability, reply rates.

---

### `/dashboard/workspaces/new` - Create Workspace

**User Story**: *"As an agency, I want to quickly onboard a new client by creating a dedicated workspace, so they have isolated campaigns and settings."*

**What You'll Find**:

- **Modal or Full-Page Form**:

  - **Workspace Name**: Input field ("e.g., Acme Corp").

  - **Logo Upload**: Drag-and-drop or file picker (optional).

  - **Subdomain/Slug**:

    - Auto-generated from name: "acme-corp".

    - Editable (with validation for uniqueness).

  - **Plan Selection** (if applicable):

    - Radio buttons: Starter, Pro, Enterprise.

    - Shows pricing and feature comparison.

- **Actions**:

  - **"Create Workspace" Button**: Creates workspace and redirects to `/dashboard/workspaces/[new-slug]`.

  - **"Cancel" Button**: Closes modal.

**User Journey Context**: Onboarding new clients. Must be fast (<1 minute to complete).

**Related Documentation**:

- [Workspace Setup Guide](/docs/operations/freelancer-management/support/tutorials/workspace-creation)

**Technical Integration**:

- **Async Creation**: Workspace provisioned via background job (creates DB schemas, default settings).

- **Webhook**: Triggers notification to admin users.

---

### `/dashboard/workspaces/[slug]/settings` - Workspace Settings

**User Story**: *"As a workspace admin, I want to manage workspace details, team members, and billing, so I can maintain control over the account."*

**What You'll Find**:

**Tabbed Interface**:

#### Tab 1: General

- **Workspace Name**: Editable input.

- **Logo**: Upload/change logo.

- **Slug**: Display only (cannot change after creation).

- **Danger Zone**:

  - **"Delete Workspace" Button**: Red button with confirmation modal.

    - Warning: "This will permanently delete all campaigns, leads, and data."

#### Tab 2: Members

- **Members Table**:

  - Columns: Name, Email, Role, Status, Actions.

  - **Roles**: Owner, Admin, Member.

  - **Status**: Active, Invited (pending).

- **"Invite Member" Button**:

  - Opens modal with Email + Role selector.

  - Sends invitation email with magic link.

- **Row Actions**: Change Role, Remove (cannot remove self).

#### Tab 3: Billing (if workspace has separate billing)

- **Current Plan**: Shows plan name and limits.

- **"Upgrade Plan" Button**: Redirects to billing flow.

- **Billing Contact**: Email for invoices.

**User Journey Context**: Infrequent access for account maintenance.

**Related Documentation**:

- [Team Management](/docs/business/collaboration/team-roles)

- [Billing & Subscriptions](/docs/business/finance/pricing/overview)

**Technical Integration**:

- **RBAC**: Only Owners/Admins can access this page.

- **Stripe Integration** (if workspace billing):

  - Customer portal link for payment method updates.

---

### Workspace Health Score Calculation

**User Story**: *"As an agency, I want to understand what contributes to workspace health, so I can improve client outcomes."*

**Health Score Factors** (Weighted Average):

1. **Campaign Activity** (30%):

   - Active campaigns vs. total campaigns.

   - Recent campaign sends.

2. **Deliverability** (40%):

   - Bounce rate < 5%.

   - Spam complaint rate < 0.1%.

3. **Engagement** (20%):

   - Open rate > 20%.

   - Reply rate > 5%.

4. **System Health** (10%):

   - DNS records verified.

   - Email accounts not blacklisted.

**Display**:

- Score shown on workspace card and in workspace dashboard header.

- **Tooltip**: "Health Score factors: Deliverability, Engagement, Activity."

**Related Documentation**:

- [Health Metrics](/docs/operations/analytics/metrics/workspace-health)

---

### Workspace Context Switching (Revisited)

**User Story**: *"As an agency user working across multiple clients, I want seamless context switching, so I don't lose my workflow."*

**Mechanism** (as described in [Core App Structure](/docs/design/routes/core-app-structure)):

- **Context Switcher in Sidebar**: Dropdown showing current workspace.

- **Quick Switch**: Click to change, updates URL and sidebar nav.

- **Persistence**: Last workspace stored in localStorage.

**Workspace-Specific State**:

- Campaigns, Templates, Domains, Settings all scoped to selected workspace.

- Global views (Inbox, Leads) aggregate across accessible workspaces.

**Related Documentation**:

- [Agency Client Workspaces UX Patterns](/docs/design/ui-library#multi-tenancy)

## 5. Related API Endpoints

| Route | Related API | Description |
|---|---|---|
| `/workspaces` | [Workspaces API](/docs/implementation-technical/api/tenant-api/workspaces) | `GET /api/v1/tenant/workspaces` (List), `POST` (Create). |
| `/workspaces/[slug]` | [Workspaces API](/docs/implementation-technical/api/tenant-api/workspaces) | `GET /api/v1/tenant/workspaces/{id}` (Details & Stats). |
| `/workspaces/[slug]/settings` | [Workspaces API](/docs/implementation-technical/api/tenant-api/workspaces) | `PATCH /api/v1/tenant/workspaces/{id}` (Update settings). |
| `/workspaces/[slug]/members` | [Workspaces API](/docs/implementation-technical/api/tenant-api/workspaces) | `POST /api/v1/tenant/workspaces/{id}/members` (Invite). |

## 6. Data Strategy

- **Fetching Method**:

  - **Workspace List**: Server Component.

  - **Aggregated Stats**: Fetched in parallel with list.

- **Caching**:

  - **Workspace List**: Cached for 1 minute.

  - **Health Scores**: Cached for 1 hour (computed via background job).

- **Optimistic Updates**:

  - "Create Workspace" adds card to grid immediately (with loading state while provisioning).

## 7. Edge Cases & Error Handling

- **Slug Collision**: If auto-generated slug exists, append number (e.g., `acme-corp-1`).

- **Creation Failure**: If background provisioning fails, show error on card and "Retry" action.

- **Last Admin**: Cannot remove self if you are the only admin.

- **Delete Workspace**: Requires typing workspace name to confirm (destructive action).

## 8. Component Architecture

### Page Components

- **`WorkspaceGrid`** (Server)

  - Props: `workspaces: Workspace[]`

  - Sub-components: `WorkspaceCard` (with stats sparklines).

- **`MemberInviteForm`** (Client)

  - Props: `workspaceId: string`

  - Features: Multi-email input, role selector.

- **`SettingsForm`** (Client)

  - Props: `initialValues: WorkspaceSettings`

  - Features: Logo upload (Drag & Drop), dirty state tracking.

### Shared Components

- **`RoleBadge`**: Reused in Team Settings and Admin Users.

- **`Sparkline`**: Reused in Workspace Card and Dashboard.
