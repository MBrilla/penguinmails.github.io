---
title: "Profile Management"
last_modified_date: "2025-12-15"
level: "3"
persona: "All Users, Developers"
---

# Profile Management

View and edit profile information and preferences.

## View Profile

User profile information:

```javascript
GET /api/v1/users/me
Authorization: Bearer {access_token}

Response:
{
  "user_id": "user_abc123",
  "email": "user@example.com",
  "name": "John Doe",
  "tenant_id": "tenant_xyz789",
  "role": "owner",
  "email_verified": true,
  "created_at": "2025-11-01T10:00:00Z",
  "preferences": {
    "timezone": "America/Los_Angeles",
    "date_format": "MM/DD/YYYY",
    "email_notifications": true
  }
}
```

## Update Profile

Edit profile information:

```text
Edit Profile

Full Name: John Doe
Email: user@example.com (verified ✓)
Timezone: America/Los_Angeles
Date Format: MM/DD/YYYY

[Save Changes]
```

### API Endpoint

```javascript
PUT /api/v1/users/me
Authorization: Bearer {access_token}

{
  "name": "John Smith",
  "preferences": {
    "timezone": "America/New_York",
    "date_format": "YYYY-MM-DD"
  }
}

Response:
{
  "success": true,
  "user": {
    "name": "John Smith",
    "preferences": {
      "timezone": "America/New_York",
      "date_format": "YYYY-MM-DD"
    }
  }
}
```

### Changing Email

```javascript
// Requires email verification
POST /api/v1/users/me/change-email
{
  "new_email": "newemail@example.com",
  "password": "CurrentPass123!" // Confirm with password
}

Response:
{
  "success": true,
  "email_verification_sent": true,
  "message": "Verify your new email address"
}
```

## User Preferences

Customizable user settings:

```javascript
{
  "preferences": {
    // Regional Settings
    "timezone": "America/Los_Angeles",
    "date_format": "MM/DD/YYYY",
    "time_format": "12h", // 12h or 24h
    "language": "en",

    // Notification Settings
    "email_notifications": true,
    "campaign_alerts": true,
    "weekly_reports": true,
    "billing_alerts": true,

    // Dashboard Settings
    "default_workspace": "ws_abc123",
    "dashboard_layout": "compact",
    "show_onboarding": false
  }
}
```

---

[← Back to User Management](./README)
