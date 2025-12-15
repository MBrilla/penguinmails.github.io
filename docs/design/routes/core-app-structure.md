---
title: "Core Application Structure"
description: "Shell of the application handling navigation, context switching, and global state"
last_modified_date: "2025-12-16"
level: "2"
persona: "Technical Teams"
---

# Core Application Structure

## 1. Purpose & Context (The "Why")

* **Goal**: Define the shell of the application that persists across views, handling navigation, context switching, and global state.

* **Feature References**:

  * [Frontend Routing Map](/docs/design/frontend-routing-map)

* **User Journey**: The user spends 100% of their authenticated time within this shell. It must be responsive and context-aware.

## 2. UI Patterns & Components (The "How")

* **Core Components**:

  * `Sidebar`: Collapsible, context-aware navigation tree.

  * `Topbar`: Breadcrumbs, Global Search, User Menu.

  * `ContextSwitcher`: Dropdown in Sidebar header to switch between Workspaces.

* **Layout**: `DashboardLayout` (Shell).

## 3. Route Specifications

| Route | Access | Purpose | State/Data Requirements |
|---|---|---|---|
| `/dashboard` | Auth | Layout Wrapper | **Server Component**. Fetches User Profile + List of accessible Workspaces. Hydrates Client Context. |
| `/dashboard/*` | Auth | Nested Routes | Renders children within the `DashboardLayout`. Handles loading states via `loading.tsx` (Suspense). |

### Navigation Logic

* **Global Context**: When at `/dashboard` root or global tools (Inbox, Leads), Sidebar shows global nav items.

* **Workspace Context**: When at `/dashboard/workspaces/[slug]`, Sidebar switches to Workspace-specific items (Campaigns, Templates, etc.).

* **Breadcrumbs**: Auto-generated based on URL segments.

## 4. Detailed View Descriptions

### `/dashboard` - Application Shell

**User Story**: *"As a user, I need a consistent navigation structure that adapts to my current context (global vs. workspace), so I can efficiently access all features without getting lost."*

**What You'll Find**:

* **Sidebar (Left Panel)**:

  * **Logo & Context Switcher**: Dropdown showing current workspace or "Global View".

  * **Navigation Tree**:

    * Global Context: Dashboard, Inbox, Leads, Workspaces, Settings.

    * Workspace Context: Overview, Campaigns, Templates, Domains, Settings.

  * **Collapse Button**: Minimize sidebar to icon-only mode for more screen space.

* **Topbar (Horizontal)**:

  * **Breadcrumbs**: `Dashboard > Workspaces > Client A > Campaigns`.

  * **Global Search**: Quick access to campaigns, leads, templates (Cmd+K shortcut).

  * **Notifications Bell**: Unread count badge, dropdown with recent activity.

  * **User Menu**: Avatar with dropdown (Profile, Billing, Logout).

* **Main Content Area**:

  * Renders children routes.

  * **Loading State**: Skeleton screens during Suspense boundaries.

**User Journey Context**: Persistent shell for all authenticated views. Optimized for daily power users who navigate frequently.

**Related Documentation**:

* [Navigation Patterns](/docs/design/ui-library#navigation)

* [Layout Components](/docs/design/component-library#layouts)

**Technical Integration**:

* **Server Component**: Fetches User Profile + Workspaces list on server.

* **Client Context**: Hydrates React Context with user data for client components.

* **Middleware Protection**: All `/dashboard/*` routes protected by auth middleware.

---

### Context Switching Behavior

**User Story**: *"As an agency owner managing multiple clients, I want to seamlessly switch between different workspaces without losing my place, so I can efficiently multitask across accounts."*

**How It Works**:

1. **Dropdown in Sidebar Header**: Shows all accessible workspaces.

2. **Click to Switch**: Updates URL to `/dashboard/workspaces/[new-slug]` without full page reload (client-side navigation).

3. **Sidebar Rehydrates**: Navigation items update to show workspace-specific routes.

4. **Persisted State**: Last viewed workspace stored in localStorage for return visits.

**Edge Cases**:

* User with only 1 workspace: Context switcher hidden, always in workspace mode.

* User with 20+ workspaces: Dropdown becomes searchable for quick filtering.

**Related Documentation**:

* [Agency Features](/docs/business/user-personas/agency-owners)

* [Multi-tenancy Architecture](/docs/technical/architecture/multi-tenancy)

---

### Loading & Error States

**User Story**: *"As a user on a slow connection, I want to see clear loading indicators, so I know the app is working and not frozen."*

**Loading Patterns**:

* **Initial Load**: Full-page spinner with PenguinMails logo.

* **Route Navigation**: Top progress bar (NProgress.js style).

* **Suspense Boundaries**: Skeleton screens for main content area.

**Error Patterns**:

* **Network Error**: "Connection lost. Retrying..." banner at top.

* **404 on Child Route**: "Page not found" in main content, sidebar remains active.

* **Auth Expired**: Auto-redirect to `/login` with `?redirect=/dashboard/...` query.

**Related Documentation**:

* [Error Handling Patterns](/docs/design/ui-library#error-handling)

* [Loading States](/docs/design/design-system#feedback)

## 5. Related API Endpoints

| Route | Related API | Description |
|---|---|---|
| `/dashboard` | [Workspaces API](/docs/implementation-technical/api/tenant-api/workspaces) | `GET /api/v1/tenant/workspaces` used to populate the sidebar context switcher. |
| `/dashboard` | **NileDB SDK** | `getUser()` used to hydrate user profile in the Topbar. |

## 6. Data Strategy

* **Fetching Method**:

* **Fetching Method**:

  * **Server Components**: The root layout fetches the user profile and list of workspaces.

  * **Parallel Fetching**: `Promise.all([getUser(), getWorkspaces()])` to minimize waterfall.

* **Caching**:

  * Workspace list cached for 1 minute or invalidated on workspace creation/deletion.

  * User profile cached for session duration (unless updated).

* **State Management**:

  * **Global Context**: `UserContext` and `WorkspaceContext` providers wrap the application.

  * **URL State**: Workspace slug in URL (`/dashboard/workspaces/[slug]`) drives the current context.

## 7. Edge Cases & Error Handling

* **Workspace Not Found**: Accessing `/dashboard/workspaces/invalid-slug` -> Redirects to `/dashboard` (Global View) with "Workspace not found" toast.

* **No Access**: Accessing a valid workspace the user doesn't belong to -> 403 Forbidden Page.

* **Session Expiry**: API 401 response -> Middleware redirects to `/login?redirect=...`.

* **Deleted Workspace**: If current workspace is deleted by another admin, real-time event or next fetch redirects user to `/dashboard`.

## 8. Component Architecture

### Layout Components

* **`AppShell`** (Server)

  * Root layout provider.

* **`Sidebar`** (Client)

  * Features: Collapsible, active state highlighting, workspace switcher.

* **`Topbar`** (Client)

  * Features: Breadcrumbs, UserDropdown, NotificationBell.

* **`CommandPalette`** (Client)

  * Features: Global search (Ctrl+K), quick actions.

### Shared Components

* **`Breadcrumbs`**: Auto-generated from path segments.

* **`ThemeToggle`**: Dark/Light mode switcher.
