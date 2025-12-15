---
title: "Inbox Management"
description: "Email organization, response management, and conversation tracking"
last_modified_date: "2025-12-16"
level: "1"
persona: "All Users"
---

# Inbox Management

Comprehensive email organization, response management, and conversation tracking capabilities.

## Overview

**Status:** Architecture Spike Required (Critical Blocker)
**Timeline:** Q4 2025 (Spike) → Q1 2026 (MVP) → Q2-Q3 2026 (AI & Analytics)
**Priority:** Critical (Foundation feature for response management)

**🚨 Critical Blocker:** Stalwart Mail Server database schema investigation must be completed before any inbox development can proceed. See [Roadmap](/docs/features/inbox/roadmap) for details.

---

## Features in This Category

### Documented (Implementation Pending)

#### [Unified Inbox](/docs/features/inbox/unified-inbox/overview)

Centralized command center for managing responses across all email accounts with AI categorization and real-time sync.

**Key Capabilities**:

- **Universal Aggregation**: Sync emails from Gmail, Outlook, and SMTP in one view

- **AI Intent Detection**: Auto-categorize as Interested, OOO, Not Interested

- **Real-Time Sync**: Bi-directional synchronization with provider mailboxes

- **Team Collaboration**: Assignment, internal notes, and status tracking

- **Advanced Filtering**: Filter by campaign, lead score, and intent

**Status**: Documented | **Priority**: Critical | **Timeline**: Q1 2026

#### [Inbox Rotation](/docs/features/inbox/inbox-rotation/overview)

Maximize deliverability by automatically rotating sending across multiple email accounts with smart health monitoring.

**Key Capabilities**:

- **Rotation Pools**: Group accounts for load balancing

- **Smart Distribution**: Weighted rotation based on health scores

- **Auto-Pause & Recovery**: Protect accounts with high bounce rates

- **Domain Diversity**: Prevent pattern detection

- **Volume Ramping**: Automated warmup schedules

**Status**: Documented | **Priority**: High | **Timeline**: Q1 2026

---

## Current Status

### What's Documented

- ✅ Complete unified inbox architecture with database schemas

- ✅ AI intent detection system design

- ✅ Inbox rotation algorithm with health monitoring

- ✅ Route specifications for inbox UI

- ✅ API endpoint specifications

- ✅ Background job architecture

### Critical Gap: Stalwart Integration

**Problem:** The current documentation proposes custom tables (`inbox_threads`, `inbox_messages`) without investigating how they integrate with Stalwart Mail Server's existing PostgreSQL schema.

## Required Investigation

- Can we extend Stalwart's schema with custom fields?

- Should inbox metadata live in OLTP, Content DB, or Stalwart's database?

- How does Stalwart store emails and can we add foreign keys?

**Impact:** This architectural decision blocks all inbox development. A 3-5 day spike is required to determine the correct approach.

### MVP Gaps (Post-Spike)

Once the Stalwart investigation is complete, the following MVP features need implementation:

1. **Email Threading Algorithm** - Match replies to original messages (1-2 weeks)

2. **Star/Favorite Feature** - Mark important threads (2-3 days)

3. **Folder/View System** - All, Starred, Sent, Archived, Trash views (3-5 days)

4. **Attachment Support** - Document existing Content DB integration (1-2 days)

5. **Browser Notifications** - Integrate with Notifications DB (1-2 days)

**Total MVP Effort:** Unknown until spike complete (estimated 2-4 weeks post-spike)

---

## Roadmap

See the [Inbox Management Roadmap](/docs/features/inbox/roadmap) for:

- **Q4 2025**: Stalwart database schema investigation (3-5 days spike) - **BLOCKER**

- **Q1 2026**: MVP implementation (email threading, star/favorite, folders) - TBD post-spike

- **Q2 2026**: AI features (smart replies, sentiment analysis, advanced analytics)

- **Q3 2026**: International features (multi-language support, offline support)

## Key Insights

- 🚨 **Architecture unknown** - Cannot estimate MVP timeline until Stalwart spike complete

- âš ï¸ **Proposed schema may be wrong** - Documentation needs update based on spike findings

- ✅ **Some infrastructure exists** - Content DB (attachments), Notifications DB (alerts)

- 🎯 **AI features ready** - Gemini AI integration planned for Post-MVP

---

## Related Documentation

### Planning & Design

- [Inbox Management Roadmap](/docs/features/inbox/roadmap) - Detailed timeline and roadmap items

- [Tenant Inbox Routes](/docs/design/routes/tenant-inbox) - UI specifications

- [Inbox API](/docs/implementation-technical/api/tenant-api/inbox) - API endpoints

### Related Features

- [Campaign Management](/docs/features/campaigns/campaign-management/hub) - Campaigns generate inbox responses

- [Lead Management](/docs/features/leads/leads-management) - Lead scoring and context

- [Email Operations](/docs/features/campaigns/email-sequences/overview) - Automated sequences

### Database Architecture

- [Content Database](/docs/implementation-technical/database-infrastructure/content-database/schema-guide) - Email storage and attachments

- [Notifications Database](/docs/implementation-technical/database-infrastructure/notifications-database/schema-guide) - Browser notifications

- [5-Tier Database Architecture](/docs/implementation-technical/database-infrastructure/architecture/5-tier-database-architecture-guide) - Overall architecture

---

[← Back to All Features](https://github.com/penguinmails/penguinmails.github.io/blob/main)
