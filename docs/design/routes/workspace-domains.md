---
title: "Workspace Domains & Email Accounts"
description: "Manage sending domains and email accounts for the workspace"
last_modified_date: "2025-12-16"
level: "2"
persona: "Technical Teams"
---

# Workspace Domains & Email Accounts

## 1. Purpose & Context (The "Why")

- **Goal**: Manage sending domains and email accounts (infrastructure) for the workspace.

- **Feature References**:

  - [Inbox Rotation](/docs/features/inbox/inbox-rotation/overview)

  - [Warm-ups](/docs/features/warmup/email-warmups/overview)

  - [Free Mailbox Creation](/docs/features/infrastructure/free-mailbox-creation/overview)

- **User Journey**: Setup domains first (Infrastructure) during onboarding, then add email accounts for sending.

## 2. UI Patterns & Components (The "How")

- **Core Components**:

  - `DataTable`: For Domains list and Email Accounts list.

  - `StatusBadge`: For Domain DNS status (Verified/Pending/Failed).

  - `CodeBlock`: For displaying DNS records to copy.

  - `Wizard`: For domain setup flow.

- **Analytics Patterns**:

  - `DataCard`: Domain health score, Reputation metrics.

  - `ProgressBar`: Warmup progress visualization.

- **Layout**: Workspace Context.

## 3. Route Specifications

| Route | Access | Purpose | State/Data Requirements |
|---|---|---|---|
| `/dashboard/workspaces/[slug]/domains` | Tenant | List Domains | **Server Component**. Table. Columns: Domain, DNS Status, Reputation, Accounts Count. |
| `/dashboard/workspaces/[slug]/domains/new` | Tenant | Add Domain | **Client Component**. Wizard: Input domain → Show DNS records → Verification. |
| `/dashboard/workspaces/[slug]/domains/[id]` | Tenant | Domain Details | **Server Component**. DNS status panel. List of associated email accounts. |
| `/dashboard/workspaces/[slug]/domains/[id]/settings` | Tenant | Domain Settings | **Client Component**. DNS records, Tracking settings, DKIM/SPF rotation. |
| `/dashboard/workspaces/[slug]/domains/[id]/emails/new` | Tenant | Add Email Account | **Client Component**. Form: Username, SMTP credentials. |
| `/dashboard/workspaces/[slug]/domains/[id]/emails/[emailId]` | Tenant | Email Account Details | **Server Component**. Health score, Sending limits, Daily stats. |
| `/dashboard/workspaces/[slug]/domains/[id]/emails/[emailId]/warmup` | Tenant | Warmup Management | **Client Component**. Warmup progress graph, Ramp-up settings. |

## 4. Detailed View Descriptions

### `/dashboard/workspaces/[slug]/domains` - Domain List

**User Story**: *"As a technical admin, I want to monitor the health and DNS status of all my sending domains, so I can ensure deliverability."*

**What You'll Find**:

- **Data Table**:

  - Columns: Domain, DNS Status, Reputation Score, Email Accounts, Actions.

  - **DNS Status Badge**:

    - ✅ Verified (Green)

    - ⏳ Pending (Yellow)

    - ❌ Failed (Red) with "Check DNS" link.

