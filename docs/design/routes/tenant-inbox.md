---
title: "Tenant Unified Inbox"
description: "Manage all replies from all campaigns and email accounts in a single interface"
last_modified_date: "2025-12-16"
level: "2"
persona: "Technical Teams"
---

# Tenant Unified Inbox

## 1. Purpose & Context (The "Why")

- **Goal**: Manage all replies from all campaigns and email accounts in a single interface.

- **Feature References**:

  - [Unified Inbox](/docs/features/inbox/unified-inbox/overview)

- **User Journey**: Daily workflow: Check Inbox → Reply/Archive/Mark Interested → Zero Inbox.

## 2. UI Patterns & Components (The "How")

- **Core Components**:

  - `ThreePaneLayout`: 1. Folders/Filters, 2. Thread List, 3. Conversation View.

  - `ThreadList`: Virtualized list for performance.

  - `ConversationView`: Chat-like interface for email threads.

  - `ReplyComposer`: Rich text editor with template insertion.

- **Analytics Patterns**: N/A.

- **Layout**: Global Context (Full Height).

## 3. Route Specifications

| Route | Access | Purpose | State/Data Requirements |
|---|---|---|---|
| `/dashboard/inbox` | Tenant | Inbox Root | **Client Component**. Fetches initial threads. Subscribes to real-time updates (WebSocket/Polling) for new messages. |
| `/dashboard/inbox/[threadId]` | Tenant | Thread View | **Client Component**. Loads specific thread. Optimistic updates for "Mark Read" / "Archive". |

### Key Interactions

- **Filtering**: Filter by "Unread", "Interested", "Campaign", "Workspace".

- **Quick Actions**: Keyboard shortcuts for Archive (E), Reply (R), Mark Spam (!).

## 4. Detailed View Descriptions

### `/dashboard/inbox` - Unified Inbox

**User Story**: *"As a busy entrepreneur, I want all campaign replies in one inbox with smart categorization, so I can respond quickly without switching between accounts."*

**What You'll Find**:

**Three-Pane Layout**:

1. **Left Panel: Folders & Filters** (Collapsible)

   - **Folders**:

     - 📥 All Mail

     - 🔥 Unread (badge with count)

     - ⭐ Interested

     - 📂 Archived

   - **Filters**:

     - By Campaign: Dropdown list of active campaigns.

     - By Workspace: For agency users.

     - By Status: "Needs Reply", "Auto-reply", "Bounced".

2. **Middle Panel: Thread List**

   - **Each Thread Item Shows**:

     - **Lead Name** (bold if unread).

     - **Subject Line** (truncated).

     - **Snippet**: First line of last message.

     - **Timestamp**: "2 hours ago" or "Oct 15".

     - **Icons**: 💬 (has reply), 📎 (attachment), ⭐ (interested).

   - **Virtualized Scrolling**: Smooth performance with 1000+ threads.

   - **Select Multiple**: Checkbox for bulk actions.

3. **Right Panel: Conversation View**

   - **Thread Header**:

     - Lead name + email.

     - Related campaign link.

     - **Action Buttons**: Mark Interested, Archive, Delete, More (⋯).

   - **Message Thread** (Chat-style):

     - Sent messages (right-aligned, blue).

     - Received messages (left-aligned, gray).

     - Timestamps between messages.

   - **Reply Composer** (Bottom):

     - Rich text editor.

     - **"Insert Template" Dropdown**: Pre-written responses.

     - **Attachment Button**.

     - **Send Button** (Ctrl+Enter shortcut).

**User Journey Context**: Daily hub for managing prospect interactions. Must be fast and keyboard-friendly.

**Related Documentation**:

- [Unified Inbox Overview](/docs/features/inbox/unified-inbox/overview)

- [Response Best Practices](/docs/business/messaging/reply-strategies)

**Technical Integration**:

- **Real-time Updates**:

  - **WebSocket Connection**: Receives new message events.

  - **Fallback Polling**: Every 30s if WebSocket fails.

- **Optimistic UI**: Replies appear immediately, synced to backend asynchronously.

- **Thread Caching**: Recently viewed threads cached in IndexedDB for offline access.

---

### `/dashboard/inbox/[threadId]` - Thread Detail View

**User Story**: *"As a user who clicked a thread from elsewhere (e.g., email notification), I want to land directly in the thread view, so I can respond immediately."*

**What You'll Find**:

- **Same Three-Pane Layout** as root inbox.

- **Thread Automatically Selected** in middle panel.

- **Right Panel**: Shows full conversation for selected thread.

