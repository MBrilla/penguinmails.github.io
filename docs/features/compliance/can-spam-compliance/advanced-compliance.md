---
title: "Advanced CAN-SPAM Compliance"
description: "Email classification, consent management, and record keeping for CAN-SPAM"
last_modified_date: "2025-11-24"
level: "2"
persona: "Marketing Teams, Compliance Officers"
status: "ACTIVE"
category: "Compliance"
---

# Advanced CAN-SPAM Compliance

## Transactional vs. Commercial Emails

### Transactional Emails (CAN-SPAM Exempt)

Emails that facilitate an agreed-upon transaction or update about an ongoing relationship:

- ✅ **Order confirmations** - Purchase receipts
- ✅ **Shipping notifications** - Delivery updates
- ✅ **Account alerts** - Password resets, security alerts
- ✅ **Service updates** - Changes to terms, features
- ✅ **Legal notices** - Privacy policy updates

**Rules for Transactional:**

- Primary purpose must be transactional
- Can include minimal commercial content
- No unsubscribe required (but best practice)
- Subject line must relate to transaction

### Commercial Emails (CAN-SPAM Applies)

Emails whose primary purpose is commercial advertisement:

- ⚠️ **Promotional campaigns** - Sales, discounts, offers
- ⚠️ **Newsletter with ads** - Content + promotional material
- ⚠️ **Product announcements** - New product launches
- ⚠️ **Event invitations** - Webinars, conferences

**Full CAN-SPAM compliance required:**

- Must include physical address
- Must include unsubscribe option
- Must honor opt-outs within 10 days
- Cannot use deceptive headers or subjects

### Email Classification in PenguinMails

**Campaign Type Selection:**

```yaml
Campaign Setup:
  Email Type:
    - Commercial (default) → Full CAN-SPAM requirements
    - Transactional → Minimal requirements (best practice compliance)
    - Administrative → Internal team communications
```

## Consent & Permission

### Express Consent (Best Practice)

While CAN-SPAM doesn't require prior consent, best practice is:

- **Opt-in forms** - Clear consent to receive emails
- **Confirmed interest** - Double opt-in for verification
- **Specific consent** - What types of emails they'll receive
- **Documented consent** - Timestamp and source

### PenguinMails Consent Features

- **Double opt-in** - Email confirmation required
- **Consent records** - Timestamp, IP, source
- **Preference center** - Users control email types
- **Consent proof** - Audit trail for compliance

## Record Keeping

### Required Documentation

- **Email content** - Archive all sent emails
- **Send dates** - When emails were sent
- **Opt-out requests** - Log all unsubscribe actions
- **Processing time** - Verify 10-day compliance
- **Recipient lists** - Who received each campaign

### PenguinMails Retention

```yaml
Retention Policy:
  Email Content: 24 months
  Send Logs: 36 months
  Opt-out Records: Indefinite (perpetual suppression)
  Audit Logs: 7 years
  Campaign Analytics: 12 months
```

---

## Related Documents

- [Overview](/docs/features/compliance/can-spam-compliance/overview) - CAN-SPAM overview

- [Core Requirements](/docs/features/compliance/can-spam-compliance/core-requirements) - Seven requirements

- [Technical Implementation](/docs/features/compliance/can-spam-compliance/technical-implementation) - Automated features
