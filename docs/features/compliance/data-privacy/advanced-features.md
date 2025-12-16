---
title: "Advanced Privacy Features"
description: "Privacy by design, data retention policies, and third-party data sharing controls"
last_modified_date: "2025-01-15"
level: "2"
persona: "Privacy Officers, Compliance Teams"
---

# Advanced Privacy Features

Level 2 privacy features implement privacy by design principles, automated data lifecycle management, and transparent third-party sharing controls.

## Privacy by Design and Default

Privacy considerations are built into every feature from the ground up.

### Design Principles

**Proactive Not Reactive:** Privacy protections in place before data collected. Security reviews conducted for new features. Privacy impact assessments required for high-risk processing.

**Privacy as Default:** Most privacy-protective settings enabled by default. Users must opt-in to data sharing. No pre-checked consent boxes. Minimal data collection by default.

**Privacy Embedded:** Not a bolt-on feature but integrated into architecture. Part of development lifecycle with continuous monitoring and improvement.

### Default Settings

New user accounts are configured with privacy-protective defaults:

```yaml
New User Defaults:
  data_sharing: false
  analytics_tracking: minimal
  marketing_emails: false
  third_party_integrations: disabled
  session_timeout: 30 minutes
  mfa: recommended (not required)
```

## Data Retention and Deletion

Automated data lifecycle management ensures compliance with retention policies.

### User Data Retention

**Active Users:**
- Profile Data: Indefinite (while account active)
- Activity Logs: 12 months (rolling)

**Deleted Accounts:**
- Soft Delete: 30 days (recovery period)
- Hard Delete: Permanent after 30 days
- Audit Logs: 7 years (anonymized, compliance)

### Contact Data Retention

**Active Contacts:**
- Engaged (opened/clicked): Indefinite
- Inactive (no engagement): 24 months

**Unsubscribed:**
- Email address: Perpetual (suppression list)
- Other data: Deleted after 30 days

**Bounced:**
- Hard bounce: Immediate suppression
- Soft bounce (3x): Suppression after 3 attempts

### Campaign Data Retention

- Metadata: 36 months
- Email Content: 24 months
- Send Logs: 36 months
- Analytics: 12 months (aggregated indefinitely)

### Automated Deletion Schedule

- **Daily:** Remove expired soft-deleted accounts
- **Weekly:** Clean up inactive bounce logs
- **Monthly:** Archive old campaign data
- **Quarterly:** Review and clean up test data

## Third-Party Data Sharing

Transparency about sub-processors and when data is shared.

### Infrastructure Sub-Processors

**NileDB** - Multi-tenant PostgreSQL database
- Purpose: Data storage
- Location: US (EU option Q2 2026)
- Data: All platform data

**Redis** - Caching and queue management
- Purpose: Performance, background jobs
- Location: Same as application server
- Data: Temporary cache, job queues

**Hostwind** - VPS hosting
- Purpose: Infrastructure hosting
- Location: US data centers
- Data: All application and database data

### Payment Processing

**Stripe** - Payment processor
- Purpose: Subscription billing
- Location: Global (US-based)
- Data: Payment method, billing address (tokenized)

### Email Delivery (Optional)

**Postmark** - Transactional email delivery
- Purpose: High-deliverability email sending
- Location: US
- Data: Email content, recipient addresses

**Mailgun** - Bulk email delivery
- Purpose: Marketing campaign delivery
- Location: US (EU option available)
- Data: Email content, recipient addresses

### Data Sharing Controls

**When We Share Data:**
- With user consent (integrations)
- To provide requested services
- With sub-processors (listed above)
- For legal compliance (subpoena, court order)

**When We Don't Share Data:**
- Selling to third parties
- Marketing to non-users
- Sharing with affiliates
- Unrelated business purposes

---

**Related Documents:**
- [Contact Privacy](contact-privacy.md)
- [Technical Implementation](technical-implementation.md)
- [GDPR Compliance](/docs/features/compliance/gdpr-compliance)
