---
title: "ESP Provider Configuration Routes"
description: "Route specifications for Postmark and Mailgun ESP configuration"
last_modified_date: "2025-11-25"
level: "3"
persona: "Developers, Operations Teams"
---

# ESP Provider Configuration Routes

## `/dashboard/settings/integrations/esp/postmark` - Postmark Configuration

**User Story**: *"As an admin, I want to configure Postmark for transactional emails, so critical messages like password resets reach users reliably."*

### Connection Status

- **Status Indicator**: Connected (Green) or Disconnected (Red)
- **Last Tested**: "5 minutes ago"
- **Test Connection Button**: Sends test email and validates API token

### API Configuration

**API Token**:

- **Input Field**: Password-masked input
- **"Show" Toggle**: Reveal token temporarily
- **Validation**: Real-time check on blur
- **Help Text**: "Get your Server API Token from Postmark → Servers → API Tokens"
- **Link**: "View Postmark Dashboard" (opens in new tab)

**Default Sender**:

- **Input Field**: Email address (e.g., `noreply@yourdomain.com`)
- **Validation**: Must be verified in Postmark
- **Status**: "Verified" (Green checkmark) or "Unverified" (Warning icon)

### Tracking Options

**Checkboxes**:

- ☑ **Track Opens**: Enable open tracking pixels
- ☑ **Track Links**: Convert links to tracking URLs
- ☐ **Track Delivery**: Receive delivery confirmations (webhook required)

**Note**: "Tracking requires webhook configuration. [Set up webhooks →](/dashboard/settings/integrations/esp/webhooks)"

### Domain Verification

**Verified Domains** (Table):

- Columns: Domain, Status, DKIM, Return-Path, Actions
- **Status**: Verified (Green), Pending (Yellow), Failed (Red)
- **Actions**: "View DNS Records", "Re-verify", "Remove"

**"+ Add Domain" Button**: Opens domain verification wizard.

### Advanced Settings

**Expandable Section**:

- **Message Stream**: Dropdown (Default, Broadcasts, etc.)
- **Batch Size**: Number input (1-500, default: 50)
- **Retry Attempts**: Number input (0-5, default: 3)
- **Timeout**: Seconds (5-60, default: 30)

### Test Email

**Send Test Email Section**:

- **Recipient Email**: Input field
- **"Send Test" Button**: Sends sample email via Postmark
- **Result Display**: Success message with delivery time or error details

**Save Button**: Bottom-right, saves all configuration.

### Technical Integration

- **API**: Postmark REST API v1
- **Validation**: Test API call to `/email` endpoint on save
- **Storage**: API token encrypted at rest in database

### Related Documentation

- [Postmark Integration Guide](/docs/features/integrations/esp-integration#postmark-integration)
- [Domain Verification](/docs/features/domains/domain-management)

---

## `/dashboard/settings/integrations/esp/mailgun` - Mailgun Configuration

**User Story**: *"As an admin, I want to configure Mailgun for marketing campaigns, so I can send high-volume emails with advanced features."*

### Connection Status

- **Status Indicator**: Connected (Green) or Disconnected (Red)
- **Last Tested**: "10 minutes ago"
- **Test Connection Button**: Validates API key and domain

### API Configuration

**API Key**:

- **Input Field**: Password-masked input
- **"Show" Toggle**: Reveal key temporarily
- **Validation**: Real-time check on blur
- **Help Text**: "Get your Private API Key from Mailgun → Settings → API Keys"
- **Link**: "View Mailgun Dashboard" (opens in new tab)

**Domain**:

- **Input Field**: Sending domain (e.g., `mail.yourdomain.com`)
- **Validation**: Must be verified in Mailgun
- **Status**: "Active" (Green) or "Unverified" (Warning)

**Region**:

- **Radio Buttons**:
  - ○ US (api.mailgun.net)
  - ○ EU (api.eu.mailgun.net)
- **Help Text**: "Choose based on your Mailgun account region."

### Tracking Options

**Checkboxes**:

- ☑ **Track Opens**: Enable open tracking
- ☑ **Track Clicks**: Enable click tracking
- ☑ **Automatic Unsubscribe**: Add unsubscribe links automatically
- ☐ **DKIM Signing**: Sign emails with DKIM (requires domain setup)

### Domain Verification

**Verified Domains** (Table):

- Columns: Domain, Status, SPF, DKIM, CNAME, Actions
- **Status**: Verified (Green), Pending (Yellow), Failed (Red)
- **Actions**: "View DNS Records", "Re-verify", "Remove"

**"+ Add Domain" Button**: Opens domain verification wizard with DNS instructions.

### Advanced Settings

**Expandable Section**:

- **Tag Prefix**: Text input (e.g., "pm-" for PenguinMails tags)
- **Batch Size**: Number input (1-1000, default: 100)
- **Retry Attempts**: Number input (0-5, default: 3)
- **Timeout**: Seconds (5-60, default: 30)
- **Testmode**: Toggle (sends to Mailgun's test mode, no actual delivery)

### Email Validation

**List Validation Section**:

- **"Validate Email List" Button**: Opens modal for CSV upload
- **Validation Results**: Shows valid, invalid, risky emails
- **Cost Estimate**: "$0.50 per 1,000 emails validated"

### Test Email

**Send Test Email Section**:

- **Recipient Email**: Input field
- **Tags** (optional): Comma-separated tags
- **"Send Test" Button**: Sends sample email via Mailgun
- **Result Display**: Success with message ID or error details

**Save Button**: Bottom-right, saves all configuration.

### Technical Integration

- **API**: Mailgun REST API v3
- **Validation**: Test API call to `/messages` endpoint on save
- **Storage**: API key encrypted at rest in database

### Related Documentation

- [Mailgun Integration Guide](/docs/features/integrations/esp-integration#mailgun-integration)
- [Email Validation](/docs/features/leads/import-export#email-validation)
