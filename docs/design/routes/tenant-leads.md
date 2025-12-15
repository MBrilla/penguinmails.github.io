---
title: "Tenant Leads"
description: "Centralized management of all contacts/leads across the tenant's account"
last_modified_date: "2025-12-16"
level: "2"
persona: "Technical Teams"
---

# Tenant Leads (Global)

## 1. Purpose & Context (The "Why")

- **Goal**: Centralized management of all contacts/leads across the tenant's account, regardless of which workspace they belong to.

- **Feature References**:

  - [Lead Management](/docs/features/analytics/core-analytics/overview) (Segmentation)

- **User Journey**: Import leads → Segment them → Assign to Campaigns.

## 2. UI Patterns & Components (The "How")

- **Core Components**:

  - `DataTable`: High-performance table for large datasets. Needs "Select All", "Bulk Actions".

  - `FilterBuilder`: Complex filtering UI (e.g., "Industry IS Tech AND Status IS NOT Bounced").

  - `SlideOver`: For quick view of Lead Details without leaving the table.

- **Analytics Patterns**: N/A.

- **Layout**: Global Context.

## 3. Route Specifications

| Route | Access | Purpose | State/Data Requirements |
|---|---|---|---|
| `/dashboard/leads` | Tenant | All Leads | **Server Component**. Heavy data view. Server-side pagination/sorting. Bulk Actions: "Add to Campaign", "Delete", "Export". |
| `/dashboard/leads/[id]` | Tenant | Lead Profile | **Client Component** (or Page). Shows timeline of interactions (Emails sent, Replies, Opens). Editable fields. |
| `/dashboard/leads/segments` | Tenant | Segments | List of saved dynamic segments. |
| `/dashboard/leads/lists` | Tenant | Static Lists | List of static uploaded lists. Upload CSV modal. |

## 4. Detailed View Descriptions

### `/dashboard/leads` - Global Leads Database

**User Story**: *"As a sales manager, I want to see all my leads in one place with filtering and bulk actions, so I can efficiently manage my entire contact database."*

**What You'll Find**:

- **Header Actions**:

  - **"Import Leads" Button**: Opens modal with CSV upload (drag-and-drop).

  - **" Export" Button**: Downloads filtered view as CSV.

  - **Search Bar**: Search by name, email, company.

- **Filter Panel** (Left Sidebar or Collapsible):

  - **Quick Filters**: All Leads, Recently Added, Engaged, Bounced.

  - **Advanced Filter Builder**:

    - Add conditions: "Industry", "Status", "Last Contact Date", etc.

    - Logic: AND/OR operators.

    - **Save as Segment** button.

