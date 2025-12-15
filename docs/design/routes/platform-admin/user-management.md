---
title: "User Management Routes"
description: "Admin routes for global user administration and audit trails"
last_modified_date: "2025-12-16"
level: "2"
persona: "Technical Teams"
---

# User Management Routes

## `/dashboard/users` - Global User Management

**User Story**: *"As a support admin, I want to quickly look up any user, view their account details, and review their audit trail to debug issues, so I can resolve tickets faster."*

### Search & Filter

- **Global Search Bar** (Prominent):
  - Placeholder: "Search by email, name, or user ID".
  - **Autocomplete**: Shows results as you type.
- **Filters**:
  - Role: All, Owner, Admin, Member.
  - Status: Active, Suspended, Deleted.
  - Plan: Free, Pro, Enterprise.

### Users Table

- **Columns**: UserID, Name, Email, Plan, Tenant, Status, Created Date, Last Login, Actions.
- **Color Coding**:
  - Suspended users: Red background.
  - Churned users (plan expired): Orange.
- **Row Actions** (Gear icon):
  - **View Details**: Opens modal with full user profile, activity log.
  - **View Audit Trail**: Opens comprehensive audit viewer.
  - **Reset Password**: Sends reset email.
  - **Suspend Account**: Confirmation modal, reason required.
  - **Revoke All Sessions**: Immediate token invalidation (Security).
  - **Change Role**: Promote/demote user (e.g., Member ↔ Admin).
  - **Delete Account** (GDPR): Hard delete, irreversible.

### Audit Trail Viewer

- **Modal or Slide-over** when clicking "View Audit Trail":
  - **Activity Timeline**: Logins, Campaign Creates, Settings Changes, API Calls.
  - **Support Access Log**: Admin views and support ticket references.
  - **Filters**: Date range, action type, IP address.
  - **Search**: Full-text search across all audit entries.
  - **Export**: Download filtered results as CSV.

> [!NOTE]
> **Database Schema**: References audit log table from database documentation.
> **API Endpoint**: Requires executive/platform API endpoint for audit trail retrieval.

**User Journey Context**: Reactive (support-driven). Must be fast and auditable.

**Related Documentation**:
- [Support Processes](/docs/customer-success/support-playbooks/account-lookup)
- [GDPR Compliance](/docs/compliance-security/data-privacy/gdpr)

**Technical Integration**:
- **Audit Trail**: Comprehensive logging of all user actions with retention policy.
- **Search**: Elasticsearch for fast full-text search across millions of users and audit entries.
