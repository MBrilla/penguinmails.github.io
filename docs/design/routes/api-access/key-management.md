---
title: "API Key Management Route"
description: "Route specification for creating, viewing, and revoking API keys"
last_modified_date: "2025-11-25"
level: "3"
persona: "Developers"
---

# API Key Management Route

## `/dashboard/settings/developers/keys` - API Key Management

**User Story**: *"As a developer, I want to manage multiple API keys with different permissions, so I can control access for different applications."*

## API Keys List

**Filter Bar**:

- **Search**: Search by key name or ID
- **Status Filter**: All, Active, Revoked
- **Sort**: Created Date, Last Used, Name

**API Key Cards** (Detailed View):

Each card displays:

**Key Name**: "Production App"

- **Key ID**: `pm_live_abc123def456...`
- **Copy Button**: Copy full key to clipboard
- **"Show Key" Toggle**: Reveal full key temporarily (requires re-authentication)

**Status Badge**:

- **Active** (Green): Currently valid
- **Revoked** (Red): No longer valid
- **Expired** (Gray): Past expiration date

**Metadata**:

- **Created**: "November 1, 2025 at 3:45 PM"
- **Created By**: "john@company.com"
- **Last Used**: "2 hours ago"
- **Expires**: "Never" or specific date

**Permissions** (Expandable):

- ☑ **Read Campaigns**: View campaign data
- ☑ **Write Campaigns**: Create and modify campaigns
- ☑ **Read Contacts**: View contact lists
- ☑ **Write Contacts**: Add and update contacts
- ☑ **Send Emails**: Send transactional emails
- ☐ **Admin Access**: Full account access (dangerous)

**Usage Stats**:

- **Total Requests**: "12,345"
- **Last 24 Hours**: "234 requests"
- **Error Rate**: "0.5%"

**Actions**:

- **"Edit Permissions" Button**: Opens permission editor
- **"Rotate Key" Button**: Generates new key, revokes old one
- **"Revoke Key" Button**: Permanently disables key (confirmation required)
- **"View Usage" Link**: Navigate to usage analytics for this key

## Create New API Key

**"+ Create API Key" Button**: Opens modal.

### Key Creation Modal

#### Step 1: Basic Information

- **Key Name**: Input field (e.g., "Production App", "Staging Server")
  - Help text: "Choose a descriptive name to identify this key."
- **Description** (optional): Textarea for notes
- **Environment**: Radio buttons
  - ○ Production (`pm_live_...`)
  - ○ Test (`pm_test_...`)

#### Step 2: Permissions

**Permission Categories** (Expandable sections):

**Campaigns**:
- ☑ Read Campaigns
- ☑ Write Campaigns
- ☐ Delete Campaigns

**Contacts**:
- ☑ Read Contacts
- ☑ Write Contacts
- ☐ Delete Contacts

**Analytics**:
- ☑ Read Analytics
- ☐ Export Data

**Emails**:
- ☑ Send Emails
- ☐ Send Bulk Emails (requires approval)

**Admin** (Dangerous):
- ☐ Full Account Access
- ☐ Manage API Keys
- ☐ Manage Billing

**Preset Templates**:
- **"Read Only" Button**: Select all read permissions
- **"Full Access" Button**: Select all permissions
- **"Email Sending" Button**: Select email-related permissions only

#### Step 3: Expiration (Optional)

- **Never Expire**: Radio button (default)
- **Expire After**: Radio button with date picker
  - Options: 30 days, 90 days, 1 year, Custom date

#### Step 4: IP Restrictions (Optional)

- **Allow All IPs**: Radio button (default)
- **Restrict to IPs**: Radio button with IP input
  - Input: Comma-separated IP addresses or CIDR ranges
  - Example: `203.0.113.0/24, 198.51.100.5`

**"Generate API Key" Button**: Creates key and displays it.

## Key Generated Success

**Success Modal**:

- **Title**: "API Key Created Successfully"
- **Warning**: "Copy this key now. You won't be able to see it again."

**Key Display**:

```text
pm_live_abc123def456ghi789jkl012mno345pqr678stu901vwx234yz
```

**Copy Button**: Copy to clipboard.

**Code Example** (Auto-generated):

```bash
curl -X GET https://api.penguinmails.com/v1/campaigns \
  -H "Authorization: Bearer pm_live_abc123..."
```

**"I've Saved My Key" Button**: Closes modal.

## User Journey Context

Set up once per application/environment, rotate periodically for security.

## Technical Integration

- **Key Generation**: Cryptographically secure random string (64 chars)
- **Storage**: Hashed with bcrypt before storing in database
- **Validation**: Keys validated on every API request

## Related Documentation

- [API Authentication](/docs/features/integrations/api-access#authentication)
- [Security Best Practices](/docs/compliance-security/enterprise/overview)