**Behavior**:

- If accessed from external link (e.g., email notification), inbox loads with this thread pre-selected.

- **Mark as Read**: Automatically triggered when thread opens.

**User Journey Context**: Deep-link entry point. Common from notification emails.

**Related Documentation**:

- [Email Notifications](/docs/operations/notifications/email-alerts)

---

### Keyboard Shortcuts (All Inbox Views)

**User Story**: *"As a power user processing 100+ emails daily, I want keyboard shortcuts, so I can manage my inbox without touching the mouse."*

**Shortcuts**:

- **`j` / `k`**: Navigate down/up thread list.

- **`Enter`**: Open selected thread.

- **`r`**: Reply to thread.

- **`e`**: Archive thread.

- **`!`**: Mark as spam.

- **`*` + `a`**: Select all threads.

- **`Esc`**: Close conversation view (return to list).

**Implementation**:

- **Keyboard event listeners** on inbox component.

- **Visual indicators**: Shortcuts shown in tooltips.

**Related Documentation**:

- [Keyboard Navigation Patterns](/docs/design/ui-library#accessibility)

---

### Filters & Search

**User Story**: *"As a user with hundreds of threads, I want powerful search and filtering, so I can find specific conversations quickly."*

**Search Bar** (Top of Middle Panel):

- **Placeholder**: "Search by name, email, or subject".

- **Real-time Search**: Updates results as you type.

- **Advanced Filters** (Dropdown):

  - Date range: "Last 7 days", "Last 30 days", "Custom".

  - Has attachment: Yes/No.

  - Campaign: Select from list.

**Filter Persistence**:

- Active filters saved in URL query params.

- **"Clear Filters" Button** appears when active.

**Related Documentation**:

- [Search Best Practices](/docs/design/ui-library#search)

---

### Reply Management

**User Story**: *"As a salesperson, I want template-based replies with personalization, so I can respond professionally without retyping common messages."*

**Reply Composer Features**:

- **Template Insertion**:

  - Dropdown button: "Insert Template".

  - Shows categorized templates (Follow-up, Meeting Request, etc.).

  - Click to insert, automatically personalizes with lead data.

- **Signature**:

  - Auto-appends user signature (configurable in settings).

- **Tracking**:

  - "Track opens" toggle (default ON).

- **Scheduled Send** (Advanced):

  - "Send Later" option.

  - Time picker for delayed sending.

**Validation**:

- **Empty Subject Warning**: "Subject line is empty. Continue?"

- **No Recipient Error**: Cannot send without valid recipient.

**Related Documentation**:

- [Email Templates](/docs/design/ui-library#templates)

- [Reply Tracking](/docs/technical/email-tracking/overview)

---

### Real-time Notifications

**User Story**: *"As a user with inbox open all day, I want non-intrusive notifications for new replies, so I can respond immediately."*

**Notification Behavior**:

- **New Message Toast** (Top-right):

  - Shows sender name + subject.

  - Click to open thread.

  - Auto-dismiss after 5 seconds.

- **Browser Notification** (if tab inactive):

  - Requires user permission.

  - Shows same info as toast.

- **Sound Alert** (Optional):

  - Subtle "ding" sound.

  - Configurable in settings (ON by default).

**User Preference Controls**:

- Toggle notifications in `/dashboard/settings/profile`.

**Related Documentation**:

- [Notification Settings](/docs/operations/notifications/preferences)

- [Browser Notifications API](/docs/design/ui-library#notifications)

## 5. Related API Endpoints

| Route | Related API | Description |
|---|---|---|
| `/inbox` | [Inbox API](/docs/implementation-technical/api/tenant-api/inbox) | `GET /api/v1/tenant/inbox/threads` (List threads). |
| `/inbox/[threadId]` | [Inbox API](/docs/implementation-technical/api/tenant-api/inbox) | `GET /api/v1/tenant/inbox/threads/{id}` (Thread details). |
| `/inbox/[threadId]/reply` | [Inbox API](/docs/implementation-technical/api/tenant-api/inbox) | `POST /api/v1/tenant/inbox/threads/{id}/reply` (Send reply). |
| `/inbox/ws` | [Inbox API](/docs/implementation-technical/api/tenant-api/inbox) | WebSocket endpoint for real-time updates. |

## 6. Data Strategy

- **Fetching Method**:

  - **Initial Load**: Client-side fetch (`useSWR` or `react-query`) to populate thread list.

  - **Real-time**: WebSocket connection pushes `NEW_MESSAGE` and `THREAD_UPDATED` events.

- **Caching**:

  - **Thread List**: Cached in memory (SWR cache).

  - **Offline Support**: `IndexedDB` stores last 50 threads for offline viewing.

- **Optimistic Updates**:

  - **Sending Reply**: Message appears in chat immediately with "Sending..." status.

  - **Archive/Delete**: Thread removed from list immediately.

- **State Management**:

  - **Unread Counts**: Global context updated via WebSocket events.

## 7. Edge Cases & Error Handling

- **WebSocket Disconnect**: Show "Reconnecting..." badge. Fallback to polling.

- **Send Failure**: If reply fails, show red "Failed to send" icon with "Retry" button.

- **Thread Not Found**: If accessing deleted thread via URL, redirect to Inbox root.

- **Concurrent Reply**: If another user replies to same thread, show new message immediately to avoid double-reply.

## 8. Component Architecture

### Page Components

- **`InboxLayout`** (Client)

  - Manages split-pane layout (ThreadList | ChatWindow).

  - Handles WebSocket connection context.

- **`ThreadList`** (Client)

  - Props: `initialThreads: Thread[]`

  - Features: Virtualized list (react-window), search filter, unread indicators.

- **`ChatWindow`** (Client)

  - Props: `activeThreadId: string`

  - Sub-components: `MessageBubble`, `ReplyComposer`, `LeadContextSidebar`

### Shared Components

- **`ReplyComposer`**: Rich text input with "Insert Template" and "AI Assist" actions.

- **`LeadContextSidebar`**: Reused from Lead Profile Drawer.

---

## 9. Implementation Status & MVP Gaps

### Current Status

**Documentation**: Complete route specifications with comprehensive UI patterns and component architecture.

**Implementation**: Pending architecture spike completion.

### Critical Blocker: Stalwart Integration

**Problem**: The inbox architecture proposes custom tables (`inbox_threads`, `inbox_messages`) without investigating how they integrate with Stalwart Mail Server's existing PostgreSQL schema.

**Required Investigation** (Q4 2025 - 3-5 days):

- Can we extend Stalwart's schema with custom fields?

- Should inbox metadata live in OLTP, Content DB, or Stalwart's database?

- How does Stalwart store emails and can we add foreign keys?

**Impact**: This architectural decision blocks all inbox development.

### MVP Gaps (Q1 2026 - Post-Spike)

Once the Stalwart investigation is complete, the following features need implementation:

1. **Email Threading Algorithm** (1-2 weeks)

   - Match replies using RFC 5322 headers (References, In-Reply-To)

   - Fallback to Subject + Contact matching

   - Handle edge cases (forwarded emails, subject changes)

2. **Star/Favorite Feature** (2-3 days)

   - Add star icon to thread list UI

   - Create "Starred" folder view

   - Implement star/unstar API endpoint

3. **Folder/View System** (3-5 days)

   - Implement All, Starred, Sent, Archived, Trash views

   - Add "Archive" and "Delete" actions with bulk support

   - Implement soft delete (trash folder, permanent delete after 30 days)

4. **Attachment Support** (1-2 days documentation)

   - Document integration with existing Content DB attachments table

   - Add file upload UI specifications

   - Document file size limits (25 MB per attachment)

5. **Browser Notifications** (1-2 days integration)

   - Integrate with existing Notifications Database

   - Request browser notification permission on first inbox visit

   - Create notification when new reply arrives

**Total MVP Effort**: Unknown until Stalwart spike complete (estimated 2-4 weeks post-spike)

### Post-MVP Enhancements (Q2-Q3 2026)

### Q2 2026: AI Features

- Smart Reply Suggestions (Gemini AI) - 1-2 weeks

- Sentiment Analysis & Tone Detection - 3-5 days

- Advanced Inbox Analytics - 3-4 weeks

- Scheduled Sending - 3-5 days

- Lead Context Sidebar - 2-3 weeks

### Q3 2026: International Feature

- Offline Support (IndexedDB) - 3-5 days

- Multi-Language Support & Translation - 1-2 weeks

### Related Documentation

- [Inbox Management Roadmap](/docs/features/inbox/roadmap) - Detailed timeline and roadmap items

- [Unified Inbox Overview](/docs/features/inbox/unified-inbox/overview) - Feature specifications

- Stalwart Investigation Spike - Internal task reference for architecture investigation

---

**Last Updated**: November 26, 2025
**Status**: Route specifications complete, awaiting Stalwart spike completion
**Next Review**: After Stalwart spike (Q4 2025)
