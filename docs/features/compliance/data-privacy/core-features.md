---
title: "Core Privacy Features"
description: "Data collection, transparent policies, user control, and security fundamentals"
last_modified_date: "2025-01-15"
level: "2"
persona: "Legal Teams, Privacy Officers"
---

# Core Privacy Features

Level 1 privacy features provide foundational protection through minimal data collection, transparent policies, user control, and robust security.

## Privacy-First Data Collection

Collect minimum necessary data for email marketing functionality.

### Required Data

**For Registration:**
- Email address (authentication)
- Name (personalization)
- Password (hashed, never stored in plain text)

**For Campaigns:**
- Contact email addresses
- Optional: First name, last name, company
- Custom fields (user-defined, optional)

### Not Collected

PenguinMails explicitly avoids collecting:
- Social Security Numbers
- Financial information (Stripe handles payments)
- Sensitive health data
- Biometric data
- Unnecessary tracking data

## Transparent Privacy Policies

Clear, accessible privacy policies explain data practices.

### Policy Availability

- **Marketing website** - https://penguinmails.com/privacy
- **In-app link** - Footer of every page
- **Onboarding** - Shown during signup
- **API documentation** - Privacy considerations for developers

### Policy Contents

1. What data we collect - Comprehensive list
2. Why we collect it - Purpose for each data type
3. How we use it - Processing activities
4. Who we share it with - Third-party processors
5. How long we keep it - Retention policies
6. User rights - Access, deletion, portability
7. Contact information - Privacy officer contact

### Policy Updates

Notification required for material changes via email. Version history tracks policy changes over time. Clear indication of effective dates and whether continued use implies acceptance or requires explicit re-consent.

## User Data Control

Empower users to manage their own data through self-service tools.

### Data Access

**User Profile Data Export:**
Download all personal data in JSON or CSV format. Includes profile, preferences, campaign history, analytics. Generated within 24 hours. Available via Settings → Privacy → Export My Data.

**Contact Data Export:**
Export all contacts and lists in CSV format compatible with other tools. Includes all custom fields. Available via Leads → Export.

### Data Correction

- Self-service profile editing for name, email, preferences
- Contact data updates for correcting contact information
- Bulk corrections via CSV import
- API updates for programmatic data correction

### Data Deletion

**Account deletion** provides complete removal of user account with 30-day grace period for soft delete recovery. Permanent deletion after 30 days is irreversible. Cascade delete removes all associated data.

**What Gets Deleted:**
- User profile and credentials
- Contact lists and segmentation
- Campaign data and templates
- All personally identifiable information (PII)

**What's Retained (Legal/Compliance):**
- Audit logs (7 years, anonymized)
- Transaction records (7 years for tax compliance)
- Abuse/spam reports (perpetual, for platform security)

## Data Security

Multi-layered security protects data at rest and in transit.

### Encryption at Rest

**Database encryption:** PostgreSQL transparent data encryption protects all stored data.

**Field-level encryption:** Additional encryption for sensitive fields provides defense in depth.

**Encrypted backups:** All backups encrypted with secure key management and rotation.

### Encryption in Transit

**TLS 1.3:** All connections encrypted (web, API) using modern protocols.

**SMTP TLS:** Email transmission encrypted for protection during delivery.

**No plain HTTP:** HTTPS enforced with HSTS enabled to prevent downgrade attacks.

### Access Controls

- Role-based access ensures users have minimum necessary permissions
- Multi-factor authentication available for enhanced security (Planned)
- Session management with automatic logout and secure session tokens
- IP restrictions available for optional IP allowlisting for enterprise

---

**Related Documents:**
- [Advanced Privacy Features](advanced-features.md)
- [Technical Implementation](technical-implementation.md)
- [Security Framework](/docs/compliance-security/enterprise/security-framework)
