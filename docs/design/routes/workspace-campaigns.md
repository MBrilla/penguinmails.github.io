---
title: "Workspace Campaigns"
description: "Enable users to create, manage, and analyze cold email campaigns"
last_modified_date: "2025-12-16"
level: "2"
persona: "Technical Teams"
---

# Workspace Campaigns

## 1. Purpose & Context (The "Why")

* **Goal**: Enable users to create, manage, and analyze cold email campaigns.

* **Feature References**:

  * [Campaign Management](/docs/features/analytics/core-analytics/overview) (Analytics integration)

* **User Journey**: Core loop: Create Campaign -> Configure -> Launch -> Monitor -> Optimize.

## 2. UI Patterns & Components (The "How")

* **Core Components**:

  * `DataTable`: For the campaigns list (Sortable, Filterable).

  * `Wizard`: For `/new` campaign creation flow.

  * `RichTextEditor`: For campaign sequence editing.

* **Analytics Patterns**:

  * `DataCard`: For campaign summary stats (Sent, Open Rate, Reply Rate).

  * `LineChart`: For engagement over time.

  * See [Analytics UI Patterns](/docs/design/analytics-ui-patterns).

* **Layout**: Workspace Context.

## 3. Route Specifications

| Route | Access | Purpose | State/Data Requirements |
|---|---|---|---|
| `/dashboard/workspaces/[slug]/campaigns` | Tenant | List Campaigns | **Server Component**. Paginated list. Columns: Name, Status, Sent, Open%, Reply%. Actions: Pause, Resume, Edit. |
| `/dashboard/workspaces/[slug]/campaigns/new` | Tenant | Create Campaign | **Client Component**. Multi-step Wizard: 1. Name/Goal, 2. Audience (Upload/Select), 3. Sequence (Editor), 4. Schedule, 5. Review. |
| `/dashboard/workspaces/[slug]/campaigns/[id]` | Tenant | Campaign Dashboard | **Server Component**. Aggregated analytics view. Shows "Headline Metrics" cards top, "Engagement Trend" chart middle, "Lead Activity" table bottom. |
| `/dashboard/workspaces/[slug]/campaigns/[id]/edit` | Tenant | Edit Campaign | **Client Component**. Tabbed interface: Sequence, Audience, Settings. Auto-saving form state. |
| `/dashboard/workspaces/[slug]/campaigns/[id]/settings` | Tenant | Configuration | Form for Sending Window, Daily Limits, Tracking Options. |
| `/dashboard/workspaces/[slug]/campaigns/import` | Tenant | Import Campaign | **Client Component**. File upload (JSON/CSV) or API connector (Mailchimp/Instantly) to migrate campaigns. |

## 4. Detailed View Descriptions

### `/dashboard/workspaces/[slug]/campaigns` - Campaign List

**User Story**: *"As a marketer, I want to see all my campaigns at a glance with key metrics, so I can quickly identify which campaigns need attention."*

**What You'll Find**:

* **Header Actions**:

  * **"New Campaign" Button**: Primary CTA (top-right).

  * **Filter Dropdown**: Status (Active, Paused, Completed, Draft).

  * **Search Bar**: Search by campaign name.

