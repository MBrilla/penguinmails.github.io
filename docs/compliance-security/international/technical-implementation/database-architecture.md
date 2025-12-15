---
title: "Database Architecture"
last_modified_date: "2025-12-15"
level: "3"
persona: "Developers"
---

# Database Architecture

PostgreSQL migration benefits and security enhancements for GDPR compliance.

## PostgreSQL Migration Benefits

- **Enhanced Security Features:** Row-level security, advanced encryption options
- **GDPR Compliance:** Built-in features for data subject rights implementation
- **Audit Capabilities:** Comprehensive logging and monitoring capabilities
- **Scalability:** Enterprise-grade performance for large contact databases

## Encryption Implementation

```sql
-- Field-level encryption example for email addresses
CREATE EXTENSION IF NOT EXISTS pgcrypto;

CREATE TABLE contacts_encrypted (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email_encrypted BYTEA NOT NULL,
    email_hash TEXT NOT NULL,
    consent_status BOOLEAN DEFAULT false,
    consent_timestamp TIMESTAMP WITH TIME ZONE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Function to encrypt email addresses
CREATE OR REPLACE FUNCTION encrypt_email(email TEXT)
RETURNS BYTEA AS $$
BEGIN
    RETURN pgp_sym_encrypt(email, 'encryption_key_here');
END;
$$ LANGUAGE plpgsql;
```

## Data Segmentation Strategy

- **Consent Status Isolation:** Separate schemas for consent records and personal data
- **Access Control Matrix:** Role-based access to different data categories
- **Audit Trail Separation:** Independent logging system for compliance monitoring

---

[← Back to Technical Implementation](./README)
