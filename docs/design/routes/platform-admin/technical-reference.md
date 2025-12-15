---
title: "Platform Admin Technical Reference"
description: "API endpoints, data strategy, and component architecture for admin routes"
last_modified_date: "2025-12-16"
level: "3"
persona: "Technical Teams"
---

# Platform Admin Technical Reference

## Related API Endpoints

| Route | Related API | Description |
|---|---|---|
| `/dashboard/admin/plans` | [Plans API](/docs/implementation-technical/api/platform-api/plans) | `GET /api/v1/platform/admin/plans`, `POST`, `PUT /{id}` |
| `/users` | [Admin API](/docs/implementation-technical/api/platform-api/admin) | `GET /api/v1/platform/admin/users`, `GET .../audit` |
| `/tenants` | [Admin API](/docs/implementation-technical/api/platform-api/admin) | `GET /api/v1/platform/admin/tenants`, `PATCH` |
| `/finance` | [Subscriptions API](/docs/implementation-technical/api/platform-api/subscriptions) | `GET /api/v1/platform/subscriptions/mrr` |
| `/system/queues` | [Queue API](/docs/implementation-technical/api/queue/jobs) | `GET /api/v1/queue/stats`, `POST .../retry` |
| `/admin/secrets` | Vault Admin API | Health, list, rotation, audit endpoints |

## Data Strategy

### Fetching Method

- **Tables (Users/Tenants)**: Server Components with searchParams.
- **Real-time Dashboards (Queues/Logs)**: Client Components with polling (every 5s) or WebSocket.

### Caching

- **Plan Data**: Heavily cached for public-facing pages. Admin updates invalidate cache tags (`plans-list`, `plan-{slug}`).
- **Aggregated Metrics (MRR)**: Cached for 1 hour.
- **Logs**: No caching (live stream).

### Pagination

- **Offset-based** for admin tables (need "Jump to page X").

### Access Control

- **Strict RBAC**:
  - Access guarded by `staff_members` table (distinct from tenant users).
  - Checks on every data fetch (Super Admin vs Support vs Finance).

## Edge Cases & Error Handling

- **Tenant Not Found**: Accessing `/tenants/[id]` with invalid ID → 404 Page.
- **Insufficient Permissions**: Support role accessing Finance dashboard → 403 Forbidden.
- **Queue Action Failures**: If "Retry All Failed Jobs" partially fails, show "Retried 45/50. 5 jobs still stuck."
- **Log Stream Overflow**: If logs exceed buffer, show "Logs truncated. Download full logs?" with export link.

## Component Architecture

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

### Admin-Specific UI Theme (Future / Post-MVP)

**Visual Differences**:
- **Color Scheme**: Dark orange accent (vs. blue for users).
- **Sidebar Header**: "Admin Panel" label.
- **Context Indicator**: Banner showing current admin context.

**Related Documentation**:
- [Design Tokens](/docs/design/design-tokens)