* **Data Table** (see [Analytics UI Patterns](/docs/design/analytics-ui-patterns#data-tables)):

  * **Columns**: Name, Status, Sent, Open Rate, Reply Rate, Created Date, Actions.

  * **Status Badge**: Color-coded (Green=Active, Yellow=Paused, Gray=Draft).

  * **Sortable Headers**: Click to sort by any column.

  * **Row Actions**: Pause/Resume, Edit, Duplicate, Delete.

* **Empty State**:

  * Illustration + "Create your first campaign" CTA.

**User Journey Context**: Primary dashboard for campaign management. Daily check-in to monitor performance.

**Related Documentation**:

* [Campaign Management](/docs/features/analytics/core-analytics/overview)

* [DataTable Component](/docs/design/ui-library#data-tables)

**Technical Integration**:

* **Server Component**: Fetches campaigns with pagination (10/page).

* **Real-time Updates**: Polling every 30s for status changes.

---

### `/dashboard/workspaces/[slug]/campaigns/new` - Campaign Creation Wizard

**User Story**: *"As a new user, I want a guided wizard to create my first campaign without getting overwhelmed by options, so I can launch quickly."*

**What You'll Find**:

* **Step 1: Campaign Basics**

  * Name, Goal (Awareness, Lead Gen, Sales).

  * "Next" button.

* **Step 2: Audience Selection**

  * **Option A**: Upload CSV (drag-and-drop).

  * **Option B**: Select from existing Leads/Segments.

  * Shows preview of first 5 records.

* **Step 3: Email Sequence**

  * **Initial Email + Follow-ups**: Add up to 7 emails.

  * Rich text editor with variable insertion (`{{firstName}}`).

  * **Preview Button**: See how email renders.

* **Step 4: Sending Schedule**

  * Sending Window: "Mon-Fri, 9am-5pm".

  * Daily Send Limit: 50/day (adjustable).

  * Start Date selector.

* **Step 5: Review & Launch**

  * Summary of all settings.

  * **"Launch Campaign" Button** or "Save as Draft".

**User Journey Context**: Critical onboarding moment. Must be simple yet comprehensive.

**Related Documentation**:

* [Campaign Setup Best Practices](/docs/features/analytics/core-analytics/overview)

* [Wizard Pattern](/docs/design/ui-library#wizards)

**Technical Integration**:

* **Client State**: Wizard state managed in React Context.

* **Auto-save**: Draft saved to backend every 30s.

---

### `/dashboard/workspaces/[slug]/campaigns/[id]` - Campaign Analytics Dashboard

**User Story**: *"As a campaign manager, I want to see detailed engagement metrics and lead activity, so I can optimize my messaging and follow-up strategy."*

**What You'll Find**:

* **Headline Metrics Row** (see [Analytics UI Patterns](/docs/design/analytics-ui-patterns#data-cards)):

  * **DataCards**: Sent, Opened (%), Clicked (%), Replied (%).

  * **Trend Indicators**: ↑ 12% vs. last 7 days.

* **Engagement Chart**:

  * **LineChart**: Sent/Opened/Replied over time (Daily view, last 30 days).

  * **Hover Tooltip**: Shows exact counts.

* **Lead Activity Table**:

  * Columns: Lead Name, Email, Status (Sent, Opened, Replied), Last Activity.

  * **Click Name**: Opens slide-over with full lead details.

* **Actions**:

  * **"Edit Campaign" Button**.

  * **"Pause Campaign" Button** (if active).

**User Journey Context**: Monitoring hub. Accessed multiple times daily during active campaigns.

**Related Documentation**:

* [Analytics Views Architecture](/docs/features/analytics/views-architecture)

* [Campaign Analytics](/docs/features/analytics/core-analytics/overview)

**Technical Integration**:

* **Aggregated Data**: Fetched from OLAP (ClickHouse) with daily refresh (Future/2026 Spike).

* **Caching**: ISR with 5-minute revalidation.

---

### `/dashboard/workspaces/[slug]/campaigns/[id]/edit` - Campaign Editor

**User Story**: *"As a user who needs to fix a typo in an active campaign, I want to make quick edits without disrupting the campaign, so I can maintain quality."*

**What You'll Find**:

* **Tabbed Interface**:

  * **Tab 1: Sequence**: Edit email content (Rich Text Editor).

  * **Tab 2: Audience**: Add/remove leads (with warnings if campaign is active).

  * **Tab 3: Settings**: Sending window, daily limits, tracking toggles.

* **Auto-save Indicator**: "Saving..." → "All changes saved" (top-right).

* **Warnings Modal**:

  * "Editing an active campaign may affect delivery. Continue?"

**User Journey Context**: Infrequent use for corrections or optimizations.

**Related Documentation**:

* [Campaign Management](/docs/features/analytics/core-analytics/overview)

**Technical Integration**:

* **Optimistic UI**: Changes reflected immediately, synced to backend asynchronously.

* **Conflict Detection**: If campaign modified elsewhere, shows merge prompt.

---

### `/dashboard/workspaces/[slug]/campaigns/[id]/settings` - Campaign Configuration

**User Story**: *"As a power user, I want granular control over sending behavior and tracking settings, so I can optimize deliverability."*

**What You'll Find**:

* **Form Groups**:

  * **Sending Window**: Time range selector (9am-5pm, Timezone-aware).

  * **Daily Send Limit**: Slider (10-500/day).

  * **Tracking Options**: Toggles for "Open Tracking", "Click Tracking", "Unsubscribe Link".

  * **Reply Handling**: "Pause campaign on reply" checkbox.

* **Save Button**: Sticky at bottom.

**User Journey Context**: Advanced settings. Accessed during campaign setup or optimization.

**Related Documentation**:

* [Warm-ups](/docs/features/warmup/email-warmups/overview)

* [Inbox Rotation](/docs/features/inbox/inbox-rotation/overview)

**Technical Integration**:

* **Background Jobs**: Changes trigger recalculation of sending schedule (background queue job).

---

### `/dashboard/workspaces/[slug]/campaigns/import` - Campaign Import (Migration)

**User Story**: *"As a user migrating from another platform, I want to import my existing campaigns and contacts so I don't have to rebuild everything from scratch."*

**What You'll Find**:

* **Import Source Selection**:
  * **File Upload**: JSON or CSV export from other tools.
  * **Direct Integration** (Post-MVP): "Connect Mailchimp", "Connect Instantly".

* **Mapping Wizard**:
  * **Campaign Name**: Auto-filled from file.
  * **Sequence Mapping**: Map email steps and delays.
  * **Contact Mapping**: Map contact fields.

* **Review & Import**:
  * Summary of campaigns to be created.
  * **"Start Import" Button**.

**User Journey Context**: Critical for new user activation (Migration).

**Related Documentation**:

* [Migration Guide](/docs/operations/migration/overview)

**Technical Integration**:

* **Parsers**: Client-side parsing of common export formats.
* **Bulk Create**: Transactional creation of Campaign + Sequence + Contacts.

## 5. Related API Endpoints

| Route | Related API | Description |
|---|---|---|
| `/campaigns` | [Campaigns API](/docs/implementation-technical/api/tenant-api/campaigns) | `GET /api/v1/tenant/campaigns` (List), `POST` (Create). |
| `/campaigns/[id]` | [Campaigns API](/docs/implementation-technical/api/tenant-api/campaigns) | `GET /api/v1/tenant/campaigns/{id}` (Details & Analytics). |
| `/campaigns/[id]/edit` | [Campaigns API](/docs/implementation-technical/api/tenant-api/campaigns) | `PATCH /api/v1/tenant/campaigns/{id}` (Update). |
| `/campaigns/[id]/settings` | [Campaigns API](/docs/implementation-technical/api/tenant-api/campaigns) | `PATCH /api/v1/tenant/campaigns/{id}/config`. |

## 6. Data Strategy

* **Fetching Method**:

  * **List View**: Server Component with searchParams for pagination/filtering.

  * **Analytics View**: Server Component fetching aggregated OLAP data.

* **Caching**:

  * **Campaign List**: Cached for 30s. Invalidated on Create/Delete/StatusChange.

  * **Analytics**: Cached for 5 minutes (heavy query).

* **Real-time Updates**:

  * **Status Polling**: Client component polls `/api/v1/tenant/campaigns/status` every 30s to update "Sending" vs "Completed" badges.

* **Optimistic Updates**:

  * Pause/Resume actions update UI immediately, revert on failure.

## 7. Edge Cases & Error Handling

* **Campaign Not Found**: 404 Page with "Return to Campaigns" button.

* **Editing Active Campaign**: Warning modal explains risks (pausing recommended).

* **Zero Leads**: Cannot launch campaign without leads. "Import Leads" CTA shown.

* **Quota Exceeded**: Launch blocked if monthly email limit reached. Upgrade prompt shown.

* **Campaign Not Found**: 404 Page with "Return to Campaigns" button.

* **Editing Active Campaign**: Warning modal explains risks (pausing recommended).

* **Zero Leads**: Cannot launch campaign without leads. "Import Leads" CTA shown.

* **Quota Exceeded**: Launch blocked if monthly email limit reached. Upgrade prompt shown.

## 8. Component Architecture

### Page Components

* **`CampaignListTable`** (Server)

  * Props: `campaigns: Campaign[]`, `pagination: PaginationData`

  * Dependencies: `StatusBadge`, `ActionMenu`

* **`CampaignWizard`** (Client)

  * State: Multi-step form state (Steps: Details -> Audience -> Content -> Schedule)

  * Sub-components: `AudienceSelector`, `TemplatePicker`, `ScheduleDatePicker`

* **`AnalyticsDashboard`** (Server)

  * Props: `stats: CampaignStats`

  * Sub-components: `MetricCard`, `EngagementChart` (Recharts), `ActivityFeed`

### Shared Components

* **`StatusBadge`**: Reused across Campaigns and Jobs.

* **`MetricCard`**: Reused in Dashboard and Analytics.
