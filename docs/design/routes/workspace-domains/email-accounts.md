---
title: "Email Accounts & Warmup Management"
description: "Email account setup, monitoring, and warmup configuration"
last_modified_date: "2025-01-09"
level: "2"
persona: "Technical Teams"
---

# Email Accounts & Warmup Management

## Add Email Account

**Route**: `/dashboard/workspaces/[slug]/domains/[id]/emails/new`

**User Story**: *"As a user, I want to quickly connect an email account to start sending campaigns, with options for both OAuth and manual credentials."*

### Connection Method

**Manual SMTP Credentials** (Current implementation):
- Email/Username field
- Password field (encrypted icon, password input type)
- SMTP Server (auto-filled for common providers like Gmail, Outlook)
- SMTP Port (default: 587 for TLS)

> **Note**: OAuth connections for Gmail/Outlook accounts are planned for **Q4 2026 post-MVP**. Current implementation uses SMTP credentials only.

### IMAP Settings (for reply tracking)

- IMAP Server (auto-filled)
- IMAP Port (default: 993 for SSL)
- Same credentials as SMTP (checkbox)

### Advanced Settings (Collapsible)

- Custom sending limits (overrides domain default)
- Signature text

**Save Button**: "Add Account & Start Warmup"

---

## Email Account Details

**Route**: `/dashboard/workspaces/[slug]/domains/[id]/emails/[emailId]`

**User Story**: *"As a user managing multiple email accounts, I want to monitor each account's health and sending activity, so I can identify issues quickly."*

### Account Header

- Email address (large text)
- Health Score badge (with color)
- Status (Active, Paused, Disabled)

### Health Metrics

- **Health Score**: 0-100 with breakdown (Bounce rate, Spam complaint rate, Engagement rate)
- **Trend Chart**: Health score over last 30 days

### Sending Activity

- **Today**: "45 sent / 50 limit"
- **This Week**: Total sent
- **This Month**: Total sent
- **Activity Chart**: Daily sends for last 30 days

### Warmup Status (if active)

- Progress bar: "Day 15 / 30"
- Current daily limit: "50 emails/day"
- Next milestone: "Day 20 → 75 emails/day"
- **Manage Warmup** button (links to warmup view)

### Recent Activity Log

- Table of recent sends with timestamps, recipients, campaign names, status

### Actions

- **Settings** button
- **Pause** button (pauses sending)
- **Delete** button (with confirmation)

---

## Warmup Management

**Route**: `/dashboard/workspaces/[slug]/domains/[id]/emails/[emailId]/warmup`

**User Story**: *"As a user, I want to rely on automated warmup to build reputation without manual intervention, while retaining the ability to pause or adjust if needed."*

### Warmup Progress

- **Progress Graph**: Shows daily send volume ramping up over time
- **Current Phase**: "Day 15 of Conservative Warmup"
- **Daily Limit**: "Sending 50/day (target: 200/day by Day 30)"

### Warmup Schedule

Table showing upcoming milestones:
- Day 20: Increase to 75/day
- Day 25: Increase to 125/day
- Day 30: Increase to 200/day

### Settings

- **Warmup Strategy**: Dropdown (Conservative, Moderate, Aggressive)
  - Conservative: 30-day ramp
  - Moderate: 20-day ramp
  - Aggressive: 10-day ramp
- **Target Daily Volume**: Input field (max based on plan)

### Actions

- **Pause Warmup**: Maintains current limits without progression
- **Resume Warmup**: Continues from current phase
- **Skip Warmup**: Immediately set to target volume (⚠️ not recommended)

### Health Monitoring

- Real-time health score during warmup
- **Alert**: "Health score dropped below 80. Warmup paused automatically. Review recent sends."

---

## Related Sections

- [Overview](/docs/design/routes/workspace-domains/overview) - Workspace domains overview
- [Domain Setup](/docs/design/routes/workspace-domains/domain-setup) - Domain wizard and configuration
- [API Integration](/docs/design/routes/workspace-domains/api-integration) - API endpoints and data strategy
