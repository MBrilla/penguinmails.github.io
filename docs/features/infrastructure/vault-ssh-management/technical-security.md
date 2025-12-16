---
title: "Technical Architecture & Security"
description: "Vault architecture, API endpoints, security considerations, and compliance"
last_modified_date: "2025-11-26"
level: 2
persona: "Technical Teams, Operations"
keywords: ["vault-architecture", "api-endpoints", "security", "compliance", "roadmap"]
---

# Technical Architecture & Security

Technical implementation details including Vault secret structure, API endpoints, access control, security considerations, and compliance requirements.

## Technical Architecture

### Vault Secret Structure

```text
vault/vps/{tenant_id}/
├── admin_ssh/
│   ├── private_key       # RSA 4096-bit private key
│   ├── public_key        # RSA 4096-bit public key
│   ├── created_at        # ISO 8601 timestamp
│   ├── last_rotated      # ISO 8601 timestamp
│   └── rotation_policy   # "90_days"
└── tenant_ssh/
    ├── private_key       # RSA 4096-bit private key
    ├── public_key        # RSA 4096-bit public key
    ├── created_at        # ISO 8601 timestamp
    ├── last_rotated      # ISO 8601 timestamp
    └── rotation_policy   # "90_days"
```

## API Endpoints

### Tenant Endpoints

```typescript
// Retrieve SSH credentials (public info only)
GET /api/v1/tenant/infrastructure/ssh-credentials
Response: {
  vpsIp: string;
  sshUsername: string;
  sshPort: number;
  keyFingerprint: string;
  lastRotated: string;
  nextRotation: string;
}

// Download private key (one-time)
POST /api/v1/tenant/infrastructure/ssh-credentials/download
Response: File download (tenant-ssh-key.pem)

// Rotate SSH key manually
POST /api/v1/tenant/infrastructure/ssh-credentials/rotate
Response: { status: 'success'; message: string; newKeyFingerprint: string; }

// Revoke SSH access
POST /api/v1/tenant/infrastructure/ssh-credentials/revoke
Response: { status: 'success'; message: 'SSH access revoked immediately.'; }

// Regenerate SSH key
POST /api/v1/tenant/infrastructure/ssh-credentials/regenerate
Response: { status: 'success'; message: string; newKeyFingerprint: string; }
```

### Admin Endpoints

```typescript
// Get Vault health status
GET /api/v1/admin/vault/health
Response: { sealed: boolean; activeNode: string; storageStatus: string; lastBackup: string; }

// List all tenant secrets
GET /api/v1/admin/secrets/tenants
Response: TenantSecretsSummary[]

// Get tenant secret details
GET /api/v1/admin/secrets/tenant/{tenant_id}
Response: { sshKeys: object; smtpCredentials: object; apiKeys: object[]; }

// Trigger bulk SSH key rotation
POST /api/v1/admin/secrets/rotate-all
Body: { secretType: 'ssh' | 'smtp' | 'dkim' }
Response: { status: 'success'; rotatedCount: number; failedCount: number; }
```

## Access Control

### Vault Policies

```hcl
# Tenant read-only access to own SSH keys
path "vps/{{identity.entity.metadata.tenant_id}}/tenant_ssh/*" {
  capabilities = ["read"]
}

# Admin full access to all SSH keys
path "vps/*/admin_ssh/*" {
  capabilities = ["create", "read", "update", "delete", "list"]
}

path "vps/*/tenant_ssh/*" {
  capabilities = ["create", "read", "update", "delete", "list"]
}
```

## Security Considerations

### Threat Mitigation

1. **VPS Compromise**: SSH keys not stored on VPS; can abandon and reprovision
2. **Credential Theft**: Private key download logged; one-time download only
3. **Unauthorized Access**: Vault access control enforces tenant isolation
4. **Key Exposure**: Automated rotation limits exposure window to 90 days
5. **Insider Threats**: Audit trail tracks all SSH key access

## Compliance

### SOC 2 Type II

- **Security**: Access control, encryption, audit logging
- **Availability**: High availability Vault cluster, disaster recovery
- **Confidentiality**: Encryption at rest and in transit

### ISO 27001

- **A.9 Access Control**: Role-based access control
- **A.10 Cryptography**: RSA 4096-bit encryption
- **A.12 Operations Security**: Automated rotation, backup

### GDPR

- **Article 32 - Security of Processing**: Encryption, access control
- **Article 33 - Breach Notification**: Audit trail, monitoring

## Roadmap

### MVP (Q1 2026)

- [ ] Vault integration for SSH key storage
- [ ] Dual SSH key system (admin + tenant)
- [ ] Tenant SSH key download UI
- [ ] Automated 90-day rotation
- [ ] Audit logging for all SSH key operations
- [ ] Connection instructions in UI

### Post-MVP (Q2 2026)

- [ ] SSH key passphrase encryption
- [ ] Multi-factor authentication for key download
- [ ] SSH session recording (audit trail)
- [ ] Custom rotation policies per tenant
- [ ] SSH key usage analytics

### Future (Q3 2026+)

- [ ] SSH certificate authority (short-lived certificates)
- [ ] Just-in-time SSH access (temporary keys)
- [ ] SSH bastion host integration
- [ ] Hardware security module (HSM) integration
- [ ] FIDO2/WebAuthn for SSH authentication

## Related Documentation

- **[Overview](overview)** - Vault SSH key management overview
- **[Key Management](key-management)** - Dual key system and rotation details
- **[User Workflows](user-workflows)** - Tenant and admin workflow guides
- **[HashiCorp Vault Documentation](https://www.vaultproject.io/docs)** - Official Vault docs
- **[SSH Best Practices](https://www.ssh.com/academy/ssh/best-practices)** - SSH security guide