- **Reputation Score** (see [Analytics UI Patterns](/docs/design/analytics-ui-patterns#data-cards)):

  - Color-coded: 90-100 (Green), 70-89 (Yellow), <70 (Red).

  - Tooltip: "Based on bounce rate, spam complaints, and blacklist status."

- **Row Actions**: View Details, Add Email Account, Settings, Delete.

- **Header Actions**:

  - **"Add Domain" Button**: Primary CTA (top-right).

- **Empty State**:

  - Illustration + "Add your first domain to start sending emails" CTA.

  - Link to domain setup tutorial.

**User Journey Context**: Infrequent access for setup and troubleshooting.

**Related Documentation**:

- [Domain Setup Guide](/docs/features/warmup/email-warmups/overview)

- [DNS Configuration](/docs/technical/infrastructure/email-deliverability)

**Technical Integration**:

- **Real-time DNS Checks**: Button triggers backend DNS lookup (via DNS resolver API).

- **Blacklist Monitoring**: Daily cron job checks against known blacklists.

- **Reputation Calculation**: Aggregated from bounce rates, spam complaints, and blacklist status.

---

### `/dashboard/workspaces/[slug]/domains/new` - Add Domain Wizard

**User Story**: *"As a new user, I want step-by-step instructions to add my domain, so I can start sending emails without technical hurdles."*

**What You'll Find**:

- **Step 1: Domain Input**

  - Input field: "Enter your domain (e.g., example.com)".

  - Validates format (no http://, no subdomains).

  - **Next Button**.
  - **Smart Subdomain Suggestion**:
    - "We recommend using a subdomain for cold email (e.g., `mail.example.com`)."
    - **"Use Suggestion" Button**: Auto-fills input with `m.` or `mail.` prefix.

- **Step 2: DNS Records**

  - **Instructions Panel**: "Add these records to your DNS provider."

  - **Code Blocks** (copyable):

    ```dns
    TXT  @  v=spf1 include:penguinmails.com ~all
    CNAME mail  mail.penguinmails.com
    TXT  _dmarc  v=DMARC1; p=none; rua=mailto:dmarc@penguinmails.com
    ```

  - **Video Tutorial Link**: "Need help? Watch this 2-minute guide."

  - **DNS Provider Quick Links**: Shortcuts to common providers (GoDaddy, Cloudflare, Namecheap).

- **Step 3: Verification**

  - **"Verify DNS" Button**: Checks if records are correctly set.

  - **Progress Indicator**: "Checking... This may take up to 72 hours."

  - **Retry Logic**: Can retry verification if initial check fails.

  - **Success**: "Domain verified! You can now add email accounts."

  - **Skip Option**: "Verify Later" (saves domain in pending state).

**User Journey Context**: Critical setup step. Must reduce support tickets via clear instructions.

**Related Documentation**:

- [Domain Setup Tutorial](/docs/operations/freelancer-management/support/tutorials/domain-setup)

- [DNS Provider Guides](/docs/technical/infrastructure/dns-providers)

**Technical Integration**:

- **DNS Validation**: Backend queries DNS records using DNS resolver.

- **Async Verification**: Polling mechanism checks DNS propagation status.

---

### `/dashboard/workspaces/[slug]/domains/[id]` - Domain Details

**User Story**: *"As a domain owner, I want to see all email accounts under a domain and their health status, so I can manage my sending infrastructure."*

**What You'll Find**:

- **Domain Header**:

  - Domain name (large text).

  - DNS Status badge.

  - **"Add Email Account" Button**.

  - **"Settings" Button**.

- **DNS Records Panel**:

  - Shows currently configured records with status indicators.

  - **"Re-check DNS" Button** (manual refresh).

  - **"View Raw Records" Expandable**: Shows actual DNS query results.

- **Domain Health Overview**:

  - **Reputation Score**: Large data card with trend indicator.

  - **Blacklist Status**: "Not blacklisted" or list of blacklists with removal links.

  - **Bounce Rate**: Last 30 days percentage.

- **Email Accounts Table**:

  - Columns: Email, Health Score, Warmup Status, Daily Sent, Actions.

  - **Health Score**: 0-100 with color coding.

  - **Warmup Status**: "Active" (with progress %), "Paused", "Completed".

  - **Daily Sent**: "45 / 50" (current/limit).

- **Actions Per Account**: View Details, Pause Warmup, Settings, Delete.

**User Journey Context**: Dashboard for domain-level management.

**Related Documentation**:

- [Email Account Health](/docs/features/warmup/email-warmups/overview)

- [Inbox Rotation Strategy](/docs/features/inbox/inbox-rotation/overview)

**Technical Integration**:

- **Server Component**: Fetches domain details and email accounts.

- **Real-time DNS Check**: Client-side button triggers API call for live DNS validation.

---

### `/dashboard/workspaces/[slug]/domains/[id]/settings` - Domain Configuration

**User Story**: *"As a power user, I want granular control over domain settings and tracking, so I can optimize deliverability."*

**What You'll Find**:

- **DNS Configuration**:

  - Display of all required DNS records.

  - Status indicator for each record.

  - **DKIM Rotation** section:

    - "Rotate DKIM keys every 90 days" toggle.

    - "Last rotation: 45 days ago".

    - **"Rotate Now" Button** (manual rotation).

- **Tracking Settings**:

  - **Open Tracking**: Toggle (ON/OFF).

  - **Click Tracking**: Toggle (ON/OFF).

  - **Custom Tracking Domain**: Input field for branded tracking subdomain.

- **Sending Policies**:

  - **Daily Send Limit Per Account**: Slider (10-500).

  - **Warmup Strategy**: Dropdown (Conservative, Moderate, Aggressive).

- **Advanced**:

  - **SPF Include**: Shows current SPF record.

  - **DMARC Policy**: Dropdown (None, Quarantine, Reject).

- **Save Button**: Bottom-right.

**User Journey Context**: Advanced configuration. Accessed during setup or optimization.

**Related Documentation**:

- [DNS Best Practices](/docs/technical/infrastructure/dns-providers)

- [DKIM Rotation](/docs/technical/infrastructure/email-deliverability)

---

### `/dashboard/workspaces/[slug]/domains/[id]/emails/new` - Add Email Account

**User Story**: *"As a user, I want to quickly connect an email account to start sending campaigns, with options for both OAuth and manual credentials."*

**What You'll Find**:

- **Connection Method**:

  - **Manual SMTP Credentials** (Current implementation)

    - Email/Username field.

    - Password field (encrypted icon, password input type).

    - SMTP Server (auto-filled for common providers like Gmail, Outlook).

    - SMTP Port (default: 587 for TLS).

> [!NOTE]
> **Future OAuth Support**: OAuth connections for Gmail/Outlook accounts are planned for **Q4 2026 post-MVP**. Current implementation uses SMTP credentials only.

- **IMAP Settings** (for reply tracking):

  - IMAP Server (auto-filled).

  - IMAP Port (default: 993 for SSL).

  - Same credentials as SMTP (checkbox).

- **Advanced Settings** (Collapsible):

  - Custom sending limits (overrides domain default).

  - Signature text.

- **Save Button**: "Add Account & Start Warmup".

**User Journey Context**: Onboarding or expansion. Must be secure and simple.

**Related Documentation**:

- [Free Mailbox Creation](/docs/features/infrastructure/free-mailbox-creation/overview)

- [Security Best Practices](/docs/compliance-security/enterprise/overview)

**Technical Integration**:

- **Authentication**: SMTP credentials encrypted at rest using AES-256.

- **Connection Testing**: Validates SMTP/IMAP credentials before saving (sends test email).

- **Auto-warmup**: Newly added account automatically enters warmup sequence.

> [!NOTE]
> Future OAuth integration will use industry-standard OAuth 2.0 flows for seamless Gmail/Outlook connections (Q4 2026 post-MVP).

---

### `/dashboard/workspaces/[slug]/domains/[id]/emails/[emailId]` - Email Account Details

**User Story**: *"As a user managing multiple email accounts, I want to monitor each account's health and sending activity, so I can identify issues quickly."*

**What You'll Find**:

- **Account Header**:

  - Email address (large text).

  - Health Score badge (with color).

  - Status (Active, Paused, Disabled).

- **Health Metrics**:

  - **Health Score**: 0-100 with breakdown:

    - Bounce rate

    - Spam complaint rate

    - Engagement rate

  - **Trend Chart**: Health score over last 30 days.

- **Sending Activity**:

  - **Today**: "45 sent / 50 limit".

  - **This Week**: Total sent.

  - **This Month**: Total sent.

  - **Activity Chart**: Daily sends for last 30 days.

- **Warmup Status** (if active):

  - Progress bar: "Day 15 / 30".

  - Current daily limit: "50 emails/day".

  - Next milestone: "Day 20 → 75 emails/day".

  - **Manage Warmup** button (links to warmup view).

- **Recent Activity Log**:

  - Table of recent sends with timestamps, recipients, campaign names, status.

- **Actions**:

  - **Settings** button.

  - **Pause** button (pauses sending).

  - **Delete** button (with confirmation).

**User Journey Context**: Monitoring and troubleshooting individual accounts.

**Related Documentation**:

- [Email Account Health](/docs/features/warmup/email-warmups/overview)

- [Deliverability Monitoring](/docs/operations/monitoring/email-health)

---

### `/dashboard/workspaces/[slug]/domains/[id]/emails/[emailId]/warmup` - Warmup Management

**User Story**: *"As a user, I want to rely on automated warmup to build reputation without manual intervention, while retaining the ability to pause or adjust if needed."*

**What You'll Find**:

- **Warmup Progress**:

  - **Progress Graph**: Shows daily send volume ramping up over time.

  - **Current Phase**: "Day 15 of Conservative Warmup".

  - **Daily Limit**: "Sending 50/day (target: 200/day by Day 30)".

- **Warmup Schedule**:

  - Table showing upcoming milestones:

    - Day 20: Increase to 75/day

    - Day 25: Increase to 125/day

    - Day 30: Increase to 200/day

- **Settings**:

  - **Warmup Strategy**: Dropdown (Conservative, Moderate, Aggressive).

    - Conservative: 30-day ramp.

    - Moderate: 20-day ramp.

    - Aggressive: 10-day ramp.

  - **Target Daily Volume**: Input field (max based on plan).

- **Actions**:

  - **Pause Warmup**: Maintains current limits without progression.

  - **Resume Warmup**: Continues from current phase.

  - **Skip Warmup**: Immediately set to target volume (⚠️ not recommended).

- **Health Monitoring**:

  - Real-time health score during warmup.

  - **Alert**: "Health score dropped below 80. Warmup paused automatically. Review recent sends."

**User Journey Context**: Active monitoring during critical warmup period.

**Related Documentation**:

- [Warmup Strategies](/docs/features/warmup/email-warmups/overview)

- [Best Practices](/docs/business/email-deliverability/warmup-guide)

**Technical Integration**:

- **Automated Progression**: Background job checks daily and adjusts limits.

- **Health-based Pausing**: If health score drops, warmup automatically pauses.

- **Email Distribution**: Warmup emails distributed across time windows for natural patterns.

## 5. Related API Endpoints

| Route | Related API | Description |
|---|---|---|
| `/domains` | [Domains API](/docs/implementation-technical/api/tenant-smtp/domains) | `GET /api/v1/tenant/domains` (List), `POST` (Add). |
| `/domains/[id]` | [Domains API](/docs/implementation-technical/api/tenant-smtp/domains) | `GET /api/v1/tenant/domains/{id}` (Details & DNS Status). |
| `/domains/[id]/emails` | [Domains API](/docs/implementation-technical/api/tenant-smtp/domains) | `POST /api/v1/tenant/domains/{id}/accounts` (Add Email). |
| `/domains` (Health) | [Reputation API](/docs/implementation-technical/api/central-smtp/reputation) | `GET /api/v1/reputation/domains/{domain}` (Health Score). |

## 6. Data Strategy

- **Fetching Method**:

  - **Domain List**: Server Component.

  - **DNS Status**: Hybrid. Initial load via Server Component, "Re-check" button triggers Client-side fetch.

- **Caching**:

  - **Domain List**: Cached for 1 minute.

  - **Reputation Scores**: Cached for 24 hours (updated via nightly job).

- **Optimistic Updates**:

  - "Delete Domain" removes row immediately.

- **Streaming**:

  - **DNS Verification**: When adding a domain, the verification step uses a streaming response or polling to show progress of record checks.

## 7. Edge Cases & Error Handling

- **DNS Propagation Delay**: Explicitly warn users that verification can take 24-48h.

- **Domain Blacklisted**: Show critical alert with specific blacklist provider and delisting link.

- **SMTP Connection Fail**: During email account setup, if credentials fail, show specific error (e.g., "Authentication Failed" vs "Timeout").

- **Rate Limited**: If sending limit reached, account status changes to "Rate Limited" until next window.

## 8. Component Architecture

### Page Components

- **`DomainList`** (Server)

  - Props: `domains: Domain[]`

  - Dependencies: `HealthScoreRing`

- **`DNSStatusTable`** (Client)

  - Props: `records: DNSRecord[]`

  - Features: Polling for verification status, copy-to-clipboard buttons.

- **`InstructionModal`** (Client)

  - Props: `provider: 'GoDaddy' | 'Namecheap' | 'Cloudflare'`

  - Content: Provider-specific step-by-step guides.

### Shared Components

- **`HealthScoreRing`**: SVG circular progress with color coding (Green/Yellow/Red).

- **`CopyButton`**: Reused globally for API keys and DNS values.
