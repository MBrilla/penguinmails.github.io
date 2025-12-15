---
title: "Developer Overview Route"
description: "Route specification for the developer overview dashboard"
last_modified_date: "2025-11-25"
level: "3"
persona: "Developers"
---

# Developer Overview Route

## `/dashboard/settings/developers` - Developer Overview

**User Story**: *"As a developer, I want to quickly access API keys and documentation, so I can integrate PenguinMails into my application."*

## Quick Start Section

**Getting Started Card**:

- **Title**: "Start Building with PenguinMails API"
- **Steps**:
  1. Generate an API key
  2. Install SDK (optional)
  3. Make your first API call
  4. Set up webhooks (optional)

**"Generate API Key" Button**: Opens key creation modal.

**Quick Links**:

- [View API Documentation →](/dashboard/settings/developers/docs)
- [Explore Code Examples →](/docs/design/routes/#code-examples)
- [Configure Webhooks →](/dashboard/settings/integrations/esp/webhooks)

## Active API Keys

**API Keys Table** (Compact View):

- Columns: Name, Key (masked), Created, Last Used, Actions
- **Key Display**: `pm_live_abc...xyz` (first 8 and last 3 chars visible)
- **Copy Button**: Copy full key to clipboard
- **Actions**: View Details, Revoke

**Example Rows**:

| Name | Key | Created | Last Used | Actions |
|------|-----|---------|-----------|---------|
| Production App | `pm_live_abc...xyz` | Nov 1, 2025 | 2 hours ago | View, Revoke |
| Staging Server | `pm_test_def...uvw` | Oct 15, 2025 | 5 days ago | View, Revoke |

**"+ Create New Key" Button**: Opens key generation modal.

**"Manage All Keys" Link**: Navigate to full key management page.

## Rate Limit Status

**Current Usage Card**:

- **Plan**: "Professional Plan"
- **Rate Limit**: "300 requests/minute"
- **Current Usage**: Progress bar showing "45 / 300 (15%)"
- **Burst Capacity**: "500 requests"
- **Reset Time**: "Resets in 42 seconds"

**Upgrade Prompt** (if approaching limit):

- "Approaching rate limit. [Upgrade plan →](/dashboard/settings/billing) for higher limits."

## API Documentation Quick Access

**Popular Endpoints** (Cards):

**Send Email**:

```http
POST /v1/emails/send
Authorization: Bearer pm_live_...
```

**Manage Contacts**:

```http
POST /v1/contacts
Authorization: Bearer pm_live_...
```

**Get Analytics**:

```http
GET /v1/analytics/campaigns/{id}
Authorization: Bearer pm_live_...
```

## SDK Installation

**Code Snippets** (Tabbed):

**Node.js**:

```bash
npm install @penguinmails/sdk
```

**Python**:

```bash
pip install penguinmails
```

**PHP**:

```bash
composer require penguinmails/sdk
```

**Status**: "Coming Q1 2026" (grayed out for unavailable SDKs).

## User Journey Context

First stop for developers. Quick access to keys and documentation.

## Related Documentation

- [API Access Overview](/docs/features/integrations/api-access)
- [API Reference](/docs/implementation-technical/api/README)
