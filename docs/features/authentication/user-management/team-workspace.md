---
title: "Team & Workspace Management"
last_modified_date: "2025-12-15"
level: "2"
persona: "All Users, Developers"
---

# Team & Workspace Management

Multi-user tenant support and workspace organization.

## MVP Status

### What's Available Today

- ✅ Team member invitation system
- ✅ Role-based access control (Owner, Admin, Member)
- ✅ View all team members with status
- ✅ Update user roles
- ✅ Remove team members from tenant
- ✅ Workspace assignment during invitation
- ✅ Multi-tenant architecture with complete data isolation

### Missing MVP Features (Q1 2026)

- ⏳ Workspace management feature documentation
- ⏳ Workspace health scoring system
- ⏳ Organization settings & branding documentation
- ⏳ RBAC permission matrix documentation
- ⏳ Team member removal workflow documentation

### Post-MVP Enhancements (2026+)

| Feature | Target |
|---------|--------|
| Advanced permissions system (custom roles) | Q2 2026 |
| Audit logs for team actions | Q3 2026 |
| Team analytics & activity monitoring | Q4 2026 |
| Bulk user management | Q3 2026 |
| User groups & teams within tenant | Q1 2027 |

## Team Management

### Inviting Users

Add team members to tenant:

```text
Invite Team Member

Email Address: _______________
Role: [Admin ▼]
Workspaces: [x] Client A  [ ] Client B

[Send Invitation]
```

### Invitation Flow

1. Admin sends invitation
2. Email sent to invitee
3. Invitee clicks link
4. Creates account or logs in
5. Automatically added to tenant
6. Assigned to selected workspaces

### API Endpoint

```javascript
POST /api/v1/tenants/{tenant_id}/invitations
Authorization: Bearer {access_token}

{
  "email": "newuser@example.com",
  "role": "admin", // owner, admin, member
  "workspaces": ["ws_abc123", "ws_def456"]
}

Response:
{
  "invitation_id": "inv_abc123",
  "email": "newuser@example.com",
  "status": "pending",
  "expires_at": "2025-12-01T10:00:00Z"
}
```

## Managing Team Members

View all team members:

```javascript
GET /api/v1/tenants/{tenant_id}/users
Authorization: Bearer {access_token}

Response:
{
  "users": [
    {
      "user_id": "user_abc123",
      "name": "John Doe",
      "email": "john@example.com",
      "role": "owner",
      "status": "active",
      "last_login": "2025-11-24T10:00:00Z"
    },
    {
      "user_id": "user_def456",
      "name": "Jane Smith",
      "email": "jane@example.com",
      "role": "admin",
      "status": "active",
      "last_login": "2025-11-23T15:30:00Z"
    }
  ],
  "total": 2
}
```

### Update User Role

```javascript
PUT /api/v1/tenants/{tenant_id}/users/{user_id}
{
  "role": "member" // Downgrade from admin
}
```

### Remove User

```javascript
DELETE /api/v1/tenants/{tenant_id}/users/{user_id}

// User removed from tenant
// Loses access to all workspaces
// Data ownership transferred to tenant owner
```

## Workspace Management

Multi-workspace support for agencies:

- Create multiple client workspaces within tenant
- Workspace-level access control (Admin, Member, Viewer)
- Assign team members to specific workspaces
- Workspace health monitoring (0-100 score)
- Isolated campaigns, leads, and settings per workspace

### Workspace Routes

- `/dashboard/workspaces` - List all workspaces with health scores
- `/dashboard/workspaces/new` - Create new workspace
- `/dashboard/workspaces/[slug]/settings` - Workspace settings

**See Also:** [Multi-Tenant Architecture](/docs/features/infrastructure/multi-tenant-architecture)

---

[← Back to User Management](./README)
