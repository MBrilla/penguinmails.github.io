---
title: "Vault Secrets Management"
description: "HashiCorp Vault integration for secure credential storage, rotation, and access control"
level: "3"
status: "PLANNED"
roadmap_timeline: "MVP 2025"
priority: "Critical"
keywords: [vault, secrets, credentials, rotation, security, access-control]
---

# Vault Secrets Management

HashiCorp Vault integration for secure storage, rotation, and access control of email infrastructure credentials.

## Overview

PenguinMails integrates with HashiCorp Vault to manage sensitive credentials including SMTP passwords, API keys, DKIM private keys, and OAuth tokens. This ensures credentials are never stored in plaintext and can be rotated automatically.

## Supported Secret Types

### SSH Keys

Server access credentials for infrastructure management:

| Property | Value |
|----------|-------|
| Path | `secret/penguinmails/ssh/{tenant_id}/{key_name}` |
| Rotation | Manual or 90-day automatic |
| Format | PEM-encoded private/public key pair |

### SMTP Credentials

Mail server authentication:

| Property | Value |
|----------|-------|
| Path | `secret/penguinmails/smtp/{tenant_id}/{server_id}` |
| Fields | username, password, host, port, encryption |
| Rotation | 30-day recommended |

### API Keys

External service API keys:

| Property | Value |
|----------|-------|
| Path | `secret/penguinmails/api-keys/{tenant_id}/{service}` |
| Services | Postmark, Mailgun, SendGrid, SES |
| Rotation | Per-provider policy |

### DKIM Keys

Domain authentication keys:

| Property | Value |
|----------|-------|
| Path | `secret/penguinmails/dkim/{tenant_id}/{domain}` |
| Format | RSA 2048-bit private key |
| Rotation | Annual recommended |

## Secret Workflows

### Credential Storage

```typescript
async function storeCredential(
  tenantId: string,
  type: 'smtp' | 'api-key' | 'dkim',
  name: string,
  credential: Record<string, string>
): Promise<void> {
  const path = `secret/penguinmails/${type}/${tenantId}/${name}`;
  await vaultClient.write(path, { data: credential });
}
```

### Credential Retrieval

```typescript
async function getCredential(
  tenantId: string,
  type: string,
  name: string
): Promise<Record<string, string>> {
  const path = `secret/penguinmails/${type}/${tenantId}/${name}`;
  const response = await vaultClient.read(path);
  return response.data;
}
```

## Rotation Policies

Automated credential rotation based on type and risk level:

| Secret Type | Default Rotation | Risk Level | Alert Threshold |
|-------------|------------------|------------|-----------------|
| SSH Keys | 90 days | High | 14 days |
| SMTP Credentials | 30 days | Medium | 7 days |
| API Keys | Provider-specific | Medium | 7 days |
| DKIM Keys | 365 days | Low | 30 days |
| OAuth Tokens | Auto-refresh | Low | N/A |

### Rotation Triggers

- **Time-based**: Automatic rotation at specified intervals
- **Event-based**: Rotate on security incidents
- **Manual**: User-initiated rotation
- **Threshold**: Rotate when usage exceeds limits

## Access Control

### Vault Policies

```hcl
# Tenant-scoped policy
path "secret/data/penguinmails/smtp/{{identity.entity.metadata.tenant_id}}/*" {
  capabilities = ["read", "list"]
}

path "secret/data/penguinmails/api-keys/{{identity.entity.metadata.tenant_id}}/*" {
  capabilities = ["read", "list"]
}

# Admin policy
path "secret/data/penguinmails/+/+/*" {
  capabilities = ["create", "read", "update", "delete", "list"]
}
```

### Access Levels

| Role | Permissions |
|------|-------------|
| System | Full access to all secrets |
| Admin | Full access to tenant secrets |
| Operator | Read access to credentials |
| Developer | Limited API key access |

## Compliance Mapping

| Requirement | Vault Feature |
|-------------|---------------|
| SOC2 CC6.1 | Encryption at rest (AES-256-GCM) |
| SOC2 CC6.7 | Access logging and audit trail |
| ISO27001 A.10 | Cryptographic key management |
| GDPR Art. 32 | Security of processing |

## Audit Logging

All secret access is logged:

```json
{
  "timestamp": "2025-11-25T10:30:00Z",
  "operation": "read",
  "path": "secret/penguinmails/smtp/tenant-123/mail-server",
  "accessor": "worker-email-sender",
  "tenant_id": "tenant-123",
  "source_ip": "10.0.1.50"
}
```

## Related Documentation

- [Integrations Overview](/docs/features/integrations/overview) - Integration architecture
- [Third-Party Roadmap](/docs/features/integrations/third-party-roadmap) - Service integrations
- [Security Architecture](/docs/compliance-security/overview) - Security framework

---

**Last Updated:** November 25, 2025
**Status:** Planned - MVP Feature (Level 3)
**Target Release:** MVP 2025
**Owner:** Security Team
