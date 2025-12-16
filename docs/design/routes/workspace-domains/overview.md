---
title: "Workspace Domains & Email Accounts Overview"
description: "Manage sending domains and email accounts for the workspace"
last_modified_date: "2025-01-09"
level: "2"
persona: "Technical Teams"
---

# Workspace Domains & Email Accounts Overview

## Purpose & Context

**Goal**: Manage sending domains and email accounts (infrastructure) for the workspace.

**Feature References**:
- [Inbox Rotation](/docs/features/inbox/inbox-rotation/overview)
- [Warm-ups](/docs/features/warmup/email-warmups/overview)
- [Free Mailbox Creation](/docs/features/infrastructure/free-mailbox-creation/overview)

**User Journey**: Setup domains first (Infrastructure) during onboarding, then add email accounts for sending.

---

## UI Patterns & Components

### Core Components

- **DataTable**: For Domains list and Email Accounts list
- **StatusBadge**: For Domain DNS status (Verified/Pending/Failed)
- **CodeBlock**: For displaying DNS records to copy
- **Wizard**: For domain setup flow

### Analytics Patterns

- **DataCard**: Domain health score, Reputation metrics
- **ProgressBar**: Warmup progress visualization

**Layout**: Workspace Context

---

## Route Specifications

| Route | Access | Purpose | State/Data Requirements |
|-------|--------|---------|------------------------|
| `/dashboard/workspaces/[slug]/domains` | Tenant | List Domains | **Server Component**. Table with Domain, DNS Status, Reputation, Accounts Count |
| `/dashboard/workspaces/[slug]/domains/new` | Tenant | Add Domain | **Client Component**. Wizard: Input domain → Show DNS records → Verification |
| `/dashboard/workspaces/[slug]/domains/[id]` | Tenant | Domain Details | **Server Component**. DNS status panel. List of associated email accounts |
| `/dashboard/workspaces/[slug]/domains/[id]/settings` | Tenant | Domain Settings | **Client Component**. DNS records, Tracking settings, DKIM/SPF rotation |
| `/dashboard/workspaces/[slug]/domains/[id]/emails/new` | Tenant | Add Email Account | **Client Component**. Form: Username, SMTP credentials |
| `/dashboard/workspaces/[slug]/domains/[id]/emails/[emailId]` | Tenant | Email Account Details | **Server Component**. Health score, Sending limits, Daily stats |
| `/dashboard/workspaces/[slug]/domains/[id]/emails/[emailId]/warmup` | Tenant | Warmup Management | **Client Component**. Warmup progress graph, Ramp-up settings |

---

## Domain List View

**Route**: `/dashboard/workspaces/[slug]/domains`

**User Story**: *"As a technical admin, I want to monitor the health and DNS status of all my sending domains, so I can ensure deliverability."*

### Data Table

- **Columns**: Domain, DNS Status, Reputation Score, Email Accounts, Actions
- **DNS Status Badge**:
  - ✅ Verified (Green)
  - ⏳ Pending (Yellow)
  - ❌ Failed (Red) with "Check DNS" link

### Reputation Score

Color-coded: 90-100 (Green), 70-89 (Yellow), <70 (Red)

**Tooltip**: "Based on bounce rate, spam complaints, and blacklist status."

### Row Actions

- View Details, Add Email Account, Settings, Delete

### Header Actions

- **"Add Domain" Button**: Primary CTA (top-right)

### Empty State

- Illustration + "Add your first domain to start sending emails" CTA
- Link to domain setup tutorial

---

## Related Sections

- [Domain Setup](/docs/design/routes/workspace-domains/domain-setup) - Add domain wizard and DNS configuration
- [Email Accounts](/docs/design/routes/workspace-domains/email-accounts) - Email account management and warmup
- [API Integration](/docs/design/routes/workspace-domains/api-integration) - Related API endpoints and data strategy
