---
title: "Advanced Unsubscribe Processing"
description: "Automated processing, suppression sync, spam handling, and manual management"
last_modified_date: "2025-01-15"
level: "2"
persona: "Marketing Teams, Operations"
status: "ACTIVE"
category: "Compliance"
---

# Advanced Unsubscribe Processing

Advanced unsubscribe features including automated processing, CRM sync, and spam complaint handling.

## Automated Opt-Out Processing

**Instant processing ensures compliance** with 10-day CAN-SPAM requirement (PenguinMails processes immediately).

### Processing Workflow

```text
Unsubscribe Request Received
  ↓
Validate Token (prevent abuse)
  ↓
Remove from Active Campaigns (immediate)
  ↓
Add to Suppression List (permanent)
  ↓
Update User Preferences (if preference center)
  ↓
Send Confirmation (optional)
  ↓
Audit Log Entry (compliance record)
```

### Processing Time Guarantee

| Action | Processing Time |
|--------|-----------------|
| Immediate removal | 0 seconds (not 10 days) |
| Active campaigns | Removed before next send |
| Scheduled sends | Excluded from queued emails |
| Engagement tracking | Unsubscribe timestamp recorded |

---

## Suppression List Sync

**Keep external suppression lists synchronized** with PenguinMails.

### Import Sources

**CSV Import:**

```csv
email,reason,date,source
user@example.com,user_request,2025-11-24,manual
spam@example.com,spam_complaint,2025-11-23,abuse_report
```

**API Import:**

```javascript
POST /api/v1/suppression-list/bulk
Content-Type: application/json

{
  "emails": [
    {
      "email": "user@example.com",
      "reason": "user_request",
      "source": "external_crm"
    }
  ]
}
```

### CRM Integration

- Sync unsubscribes with Salesforce, HubSpot
- Bi-directional sync (update both systems)
- Real-time webhook updates
- Scheduled sync jobs

### Export Options

- **CSV format** - Import into other systems
- **Full export** - All suppressed emails
- **Date range** - Exports for specific period
- **Reason filter** - Export by unsubscribe reason

---

## Spam Complaint Handling

**Automatically process spam complaints** from ISPs and email clients.

### Feedback Loop (FBL) Processing

ISPs (Gmail, Yahoo, Outlook) report when users mark emails as spam.

**PenguinMails FBL Integration:**

1. ISP sends spam complaint to PenguinMails
2. Email address automatically added to suppression list
3. User removed from all active campaigns
4. Tenant notified of complaint
5. Complaint tracked for deliverability monitoring

### Supported ISPs

- Gmail (via Google Postmaster)
- Outlook/Hotmail (via JMRP/SNDS)
- Yahoo (via Complaint Feedback Loop)
- AOL (via Feedback Loop)

### Bounce Handling

**Hard Bounces (permanent delivery failures):**
- Invalid email addresses
- Non-existent domains
- Blocked by recipient server
- **Action:** Automatically suppress after 1 failure

**Soft Bounces (temporary delivery failures):**
- Mailbox full
- Server temporarily unavailable
- Email too large
- **Action:** Suppress after 3 consecutive soft bounces (configurable)

---

## Manual Unsubscribe Management

**Admin tools for managing unsubscribes** when needed.

### Admin Actions

**Add to Suppression List:**
- Single email address
- Bulk CSV upload
- Reason required (audit trail)
- Optional expiration date

**Remove from Suppression List:**
- Requires user consent (documented)
- Re-opt-in workflow recommended
- Audit log entry created
- Admin approval required (optional)

**View Suppression History:**
- When added to suppression list
- Reason for suppression
- Source of suppression
- Admin who added (if manual)

---

**Related Documents:**
- [Overview](overview.md)
- [Core Features](core-features.md)
- [Technical Implementation](technical.md)
