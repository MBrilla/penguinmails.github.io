---
title: "SSH Key User Workflows"
description: "User workflows for tenant SSH key management and admin monitoring"
last_modified_date: "2025-11-26"
level: 2
persona: "Technical Teams, Operations"
keywords: ["user-workflows", "ssh-connection", "tenant-access", "admin-monitoring"]
---

# SSH Key User Workflows

Step-by-step workflows for tenants managing SSH access and administrators monitoring key health.

## Tenant Workflows

### Download SSH Key

1. Navigate to Settings → Infrastructure → SSH Access
2. View SSH credentials (VPS IP, username, port)
3. Click "Download Private Key" button
4. Read security warning
5. Confirm download
6. Save private key file securely
7. Follow connection instructions to connect to VPS

### Rotate SSH Key Manually

1. Navigate to Settings → Infrastructure → SSH Access
2. Click "Rotate SSH Key" button
3. Read grace period notice (24 hours)
4. Confirm rotation
5. Wait for rotation to complete (30 seconds)
6. Download new private key
7. Update SSH clients with new key
8. Old key remains valid for 24 hours

### Revoke and Regenerate SSH Key

1. Navigate to Settings → Infrastructure → SSH Access
2. Click "Revoke SSH Access" button
3. Confirm revocation (immediate, no grace period)
4. SSH access revoked
5. Click "Regenerate SSH Key" button
6. New key generated and stored in Vault
7. Download new private key
8. Connect to VPS with new key

## Admin Workflows

### Monitor SSH Key Health

1. Navigate to Admin Panel → Secrets Management
2. View Vault health dashboard
3. Search for tenant by ID or name
4. View SSH key status (active, expired, revoked)
5. Check last rotation date
6. Review audit logs for suspicious activity
7. Trigger bulk rotation if needed

## Connection Instructions

### UI Display

```bash
# SSH Connection Command
ssh -i /path/to/private-key.pem tenant-{tenant_id}@{vps_ip}

# Example
ssh -i ~/Downloads/tenant-ssh-key.pem tenant-550e8400@192.168.1.100
```

### Troubleshooting Tips

- Ensure private key has correct permissions: `chmod 600 private-key.pem`
- Verify VPS IP address is correct
- Check firewall allows SSH connections (port 22)
- Confirm SSH key hasn't been rotated (check UI for latest key)

## Best Practices

1. **Store Private Keys Securely**: Use password manager or encrypted storage
2. **Rotate Keys Regularly**: Don't disable automated rotation
3. **Monitor Audit Logs**: Review SSH key access logs monthly
4. **Revoke Compromised Keys**: Immediately revoke if key is exposed
5. **Use Strong Passphrases**: Encrypt private key with passphrase (optional)

## Related Documentation

- **[Overview](overview)** - Vault SSH key management overview
- **[Key Management](key-management)** - Dual key system and rotation details
- **[Technical & Security](technical-security)** - Architecture, API, and compliance
- **[Infrastructure SSH Access Routes](/docs/design/routes/infrastructure-ssh-access)** - Complete SSH UI routes
