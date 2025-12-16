---
title: "CAN-SPAM Core Requirements"
description: "The seven core CAN-SPAM requirements and how PenguinMails implements them"
last_modified_date: "2025-11-24"
level: "2"
persona: "Marketing Teams, Compliance Officers"
status: "ACTIVE"
category: "Compliance"
---

# CAN-SPAM Core Requirements

## 1. Don't Use False or Misleading Header Information

**Requirement:** "From," "To," "Reply-To," and routing information must be accurate and identify the business sending the message.

### PenguinMails Implementation

- ✅ **Verified sender addresses** - Only send from authenticated domains
- ✅ **Accurate From name** - Clearly identifies your business
- ✅ **Valid Reply-To** - Monitored mailbox for replies
- ✅ **No spoofing** - SPF, DKIM, DMARC prevent header manipulation

### Setup Checklist

- [ ] Use your business domain as sender
- [ ] Set "From" name to your business or brand name
- [ ] Provide working Reply-To address
- [ ] Configure SPF/DKIM/DMARC records

## 2. Don't Use Deceptive Subject Lines

**Requirement:** Subject lines must accurately reflect the content of the message.

### Best Practices

- ✅ **Honest subject lines** - Accurately describe email content
- ✅ **No clickbait** - Avoid misleading or sensational subjects
- ✅ **Match content** - Subject and email body should align
- ✅ **No fake urgency** - Don't create false sense of urgency

## 3. Identify the Message as an Advertisement

**Requirement:** You must clearly and conspicuously disclose that the message is an advertisement.

### When Required

- **Commercial messages** - Selling products or services
- **First contact** - No prior business relationship
- **Promotional content** - Marketing or advertising material

### Exemptions

- **Transactional emails** - Order confirmations, shipping, account updates
- **Relationship messages** - Existing customer communications
- **Requested information** - User-initiated contact

**Example Disclosure:**
```text
This is an advertisement from [Your Company Name]
```

## 4. Tell Recipients Where You're Located

**Requirement:** Include your valid physical postal address in every email.

### Address Requirements

- ✅ **Street address** - Valid postal delivery location
- ✅ **P.O. Box** - Registered with USPS
- ✅ **Private mailbox** - At commercial mail receiving agency

### Configuration

```yaml
Company Address Setup:
  Location: Settings → Company Profile → Physical Address
  Required Fields:
    - Street address
    - City, State, ZIP
    - Country
  Display: Automatically added to all commercial emails
```

## 5. Provide an Opt-Out Method

**Requirement:** Give recipients a clear, easy way to opt out of future emails.

### Unsubscribe Requirements

- ✅ **Conspicuous** - Easy to find and recognize
- ✅ **Easy to use** - Simple one-click or reply-to method
- ✅ **No login required** - Work without authentication
- ✅ **No fees** - Free to unsubscribe
- ✅ **Include in every email** - All commercial messages

### PenguinMails Unsubscribe Features

- **One-Click Unsubscribe** - RFC 8058 compliant
- **Web-based unsubscribe** - Hosted unsubscribe page
- **Instant processing** - Immediate removal from future sends
- **Confirmation page** - User feedback on successful unsubscribe

## 6. Honor Opt-Out Requests Promptly

**Requirement:** Process opt-out requests within 10 business days.

### PenguinMails Compliance

- ✅ **Instant opt-out** - Removed immediately (not 10 days)
- ✅ **Automated processing** - No manual intervention needed
- ✅ **Suppression list** - Prevent re-adds
- ✅ **Global opt-out** - Applies across all campaigns
- ✅ **Permanent** - Cannot resubscribe without explicit consent

## 7. Monitor What Others Are Doing on Your Behalf

**Requirement:** Even if you hire a company to handle your email marketing, you cannot contract away your legal responsibility to comply.

### PenguinMails Accountability Features

- **Audit logs** - Track all email sends and modifications
- **User activity** - Monitor team member actions
- **Approval workflows** - Require manager approval for sends
- **Compliance reports** - Regular compliance status reports

---

## Related Documents

- [Overview](/docs/features/compliance/can-spam-compliance/overview) - CAN-SPAM overview

- [Advanced Compliance](/docs/features/compliance/can-spam-compliance/advanced-compliance) - Email types and consent

- [Technical Implementation](/docs/features/compliance/can-spam-compliance/technical-implementation) - Automated features
