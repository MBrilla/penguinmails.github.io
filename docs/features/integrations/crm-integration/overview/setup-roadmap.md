---
title: "Setup & Feature Roadmap"
description: "User setup experience, admin controls, and feature phases for CRM integration"
last_modified_date: "2025-01-20"
level: 2
persona: Product Managers
keywords: setup, user experience, admin controls, roadmap, features
---

# Setup & Feature Roadmap

This document describes the user setup experience for CRM integration, admin controls, and the phased feature rollout plan.

## Integration Setup Flow

### User Experience

1. **Connect**: Click "Connect Salesforce" in settings
2. **Authenticate**: OAuth login to CRM
3. **Configure**: Select objects to sync (Contacts, Leads, etc.)
4. **Map Fields**: Map custom fields between systems
5. **Activate**: Enable bi-directional sync
6. **Monitor**: View sync status and activity logs

### Admin Controls

- Enable/disable specific CRMs
- Configure sync frequency
- Map custom fields
- View sync logs and errors
- Disconnect and reconnect integrations

## Feature Breakdown

### MVP (Q1 2026)

- ✓ Salesforce integration (contacts, leads, activities)
- ✓ HubSpot integration (contacts, companies, activities)
- ✓ Bi-directional contact sync
- ✓ Email activity logging
- ✓ Basic field mapping
- ✓ OAuth authentication

### Phase 2 (Q2-Q3 2026)

- Advanced field mapping (formulas)
- Workflow triggers
- Custom object sync
- Additional CRMs (Zoho, Close.io)
- Bulk data import/export

### Enterprise (Q4 2026+)

- Real-time sync (< 1 minute)
- Advanced conflict resolution
- Custom integration builder
- Multi-CRM support per tenant
- Advanced audit logging

## Setup UI Wireframe

### Connect Page

```text
┌─────────────────────────────────────────────────────────────┐
│  CRM Integrations                                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────┐  ┌─────────────────┐                   │
│  │   Salesforce    │  │    HubSpot      │                   │
│  │                 │  │                 │                   │
│  │   [Connect]     │  │   [Connect]     │                   │
│  └─────────────────┘  └─────────────────┘                   │
│                                                             │
│  ┌─────────────────┐  ┌─────────────────┐                   │
│  │   Pipedrive     │  │   Coming Soon   │                   │
│  │                 │  │                 │                   │
│  │   [Connect]     │  │   Zoho, Close   │                   │
│  └─────────────────┘  └─────────────────┘                   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Configuration Page

```text
┌─────────────────────────────────────────────────────────────┐
│  Salesforce Configuration                    [Disconnect]   │
├─────────────────────────────────────────────────────────────┤
│  Status: ● Connected                                        │
│  Last Sync: 2 minutes ago                                   │
│                                                             │
│  Sync Settings                                              │
│  ├─ Objects to Sync                                         │
│  │  ☑ Contacts                                              │
│  │  ☑ Leads                                                 │
│  │  ☐ Accounts                                              │
│  │                                                          │
│  ├─ Sync Direction                                          │
│  │  ◉ Bi-directional                                        │
│  │  ○ PenguinMails → Salesforce only                        │
│  │  ○ Salesforce → PenguinMails only                        │
│  │                                                          │
│  ├─ Conflict Resolution                                     │
│  │  ◉ Salesforce wins                                       │
│  │  ○ PenguinMails wins                                     │
│  │  ○ Most recent wins                                      │
│  │                                                          │
│  └─ [Manage Field Mapping]                                  │
│                                                             │
│  [Save Settings]                                            │
└─────────────────────────────────────────────────────────────┘
```

### Sync Status Page

```text
┌─────────────────────────────────────────────────────────────┐
│  Sync Activity                               [Sync Now]     │
├─────────────────────────────────────────────────────────────┤
│  Today's Activity                                           │
│  • Contacts synced: 234                                     │
│  • Activities logged: 1,892                                 │
│  • Conflicts resolved: 3                                    │
│  • Errors: 0                                                │
│                                                             │
│  Recent Sync Log                                            │
│  ┌───────────────────────────────────────────────────────┐  │
│  │ 10:34 AM  Contact synced: john@example.com → SF       │  │
│  │ 10:33 AM  Activity logged: Email Opened               │  │
│  │ 10:32 AM  Contact synced: jane@example.com ← SF       │  │
│  │ 10:31 AM  Conflict resolved: company field            │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                             │
│  [View Full Log]                                            │
└─────────────────────────────────────────────────────────────┘
```

## Related Documentation

- [Timeline & Milestones](./timeline-milestones.md)
- [Technical Security](./technical-security.md)
- [CRM Settings Routes](/docs/features/design/routes/dashboard-settings-integrations)