- **Data Table** (see [Analytics UI Patterns](/docs/design/analytics-ui-patterns#data-tables)):

  - **Columns**: Name, Email, Company, Status, Last Contact, Tags, Actions.

  - **Status Icons**: Active, Sent, Opened, Replied, Bounced.

  - **Checkbox Column**: Select for bulk actions.

  - **Pagination**: Server-side, 50 leads/page.

- **Bulk Actions Bar** (appears when rows selected):

  - "Add to Campaign", "Add Tag", "Export Selection", "Delete".

- **Empty State**: "Import your first leads" with sample CSV download link.

**User Journey Context**: Central hub for lead management. High-frequency access.

**Related Documentation**:

- [Segmentation Strategy](/docs/business/marketing/overview)

- [DataTable Component](/docs/design/ui-library#data-tables)

**Technical Integration**:

- **Virtualized Scrolling**: For tables with 10,000+ leads.

- **Server-side Pagination**: Fetches only visible page from database.

- **Export Job**: Large exports (>1000 leads) run via background job, email download link.

---

### `/dashboard/leads/[id]` - Lead Profile

**User Story**: *"As a salesperson, I want to see a lead's complete interaction history, so I can personalize my outreach and avoid redundant messaging."*

**What You'll Find**:

- **Lead Header**:

  - **Name** (editable inline).

  - **Company** (editable).

  - **Tags**: Pill-style tags with "+" to add new.

  - **Status Dropdown**: Active, Interested, Not Interested, Bounced.

- **Contact Information Section**:

  - Email (primary), Phone, LinkedIn URL (all editable).

- **Activity Timeline** (Vertical, Chronological):

  - Icons for each event type:

    - Email Sent (from Campaign X).

    - Email Opened (timestamp).

    - Link Clicked (which link).

    - Replied (shows reply preview).

  - **"View Full Email"** expandable.

- **Custom Fields Section**:

  - User-defined fields (e.g., "Industry", "Company Size").

  - **"+ Add Field" Button**.

- **Actions Panel** (Right Sidebar):

  - **"Add to Campaign" Button**.

  - **"Send One-off Email" Button** (quick compose).

  - **"Delete Lead"** (with confirmation).

**User Journey Context**: Detailed view for high-value leads. Accessed before personalized outreach.

**Related Documentation**:

- [CRM Integration](/docs/technical/integrations/crm-sync)

- [SlideOver Component](/docs/design/ui-library#overlays)

**Technical Integration**:

- **Real-time Activity**: Subscribes to lead activity events via WebSocket.

- **Optimistic Updates**: Field edits reflected immediately, synced asynchronously.

---

### `/dashboard/leads/segments` - Dynamic Segments

**User Story**: *"As a marketer, I want to create reusable segments based on lead behavior, so I can target specific groups with tailored campaigns."*

**What You'll Find**:

- **Segment List**:

  - **Card Layout**: Each segment shows Name, Lead Count, Last Updated.

  - **"Create Segment" Button** (top-right).

- **Create Segment Modal**:

  - **Segment Name**: Input field.

  - **Filter Builder** (same as Lead List advanced filters):

    - Example: "Opened email in last 30 days AND Industry = SaaS".

  - **Live Preview**: Shows lead count matching criteria.

  - **Save Button**.

- **Segment Card Actions**:

  - **View Leads**: Redirects to `/dashboard/leads?segment=[id]`.

  - **Edit**: Modify filter criteria.

  - **Use in Campaign**: Quick action to create campaign with this segment.

**User Journey Context**: Advanced feature for power users. Enables sophisticated targeting.

**Related Documentation**:

- [Segmentation Best Practices](/docs/business/marketing/overview)

- [Filter Builder Component](/docs/design/ui-library#filters)

**Technical Integration**:

- **Dynamic Queries**: Segments stored as JSON filter definitions, executed on query.

- **Caching**: Segment lead counts cached for 5 minutes.

---

### `/dashboard/leads/lists` - Static Lists

**User Story**: *"As a user, I want to upload a CSV of leads for a specific campaign, so I can quickly target an external list."*

**What You'll Find**:

- **List Cards**:

  - Name, Lead Count, Upload Date.

  - **"Upload New List" Button**.

- **Upload Modal**:

  - **Drag-and-drop CSV**: Shows file name and size.

  - **Column Mapping**:

    - "Map CSV columns to lead fields (Email, First Name, etc.)."

    - Auto-detection for common headers.

  - **Preview**: First 5 rows with mapped fields.

  - **"Import" Button**: Starts background job for large lists.

  - **Migration Tools**:
    - "Importing from Mailchimp/ActiveCampaign?"
    - **"Use Migration Wizard" Link**: Redirects to specialized migration flow.

- **List Actions**:

  - **View Leads**: See all leads in this list.

  - **Export**: Download as CSV.

  - **Delete**: Remove list (with confirmation).

**User Journey Context**: One-time or periodic imports. Must handle large files gracefully.

**Related Documentation**:

- [Lead Import Guide](/docs/operations/freelancer-management/support/tutorials/lead-import)

- [CSV Format Requirements](/docs/technical/data-formats/csv-specs)

**Technical Integration**:

- **Background Processing**: Imports >1000 leads run via background queue system.

- **Duplicate Detection**: Checks for existing emails, shows merge options.

- **Error Handling**: Invalid rows logged, user gets summary email.

## 5. Related API Endpoints

| Route | Related API | Description |
|---|---|---|
| `/leads` | [Leads API](/docs/implementation-technical/api/tenant-api/leads) | `GET /api/v1/tenant/leads` (List with filters), `POST` (Create). |
| `/leads/[id]` | [Leads API](/docs/implementation-technical/api/tenant-api/leads) | `GET /api/v1/tenant/leads/{id}` (Details & Activity). |
| `/leads/segments` | [Leads API](/docs/implementation-technical/api/tenant-api/leads) | `GET /api/v1/tenant/leads/segments`. |
| `/leads/import` | [Leads API](/docs/implementation-technical/api/tenant-api/leads) | `POST /api/v1/tenant/leads/import` (Bulk Import). |

## Related Documentation

### Planning & Strategy

- [Lead Management README](/docs/features/leads/README) - Feature area overview with MVP status

- [Lead Management Roadmap](/docs/features/leads/roadmap) - Detailed timeline and quarterly breakdown

- [Product Roadmap](/docs/operations/roadmap/product-roadmap/overview) - Overall product timeline

- [Executive Roadmap](/docs/business/roadmap/executive-roadmap) - Strategic feature delivery

### Feature Documentation

- [Leads Management](/docs/features/leads/leads-management) - Core contact database

- [Contact Segmentation](/docs/features/leads/contact-segmentation) - Dynamic and static segmentation

- [Import/Export](/docs/features/leads/import-export/overview) - Bulk contact operations

- [Lead Scoring](/docs/features/leads/lead-scoring) - Behavioral and demographic scoring

### Implementation Tasks

- Epic 7: Leads Management - Internal task reference for implementation breakdown

### Related Features

- [Campaign Management](/docs/features/campaigns/campaign-management/hub) - Use segments in campaigns

- [Inbox Management](/docs/features/inbox/README) - Reply management linked to contacts

- [Integrations](/docs/features/integrations/README) - CRM integrations (Post-MVP)

## 6. Data Strategy

- **Fetching Method**:

  - **Leads Table**: Server Component with searchParams. Returns `data` array + `totalCount`.

  - **Lead Profile**: Server Component.

- **Caching**:

  - **Leads List**: No caching (highly dynamic).

  - **Segments**: Cached for 5 minutes.

- **Pagination**:

  - **Cursor-based pagination** preferred for performance with >1M rows.

- **Optimistic Updates**:

  - "Add Tag" and "Change Status" update UI immediately.

## 7. Edge Cases & Error Handling

- **Import Failures**:

  - **Invalid CSV**: Show row-by-row error report.

  - **Threshold Exceeded**: If import > Plan Limit, block and show upgrade modal.

- **Bulk Action Failures**: If 5/100 updates fail, show "95 succeeded, 5 failed" with retry option.

- **Empty Search**: "No leads found matching 'xyz'. Clear filters?"

## 8. Component Architecture

### Page Components

- **`LeadsDataGrid`** (Client Wrapper)

  - Props: `initialData: Lead[]`, `columns: ColumnDef[]`

  - Features: TanStack Table, server-side sorting/filtering hooks, row selection.

- **`SegmentBuilder`** (Client)

  - Complex query builder UI (AND/OR logic).

  - Output: JSON logic object for API.

- **`LeadProfileDrawer`** (Client)

  - Slide-over panel for lead details.

  - Sub-components: `LeadTimeline`, `LeadProperties`, `NotesWidget`.

### Shared Components

- **`FilterBar`**: Reused in Campaigns and Logs.

- **`TagInput`**: Async select for adding/creating tags.
