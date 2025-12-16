---
title: "Privacy Technical Implementation"
description: "Privacy-enhancing technologies, APIs, and audit logging for developers"
last_modified_date: "2025-01-15"
level: "3"
persona: "Developers, Security Engineers"
---

# Privacy Technical Implementation

Level 3 technical implementation details for privacy-enhancing technologies, APIs, and audit logging.

## Privacy-Enhancing Technologies

### Pseudonymization

Separate PII from operational data to minimize exposure during processing:

```sql
-- Separate tables for PII vs operational data
CREATE TABLE contacts (
  id UUID PRIMARY KEY,
  email VARCHAR(255) ENCRYPTED, -- PII
  created_at TIMESTAMP
);

CREATE TABLE contact_engagement (
  contact_id UUID REFERENCES contacts(id),
  campaign_id UUID,
  event_type VARCHAR(50),
  occurred_at TIMESTAMP
  -- No PII stored here
);
```

This separation allows analytics queries on engagement data without accessing PII.

### Data Anonymization

Anonymize audit logs after retention period to maintain compliance records without PII:

```javascript
// Anonymization Job (runs quarterly)
UPDATE audit_logs
SET
  user_email = 'anonymized@example.com',
  ip_address = '0.0.0.0',
  user_agent = 'anonymized'
WHERE
  created_at < NOW() - INTERVAL '90 days'
  AND anonymized = false;
```

## Privacy API Endpoints

### Export User Data

Request a complete export of user data in JSON or CSV format:

```javascript
POST /api/v1/privacy/export
Authorization: Bearer {user_token}

Response:
{
  "export_id": "exp_abc123",
  "status": "processing",
  "estimated_completion": "2025-11-24T12:00:00Z",
  "download_url": null // available when complete
}
```

Exports are generated asynchronously and available for download within 24 hours.

### Delete User Account

Request account deletion with confirmation:

```javascript
DELETE /api/v1/users/me
Authorization: Bearer {user_token}

{
  "confirmation": "DELETE",
  "reason": "no_longer_needed" // optional
}

Response:
{
  "status": "scheduled_for_deletion",
  "soft_delete_until": "2025-12-24T10:30:00Z",
  "permanent_deletion_date": "2025-12-24T10:30:00Z"
}
```

Account enters 30-day soft delete period before permanent deletion.

## Privacy Audit Logging

Comprehensive logging of privacy-sensitive actions enables compliance verification:

```javascript
Privacy Audit Events:

- user.data_exported
- user.data_deleted
- user.consent_updated
- contact.data_accessed
- contact.data_exported
- contact.data_deleted
- privacy_policy.accepted
- privacy_policy.updated
- dpa.signed
```

### Audit Log Schema

```sql
CREATE TABLE privacy_audit_logs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  user_id UUID REFERENCES users(id),
  event_type VARCHAR(100) NOT NULL,
  target_type VARCHAR(50), -- 'user', 'contact', 'policy'
  target_id UUID,
  metadata JSONB,
  ip_address INET,
  user_agent TEXT,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_privacy_audit_tenant ON privacy_audit_logs(tenant_id);
CREATE INDEX idx_privacy_audit_event ON privacy_audit_logs(event_type);
CREATE INDEX idx_privacy_audit_created ON privacy_audit_logs(created_at);
```

### Audit Retention

Privacy audit logs are retained for 7 years to meet compliance requirements. After 90 days, PII fields are anonymized while event records are preserved.

## Security Integration

### Field-Level Encryption

Sensitive fields use application-layer encryption in addition to database encryption:

```typescript
import { encrypt, decrypt } from '@/lib/encryption';

// Encrypt before storage
const encryptedEmail = await encrypt(contact.email);

// Decrypt when needed
const email = await decrypt(encryptedEmail);
```

### Access Control Enforcement

All privacy-sensitive operations check user permissions:

```typescript
async function exportContactData(userId: string, contactId: string) {
  // Verify user has access to contact
  const hasAccess = await checkContactAccess(userId, contactId);
  if (!hasAccess) {
    throw new UnauthorizedError('Access denied');
  }
  
  // Log the access
  await logPrivacyEvent('contact.data_accessed', {
    userId,
    contactId
  });
  
  // Return data
  return await getContactData(contactId);
}
```

---

**Related Documents:**
- [Core Privacy Features](core-features.md)
- [Advanced Privacy Features](advanced-features.md)
- [Data Encryption](/docs/implementation-technical/security/data-encryption)
- [Access Controls](/docs/implementation-technical/security/access-controls)
