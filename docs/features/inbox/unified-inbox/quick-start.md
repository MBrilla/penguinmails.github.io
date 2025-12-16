---
title: "Unified Inbox"
description: "Centralized command center for managing responses across all email accounts with AI categorization and real-time sync"
level: "2"
status: "AVAILABLE"
roadmap_timeline: "Q1 2026"
priority: "Critical"
keywords: [unified-inbox, email-management, ai-categorization, response-management]
related_features:
  - inbox/inbox-rotation
  - campaigns/campaign-management/overview
  - leads/lead-scoring
  - integrations/crm-integration/overview
related_tasks:
  - epic-8-inbox-management
---

# Unified Inbox

**Quick Access**: Manage all prospect interactions from a single dashboard with AI-powered categorization, real-time synchronization, and team collaboration tools.

## Overview

The Unified Inbox consolidates emails from thousands of sender accounts into a single, organized interface. Instead of logging into multiple Gmail or Outlook accounts, your team manages all replies in one place. With AI-driven intent detection, it automatically categorizes responses (Interested, Not Interested, OOO) so you can focus on closing deals.

### Key Capabilities

| Capability | Description |
|------------|-------------|
| Universal Aggregation | Sync emails from Gmail, Outlook, SMTP/IMAP, and custom domains |
| AI Intent Detection | Automatically tag responses as Interested, Meeting Booked, etc. |
| Thread Management | View full conversation history across sending accounts |
| Real-Time Sync | Bi-directional synchronization with provider mailboxes |
| Team Collaboration | Assign threads, add internal notes, track response times |
| CRM Sync | Automatically push positive responses to your CRM |

## Quick Start Guide

### Step 1: Access the Inbox

Navigate to **Dashboard → Inbox** to see your consolidated view:

- **All Open** - All threads awaiting response
- **Interested** - Prospects showing interest
- **Meeting Booked** - Confirmed meetings
- **Not Interested** - Declined prospects
- **OOO / Auto-reply** - Automated responses

### Step 2: Handle a Response

1. **Click the thread** to open the conversation view
2. **Review Context** - See prospect details, campaign source, sending account
3. **Reply** - Use the editor or select a template
4. **Update Status** - AI auto-tags or manually adjust

### Step 3: Assign to Team

1. **Add Note** - Click "Add Note" tab for internal comments
2. **Mention** - Use @mentions to notify team members
3. **Assign** - Change owner to route to appropriate person

### Step 4: Bulk Actions

1. Select a view (e.g., "OOO / Auto-reply")
2. Click "Select All"
3. Choose action (Archive, Assign, Tag)
4. Apply to all selected threads

## Inbox Views

| View | Purpose |
|------|---------|
| All Open | Active threads requiring attention |
| Assigned to Me | Threads assigned to current user |
| Interested | High-priority positive responses |
| Meeting Booked | Scheduled meetings |
| Not Interested | Declined or unsubscribed |
| Snoozed | Temporarily hidden threads |
| Archived | Completed conversations |

## Related Documentation

- [Configuration](/docs/features/inbox/unified-inbox/configuration) - AI rules, templates, automation
- [Technical Implementation](/docs/features/inbox/unified-inbox/technical-implementation) - Database schema and API
- [Inbox Rotation](/docs/features/inbox/inbox-rotation) - Sender account rotation
- [Campaign Management](/docs/features/campaigns/campaign-management/overview) - Source campaigns

---

**Last Updated:** November 25, 2025
**Status:** Available - Core Feature
**Target Release:** Q1 2026
**Owner:** Inbox Team
