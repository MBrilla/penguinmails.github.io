---
title: "ESP Overview Route"
description: "Route specification for the ESP integration overview dashboard"
last_modified_date: "2025-11-25"
level: "3"
persona: "Developers, Operations Teams"
---

# ESP Overview Route

## `/dashboard/settings/integrations/esp` - ESP Integration Overview

**User Story**: *"As an admin, I want to see all configured ESPs at a glance and quickly add new providers, so I can manage my email delivery infrastructure efficiently."*

## Configured Providers Section

**Provider Cards** (Grid Layout):

Each card displays:

- **Provider Logo**: Postmark, Mailgun, or Built-in SMTP icon
- **Status Badge**:
  - **Active** (Green): Connected and sending
  - **Configured** (Blue): Set up but not actively used
  - **Error** (Red): Connection issue or invalid credentials
  - **Not Configured** (Gray): Available but not set up
- **Quick Stats**:
  - Emails sent today: "1,234"
  - Delivery rate: "99.2%"
  - Last used: "2 hours ago"
- **Actions**:
  - **Configure** button (if not configured)
  - **Test Connection** button
  - **View Settings** link
  - **Disable** toggle

**Available Providers**:

- Postmark (Transactional)
- Mailgun (Marketing)
- Built-in SMTP (Always available)
- SendGrid (Coming Soon - grayed out)
- Amazon SES (Coming Soon - grayed out)

## Quick Actions Bar

- **"+ Add ESP" Button**: Opens provider selection modal
- **"Configure Routing" Button**: Navigate to routing rules
- **"View Analytics" Button**: Navigate to ESP analytics

## Routing Summary

**Current Routing Configuration** (Collapsed by default):

```text
Transactional Emails → Postmark
Marketing Emails → Mailgun
Cold Outreach → Built-in SMTP
Default → Built-in SMTP
```

**"Edit Routing Rules" Link**: Opens routing configuration.

## User Journey Context

First stop for ESP management. Quick overview before diving into specific configurations.

## Related Documentation

- [ESP Integration Overview](/docs/features/integrations/esp-integration)
- [Email Infrastructure Setup](/docs/features/infrastructure/email-infrastructure-setup)
