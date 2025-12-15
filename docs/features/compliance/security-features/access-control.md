---
title: "Access Control"
last_modified_date: "2025-12-15"
level: "1"
persona: "Documentation Users"
---

# Access Control

Authentication and authorization for secure platform access.

## Authentication

- NileDB SDK email/password authentication
- Secure session management
- CSRF protection enabled
- Rate limiting on auth endpoints

## Authorization

- Role-based access control (RBAC)
- Tenant-level isolation
- Workspace-level permissions
- Resource-level permissions

## Roles

| Role | Access Level |
|------|--------------|
| **Platform Admin** | Full system access |
| **Tenant Admin** | Tenant-wide management |
| **Workspace Owner** | Workspace management |
| **Workspace Member** | Limited workspace access |
| **API User** | Programmatic access only |

---

[← Back to Security Features](./README)
