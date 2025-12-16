---
title: "SSH Key Management Features"
description: "Dual SSH key system, secure download, automated rotation, and key revocation"
last_modified_date: "2025-11-26"
level: 2
persona: "Technical Teams, Operations"
keywords: ["ssh-keys", "key-rotation", "key-revocation", "secure-download"]
---

# SSH Key Management Features

Detailed documentation of SSH key management capabilities including the dual key system, secure download process, automated rotation, and revocation workflows.

## Dual SSH Key System

### Admin SSH Keys (PenguinMails Access)

- Generated during VPS provisioning
- Stored in Vault at `/vps/{tenant_id}/admin_ssh`
- Used by PenguinMails for infrastructure management
- Never exposed to tenants
- Rotated every 90 days automatically

### Tenant SSH Keys (Customer Access)

- Generated during VPS provisioning
- Stored in Vault at `/vps/{tenant_id}/tenant_ssh`
- Downloadable by tenant owner/admin
- Used for programmatic VPS access
- Rotated every 90 days automatically

## Secure Key Download

### One-Time Download Process

- Tenant can download private key once from UI
- Download triggers audit log entry
- Warning message: "Store this key securely. It will not be shown again."
- Key never displayed in UI after initial download

### Download Implementation

```typescript
// User clicks "Download Private Key"
POST /api/v1/tenant/infrastructure/ssh-credentials/download

// Backend retrieves key from Vault
const privateKey = await vault.read(`vps/${tenantId}/tenant_ssh/private_key`);

// Log download event
await auditLog.create({
  action: 'ssh_key_download',
  tenantId,
  userId,
  timestamp: new Date(),
  ipAddress: req.ip
});

// Return key as downloadable file
res.download('tenant-ssh-key.pem', privateKey);
```

## Automated Key Rotation

### Rotation Schedule

- SSH keys rotated every 90 days
- Cron job checks rotation policy daily
- Manual rotation trigger available in UI

### Rotation Workflow

```text
Cron job checks last_rotated timestamp
  ↓
If rotation due (current_date - last_rotated >= 90 days):
  ↓
Generate new RSA 4096-bit key pair
  ↓
Store new key in Vault with incremented version
  ↓
Add new public key to VPS authorized_keys
  ↓
Keep old key active for 24-hour grace period
  ↓
Remove old key from authorized_keys after grace period
  ↓
Send notification to tenant (email)
  ↓
Log rotation event in audit trail
```

### Grace Period

- 24-hour grace period prevents service disruption
- Both old and new keys valid during grace period
- Tenants notified to update their SSH clients

## Key Revocation and Regeneration

### Revoke SSH Access

- Tenant can revoke SSH access immediately
- Removes public key from VPS authorized_keys
- Marks key as revoked in Vault
- No grace period (immediate revocation)

### Regenerate SSH Key

- Tenant can regenerate new SSH key after revocation
- Generates new RSA 4096-bit key pair
- Stores new key in Vault
- Adds new public key to VPS authorized_keys
- Tenant must download new private key

## Audit Logging

### Logged Events

- SSH key download (who, when, IP address)
- SSH key rotation (automated or manual)
- SSH key revocation
- SSH key regeneration
- Failed SSH authentication attempts (from VPS logs)

### Audit Log Format

```json
{
  "timestamp": "2025-11-26T10:30:00.000Z",
  "action": "ssh_key_download",
  "tenantId": "550e8400-e29b-41d4-a716-446655440000",
  "userId": "user-123",
  "ipAddress": "203.0.113.42",
  "userAgent": "Mozilla/5.0...",
  "status": "success"
}
```

## Related Documentation

- **[Overview](overview)** - Vault SSH key management overview
- **[User Workflows](user-workflows)** - Tenant and admin workflow guides
- **[Technical & Security](technical-security)** - Architecture, API, and compliance
