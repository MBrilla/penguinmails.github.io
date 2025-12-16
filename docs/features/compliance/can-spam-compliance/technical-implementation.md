---
title: "CAN-SPAM Technical Implementation"
description: "Automated compliance features, validation, and reporting in PenguinMails"
last_modified_date: "2025-11-24"
level: "2"
persona: "Marketing Teams, Compliance Officers"
status: "ACTIVE"
category: "Compliance"
---

# CAN-SPAM Technical Implementation

## Automated Compliance Features

### Email Footer Auto-Insertion

```html
<!-- Automatically added to all commercial emails -->
<footer class="can-spam-footer">
  <p><small>This is a commercial message from {{company.name}}</small></p>
  <p>{{company.address}}</p>
  <p>
    <a href="{{unsubscribe_url}}">Unsubscribe</a> |
    <a href="{{preferences_url}}">Email Preferences</a>
  </p>
</footer>
```

### List-Unsubscribe Header

```text
List-Unsubscribe: <mailto:unsubscribe@penguinmails.com?subject=unsubscribe>,
                  <https://penguinmails.com/unsubscribe/{{campaign_id}}/{{contact_id}}>
List-Unsubscribe-Post: List-Unsubscribe=One-Click
```

## Compliance Validation

### Pre-Send Validation

Before any campaign sends, PenguinMails automatically checks:

- [ ] Valid sender domain (authenticated)
- [ ] Physical address present
- [ ] Unsubscribe link included
- [ ] Subject line not deceptive (spam score)
- [ ] From/Reply-To addresses valid
- [ ] Commercial content identified

**Validation fails?** → Campaign blocked until resolved

### Unsubscribe Processing

```text
User clicks unsubscribe
  ↓
Immediate removal from active campaigns
  ↓
Added to global suppression list
  ↓
Future sends automatically blocked
  ↓
Confirmation email sent (optional)
```

### Suppression List Features

- **Perpetual storage** - Never delete unsubscribed emails
- **Import/export** - Maintain external suppression lists
- **Cross-workspace** - Share suppression across workspaces
- **Compliance reports** - Audit opt-out processing times

## Compliance Reports

### Monthly Compliance Report

**Automated reports include:**

- Total emails sent
- Opt-out rate and processing time
- Compliance score (100% = fully compliant)
- Failed validation incidents
- Remediation actions taken

### Audit Trail

**Track all compliance activities:**

- Campaign approvals
- Template modifications
- Unsubscribe processing
- Suppression list updates
- Compliance setting changes

## Support & Resources

### FTC CAN-SPAM Act Resources

- **FTC CAN-SPAM Guide** - <https://www.ftc.gov/tips-advice/business-center/guidance/can-spam-act-compliance-guide-business>
- **Email Law Compliance** - <https://www.ftc.gov/business-guidance/resources/can-spam-act-compliance-guide-business>

### PenguinMails Compliance Support

- **Compliance Team** - <compliance@penguinmails.com>
- **Legal Documentation** - Terms of Service, Privacy Policy
- **Support Portal** - Compliance questions and guidance

---

**Regulatory Authority:** Federal Trade Commission (FTC)
**Applies To:** All commercial emails sent to US recipients

*PenguinMails automates CAN-SPAM compliance but users are ultimately responsible for ensuring their email practices comply with all applicable laws.*

## Related Documents

- [Overview](/docs/features/compliance/can-spam-compliance/overview) - CAN-SPAM overview

- [Core Requirements](/docs/features/compliance/can-spam-compliance/core-requirements) - Seven requirements

- [Advanced Compliance](/docs/features/compliance/can-spam-compliance/advanced-compliance) - Email types and consent

- [Email Authentication](/docs/features/domains/sender-authentication) - SPF, DKIM, DMARC

- [Unsubscribe Management](/docs/features/compliance/unsubscribe-management) - Opt-out automation
