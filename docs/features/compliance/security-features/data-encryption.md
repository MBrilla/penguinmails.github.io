---
title: "Data Encryption"
last_modified_date: "2025-12-15"
level: "1"
persona: "Documentation Users"
---

# Data Encryption

At-rest and in-transit encryption protecting all sensitive data.

## At-Rest Encryption

### Database

- NileDB built-in encryption
- Sensitive fields individually encrypted (passwords, API keys, OAuth tokens)
- AES-256 encryption standard

### File Storage

- Email templates encrypted
- Uploaded files encrypted
- Backup encryption enabled

## In-Transit Encryption

All communications secured:

- HTTPS for all web traffic (TLS 1.2+)
- SMTP TLS for email transmission
- API requests over HTTPS only
- WebSocket connections over WSS

## Secure Credential Storage

### Password Handling

- Bcrypt hashing (cost factor 12)
- Never stored in plaintext
- Password complexity requirements enforced
- Password reset with time-limited tokens

### API Keys & Secrets

- Encrypted at rest (AES-256)
- Never logged or exposed in errors
- Rotation supported
- Scoped permissions per key

### OAuth Tokens

- Encrypted storage
- Automatic expiration
- Refresh token rotation
- Revocation support

---

[← Back to Security Features](./README)
