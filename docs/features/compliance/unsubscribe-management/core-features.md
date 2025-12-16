---
title: "Core Unsubscribe Features"
description: "One-click unsubscribe, suppression lists, and email preference center"
last_modified_date: "2025-01-15"
level: "1"
persona: "Marketing Teams, Operations"
status: "ACTIVE"
category: "Compliance"
---

# Core Unsubscribe Features

Essential unsubscribe functionality including one-click opt-out, suppression lists, and preference management.

## One-Click Unsubscribe

**Instant opt-out with a single click** - no login required, no confirmation needed.

### How It Works

1. **User clicks unsubscribe link** in email footer
2. **Immediate processing** - removed from all active campaigns
3. **Added to suppression list** - permanently blocked from future sends
4. **Confirmation page** - user sees success message
5. **Optional confirmation email** - receipt of unsubscribe request

### Technical Implementation

**List-Unsubscribe Header (RFC 8058)**

```text
List-Unsubscribe: <https://penguinmails.com/unsubscribe/{{token}}>
List-Unsubscribe-Post: List-Unsubscribe=One-Click
```

**Benefits:**
- Gmail/Outlook "Unsubscribe" button compatibility
- Reduces spam complaints (users unsubscribe vs report spam)
- Improves sender reputation
- Industry best practice

---

## Global Suppression List

**Permanent opt-out across all campaigns and workspaces** (optional tenant-wide setting).

### Suppression Types

| Type | Scope | Management | Override |
|------|-------|------------|----------|
| **Global Suppression** | Platform-wide | System admin | Cannot override |
| **Tenant Suppression** | Company-wide | Tenant admin | Within tenant |
| **Workspace Suppression** | Team-specific | Workspace admin | Within workspace |
| **Campaign-Type** | Granular by type | User preference | User-controlled |

### Suppression List Features

- **Import/Export** - CSV import of suppression lists
- **Bulk Add** - Add multiple addresses at once
- **Manual Add** - Add individual unsubscribes
- **Never Expires** - Perpetual suppression
- **Audit Trail** - Track when/why addresses added
- **API Access** - Programmatic suppression management

---

## Unsubscribe Link in Every Email

**Automatic footer insertion** ensures every commercial email includes unsubscribe option.

### Footer Template

```html
<footer class="email-footer">
  <p>{{company.name}}<br>{{company.address}}</p>

  <p>
    <a href="{{unsubscribe_url}}" class="unsubscribe-link">
      Unsubscribe
    </a> |
    <a href="{{preferences_url}}">
      Email Preferences
    </a>
  </p>

  <p><small>© {{current_year}} {{company.name}}. All rights reserved.</small></p>
</footer>
```

### Template Variables

| Variable | Description |
|----------|-------------|
| `{{unsubscribe_url}}` | One-click unsubscribe (unique per recipient) |
| `{{preferences_url}}` | Email preference center |
| `{{company.name}}` | Company name from profile |
| `{{company.address}}` | Physical address (CAN-SPAM requirement) |
| `{{current_year}}` | Auto-updating year |

### Visibility Requirements

- Clearly visible and readable
- Font size minimum 10px
- Contrasting color from background
- Positioned in footer (standard location)
- Mobile-optimized (large tap target)

---

## Email Preference Center

**Let users control what emails they receive** instead of full unsubscribe.

### Preference Options

**Email Types:**
- Newsletters (weekly updates)
- Product Updates (new features)
- Promotional Offers (sales, discounts)
- Event Invitations (webinars, conferences)
- Account Notifications (important updates)

**Frequency:**
- Daily
- Weekly
- Monthly
- Only important updates

**Format:**
- HTML (rich formatting)
- Plain Text (basic formatting)

### Benefits of Preference Center

- **Reduce full unsubscribes** - users can reduce frequency vs opt-out entirely
- **Improve engagement** - send only relevant content
- **Better insights** - understand user preferences
- **Compliance-friendly** - respect user choices

---

**Related Documents:**
- [Overview](overview.md)
- [Advanced Processing](advanced-processing.md)
- [Technical Implementation](technical.md)
