---
title: "Team Management Journey"
last_modified_date: "2025-12-15"
level: "2"
persona: "Team Leads, Administrators"
---

# Team Management Journey

Company setup, team invitations, and role assignment workflows.

## Journey Flow

```text
Company Setup → Team Invitation → Role Assignment → Active Collaboration
```

## 1. Company Setup & Onboarding

### Company Information Collection

| Field | Required | Description |
|-------|----------|-------------|
| Company name | Yes | Unique identifier |
| Industry | No | For recommendations |
| Team size | Yes | Affects plan features |
| Workspace name | Yes | URL-friendly identifier |
| Company logo | No | Branding |

### Initial Team Configuration

- **Owner Assignment**: Current user becomes Owner automatically
- **Team Size Confirmation**: Affects plan recommendations and feature limits
- **Optional Team Invites**: Add initial members immediately or skip

### Company Creation Process

1. Company record creation with tenant association
2. RLS policies configured for company isolation
3. Base plan features activated at tenant level
4. Company creation confirmation email sent

## 2. Team Invitation & Onboarding

### Invitation Flow

```text
Team Dashboard → Invite Form → Email Sent → Recipient Journey → Integration
```

### Invitation Form Elements

| Field | Required | Options |
|-------|----------|---------|
| Email address | Yes | - |
| Role selection | Yes | Member, Admin, Owner |
| Personal message | No | Custom invite message |

### Invitation Features

- **Bulk Invites**: CSV upload for multiple invitations
- **Tracking**: Status monitoring (sent, viewed, accepted, expired)
- **Security Token**: Time-limited invitation link (7 days)

### Recipient Journey

1. Email reception with personalized invite
2. Link click directs to acceptance page
3. System checks for existing account
4. New user redirected to signup if needed
5. Existing user gets company association added

## 3. Ongoing Team Management

### Team Dashboard

| Column | Description |
|--------|-------------|
| Name | Team member name |
| Email | Contact email |
| Role | Current role assignment |
| Join Date | When they joined |
| Last Active | Most recent activity |
| Status | Active, pending, inactive |

### Available Actions

- Edit role
- Remove member
- Resend invitation
- Bulk operations for multiple members

### Role Change Process

1. Open role change modal
2. View current role and permissions
3. Select new role from dropdown
4. Preview permission changes
5. Confirm with impact warning
6. Change logged for audit trail

---

[← Back to Organization Analytics](./README)
