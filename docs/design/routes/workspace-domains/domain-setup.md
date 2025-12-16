---
title: "Domain Setup & Configuration"
description: "Add domain wizard and DNS configuration settings"
last_modified_date: "2025-01-09"
level: "2"
persona: "Technical Teams"
---

# Domain Setup & Configuration

## Add Domain Wizard

**Route**: `/dashboard/workspaces/[slug]/domains/new`

**User Story**: *"As a new user, I want step-by-step instructions to add my domain, so I can start sending emails without technical hurdles."*

### Step 1: Domain Input

- Input field: "Enter your domain (e.g., example.com)"
- Validates format (no http://, no subdomains)
- **Next Button**
- **Smart Subdomain Suggestion**:
  - "We recommend using a subdomain for cold email (e.g., `mail.example.com`)."
  - **"Use Suggestion" Button**: Auto-fills input with `m.` or `mail.` prefix

### Step 2: DNS Records

- **Instructions Panel**: "Add these records to your DNS provider."
- **Code Blocks** (copyable):

```dns
TXT  @  v=spf1 include:penguinmails.com ~all
CNAME mail  mail.penguinmails.com
TXT  _dmarc  v=DMARC1; p=none; rua=mailto:dmarc@penguinmails.com
```

- **Video Tutorial Link**: "Need help? Watch this 2-minute guide."
- **DNS Provider Quick Links**: Shortcuts to common providers (GoDaddy, Cloudflare, Namecheap)

### Step 3: Verification

- **"Verify DNS" Button**: Checks if records are correctly set
- **Progress Indicator**: "Checking... This may take up to 72 hours."
- **Retry Logic**: Can retry verification if initial check fails
- **Success**: "Domain verified! You can now add email accounts."
- **Skip Option**: "Verify Later" (saves domain in pending state)

---

## Domain Details View

**Route**: `/dashboard/workspaces/[slug]/domains/[id]`

**User Story**: *"As a domain owner, I want to see all email accounts under a domain and their health status, so I can manage my sending infrastructure."*

### Domain Header

- Domain name (large text)
- DNS Status badge
- **"Add Email Account" Button**
- **"Settings" Button**

### DNS Records Panel

- Shows currently configured records with status indicators
- **"Re-check DNS" Button** (manual refresh)
- **"View Raw Records" Expandable**: Shows actual DNS query results

### Domain Health Overview

- **Reputation Score**: Large data card with trend indicator
- **Blacklist Status**: "Not blacklisted" or list of blacklists with removal links
- **Bounce Rate**: Last 30 days percentage

### Email Accounts Table

- **Columns**: Email, Health Score, Warmup Status, Daily Sent, Actions
- **Health Score**: 0-100 with color coding
- **Warmup Status**: "Active" (with progress %), "Paused", "Completed"
- **Daily Sent**: "45 / 50" (current/limit)

---

## Domain Configuration

**Route**: `/dashboard/workspaces/[slug]/domains/[id]/settings`

**User Story**: *"As a power user, I want granular control over domain settings and tracking, so I can optimize deliverability."*

### DNS Configuration

- Display of all required DNS records
- Status indicator for each record
- **DKIM Rotation** section:
  - "Rotate DKIM keys every 90 days" toggle
  - "Last rotation: 45 days ago"
  - **"Rotate Now" Button** (manual rotation)

### Tracking Settings

- **Open Tracking**: Toggle (ON/OFF)
- **Click Tracking**: Toggle (ON/OFF)
- **Custom Tracking Domain**: Input field for branded tracking subdomain

### Sending Policies

- **Daily Send Limit Per Account**: Slider (10-500)
- **Warmup Strategy**: Dropdown (Conservative, Moderate, Aggressive)

### Advanced

- **SPF Include**: Shows current SPF record
- **DMARC Policy**: Dropdown (None, Quarantine, Reject)

---

## Related Sections

- [Overview](/docs/design/routes/workspace-domains/overview) - Workspace domains overview
- [Email Accounts](/docs/design/routes/workspace-domains/email-accounts) - Email account management
- [API Integration](/docs/design/routes/workspace-domains/api-integration) - API endpoints and data strategy
