---
title: "Workspace Templates"
description: "Manage reusable email templates for the workspace"
last_modified_date: "2025-12-16"
level: "2"
persona: "Technical Teams"
---

# Workspace Templates

## 1. Purpose & Context (The "Why")

- **Goal**: Manage reusable email templates for the workspace.

- **Feature References**:

  - [Templates](/docs/features/templates)

- **User Journey**: Create templates first (Content) to be used in Campaigns. Templates grow as a library over time.

## 2. UI Patterns & Components (The "How")

- **Core Components**:

  - `CardGrid`: For Templates gallery view.

  - `RichTextEditor`: For template content editing.

  - `VariableInsertion`: For dynamic personalization tokens.

- **Analytics Patterns**:

  - `DataCard`: Template usage stats (# of campaigns using this template).

- **Layout**: Workspace Context.

## 3. Route Specifications

| Route | Access | Purpose | State/Data Requirements |
|---|---|---|---|
| `/dashboard/workspaces/[slug]/templates` | Tenant | List Templates | **Server Component**. Grid or List view. Filters: Category, Tags. |
| `/dashboard/workspaces/[slug]/templates/new` | Tenant | Create Template | **Client Component**. Rich Text Editor with variable insertion (`{{firstName}}`). Preview mode. |
| `/dashboard/workspaces/[slug]/templates/[id]` | Tenant | View Template | **Server Component**. Template preview. Usage history (which campaigns use this?). |
| `/dashboard/workspaces/[slug]/templates/[id]/edit` | Tenant | Edit Template | **Client Component**. Edit form with auto-save. Version history. |

## 4. Detailed View Descriptions

### `/dashboard/workspaces/[slug]/templates` - Template Gallery

**User Story**: *"As a marketer, I want to reuse successful email templates across campaigns, so I can maintain consistency and save time."*

**What You'll Find**:

- **View Toggle**: Grid (Cards) or List (Table) view.

- **Template Cards** (Grid View):

  - Thumbnail preview

  - Template name

  - Category tag (Welcome, Follow-up, Outreach)

  - Usage count ("Used in 5 campaigns")

  - Quick actions: Edit, Duplicate, Delete

- **Filters**:

  - **Category**: All, Welcome, Follow-up, Outreach, Nurture.

  - **Tags**: Custom tags for organization.

  - **Search**: Full-text search by name or content.

- **Empty State**: "Create your first template" with sample templates gallery.

- **Header Actions**:

  - **"New Template" Button**: Primary CTA (top-right).

**User Journey Context**: Template library grows over time. Must be searchable and well-organized.

**Related Documentation**:

- [Email Best Practices](/docs/business/marketing/overview)

- [CardGrid Component](/docs/design/ui-library#grids)

- [Templates Feature](/docs/features/templates)

**Technical Integration**:

- **Server Component**: Fetches templates with pagination.

- **Filtering**: Client-side filtering for real-time responsiveness.

---

### `/dashboard/workspaces/[slug]/templates/new` - Template Creator

**User Story**: *"As a content creator, I want to build email templates with personalization variables, so I can create engaging, customized messages."*

**What You'll Find**:

- **Rich Text Editor**:

  - Formatting toolbar (Bold, Italic, Lists, Links, Headers).

  - **Variable Insertion Button**: Dropdown showing `{{firstName}}`, `{{lastName}}`, `{{company}}`, `{{customField1}}`, etc.

  - **Image Upload**: Drag-and-drop or URL.

  - **HTML Mode**: Advanced users can edit raw HTML.

- **Sidebar**:

  - **Template Name**: Input field (required).

  - **Category**: Dropdown (Welcome, Follow-up, Outreach, Nurture, Custom).

  - **Tags**: Multi-select for organization.

  - **Description**: Optional text area for internal notes.

- **Footer Actions**:

  - **Preview Button**: Opens modal with rendered template (with sample data).

  - **Save Draft** / **Save & Close**.

**User Journey Context**: Creative process. Must be intuitive with low learning curve.

**Related Documentation**:

- [Content Guidelines](/docs/business/messaging/overview)

- [Rich Text Editor](/docs/design/ui-library#editors)

- [Template Variables](/docs/features/templates#variables)

**Technical Integration**:

- **Auto-save**: Debounced save every 5s.

- **Variable Validation**: Warns if using undefined variables.

- **Version Control**: Each save creates a new version (for history tracking).

---

### `/dashboard/workspaces/[slug]/templates/[id]` - Template Detail View

**User Story**: *"As a campaign manager, I want to see where a template is being used, so I can understand its impact and avoid breaking active campaigns."*

**What You'll Find**:

- **Template Header**:

  - Template name

  - Category badge

  - Tags

  - **Edit Button** (top-right)

- **Preview Panel**:

  - Rendered template with sample data

  - **View in HTML** toggle

- **Usage Section**:

  - **"Used in X campaigns"** heading

  - List of campaigns using this template:

    - Campaign name (linked)

    - Status (Active, Paused, Draft)

    - Last used date

- **Metadata**:

  - Created date

  - Last modified date

  - Created by (user name)

- **Actions**:

  - **Edit** (navigates to edit view)

  - **Duplicate** (creates copy with "Copy of..." prefix)

  - **Delete** (with confirmation if template is in use)

**User Journey Context**: Reference view for template management.

**Related Documentation**:

- [Template Management](/docs/features/templates)

**Technical Integration**:

- **Server Component**: Pre-rendered preview.

- **Usage Tracking**: Queries campaigns table for template references.

---

### `/dashboard/workspaces/[slug]/templates/[id]/edit` - Template Editor

**User Story**: *"As a user who needs to update a template, I want to make edits while seeing campaigns that use it, so I can make informed changes."*

**What You'll Find**:

- **Same Rich Text Editor** as `/new` route.

- **Warning Banner** (if template is in active campaigns):

  - "⚠️ This template is used in 3 active campaigns. Changes will affect future sends."

  - **View Campaigns** link.

- **Version History** (Collapsible panel):

  - List of previous versions with timestamps

  - **Restore** button for each version

- **Auto-save Indicator**: "Saving..." → "All changes saved" (top-right).

**User Journey Context**: Infrequent use for corrections or optimizations.

**Related Documentation**:

- [Template Management](/docs/features/templates)

- [Version Control](/docs/design/ui-library#versioning)

**Technical Integration**:

- **Optimistic UI**: Changes reflected immediately, synced to backend asynchronously.

- **Conflict Detection**: If template modified elsewhere, shows merge prompt.

- **Version History**: Stored as separate records with references to parent template.

## 5. Related API Endpoints

| Route | Related API | Description |
|---|---|---|
| `/templates` | [Templates API](/docs/implementation-technical/api/tenant-api/templates) | `GET /api/v1/tenant/templates` (List), `POST` (Create). |
| `/templates/[id]` | [Templates API](/docs/implementation-technical/api/tenant-api/templates) | `GET /api/v1/tenant/templates/{id}` (Details). |
| `/templates/[id]/edit` | [Templates API](/docs/implementation-technical/api/tenant-api/templates) | `PATCH /api/v1/tenant/templates/{id}` (Update). |

## 6. Data Strategy

- **Fetching Method**:

  - **List View**: Server Component. Fetches all templates (dataset usually small < 100).

  - **Detail View**: Server Component.

- **Caching**:

  - **Templates List**: Cached for 1 hour. Invalidated on Create/Update/Delete.

  - **Client-side Filtering**: List data passed to client component for instant category/tag filtering without server roundtrips.

- **State Management**:

  - **Editor State**: Controlled component state in `RichTextEditor`.

  - **Auto-save**: `useDebounce` hook triggers `PATCH` request after 2s of inactivity.

## 7. Edge Cases & Error Handling

- **Template Not Found**: 404 Page.

- **Concurrent Edits**: If another user edits the same template, show "New version available" toast (Last Write Wins strategy with notification).

- **Unsaved Changes**: Navigation away triggers "You have unsaved changes" browser alert if auto-save hasn't completed.

- **Deletion Blocked**: Cannot delete template if used in Active/Paused campaigns. Show list of blocking campaigns.

## 8. Component Architecture

### Page Components

- **`TemplateList`** (Server)

  - Props: `templates: Template[]`

  - Features: Grid/List view toggle, instant search.

- **`TemplateEditor`** (Client)

  - Props: `initialContent: string`

  - Library: TipTap or Quill (Headless).

  - Features: Variable picker (`{{firstName}}`), HTML source view.

- **`PreviewModal`** (Client)

  - Props: `html: string`, `device: 'mobile' | 'desktop'`

  - Features: Iframe isolation for style safety.

### Shared Components

- **`VariablePicker`**: Reused in Campaign Wizard.

- **`DeviceToggle`**: Reused in Preview Modal.
