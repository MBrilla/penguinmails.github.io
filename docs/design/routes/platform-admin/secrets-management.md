---
title: "Secrets Management Routes"
description: "Admin routes for Vault secrets, SSH keys, and credential management"
last_modified_date: "2025-12-16"
level: "3"
persona: "Technical Teams"
---

# Secrets Management Routes

## `/admin/secrets` - Vault Secrets Management

**User Story**: *"As a platform admin, I want to monitor Vault health and manage tenant secrets centrally, so I can ensure secure credential storage and respond quickly to security incidents."*

### Vault Health Dashboard

- **Status Cards** (Traffic light):
  - **Vault Seal Status**: Unsealed (Green) / Sealed (Red).
  - **Active Node**: vault-node-1 (Primary).
  - **Storage Backend**: PostgreSQL - Healthy.
  - **Last Backup**: 2 hours ago (Green) / 25 hours ago (Yellow warning).
- **Quick Actions**:
  - **"Force Backup Now" Button**: Manually trigger Vault snapshot.
  - **"View Backup History" Link**: Shows last 30 backups with restore points.

### Tenant Secrets Overview

- **Search Bar**: "Search by tenant ID, tenant name, or VPS IP".
- **Filters**:
  - **Secret Type**: All, SSH Keys, SMTP Credentials, API Keys, DKIM Keys.
  - **Status**: Active, Expired, Revoked.
  - **Rotation Due**: Next 7 days, Next 30 days, Overdue.
- **Tenants Table**:
  - **Columns**: Tenant ID, Tenant Name, VPS IP, SSH Key Status, SMTP Status, API Keys Count, DKIM Keys Count, Last Rotation, Actions.
  - **Color Coding**: Expired (Red), Rotation due (Yellow), Healthy (Green).

### Secret Details View

- **SSH Keys Section**:
  - Admin/Tenant SSH Key status, Last Rotated, Key Fingerprint.
  - **Actions**: "Rotate Now", "View Rotation History".
- **SMTP Credentials Section**:
  - Admin Username (masked), Webmail URL, Last Rotated.
  - **Actions**: "View Credentials", "Rotate Now", "Emergency Reset".
- **API Keys Section**:
  - Table: Key ID, Permissions, Rate Limit, Last Used.
  - **Actions**: "Revoke Key", "View Usage Analytics".
- **DKIM Keys Section**:
  - Table: Domain, Selector, Last Rotated, Status.
  - **Actions**: "Rotate Now", "View DNS Records".

### Rotation Management

- **Bulk Rotation Panel**:
  - "Rotate All SSH Keys", "Rotate All SMTP Credentials", "Rotate Overdue Secrets".
- **Rotation Schedule Configuration**:
  - SSH Keys: 90 days (default)
  - SMTP Credentials: 180 days (default)
  - DKIM Keys: 365 days (default)
- **Grace Period Settings**:
  - SSH Keys: 24 hours (both old and new keys valid)
  - DKIM Keys: 48 hours (DNS propagation time)

### SMTP Credentials Viewing Modal

**Security Requirements**:
- Re-authentication Required (password + 2FA)
- Time-Limited Access: 15 minutes only
- Audit Logging: All credential views logged

**Modal Components**:
1. Re-authentication step with password + 2FA
2. Credentials display with countdown timer
3. Auto-hide after 15 minutes

### Audit Log Viewer

- **Real-time Audit Stream**:
  - Columns: Timestamp, Action, Tenant, User, Secret Type, IP Address, Status.
  - Auto-refresh every 5 seconds.
- **Filters**: Tenant, Secret Type, Action, Date Range, User.
- **Suspicious Activity Alerts**:
  - Multiple failed authentication attempts
  - Access outside normal hours
  - Bulk secret reads
- **Export**: CSV and JSON formats.

**Related Documentation**:
- [Vault Integration Architecture](/docs/design/routes/infrastructure-ssh-access)
- [Vault SSH Management Feature](/docs/features/infrastructure/vault-ssh-management)
- [Security Compliance](/docs/compliance-security/enterprise/infrastructure-security)

**Technical Integration**:
- **HashiCorp Vault**: Centralized secrets storage with KV v2 engine.
- **Access Control**: Role-based policies.
- **Audit Logging**: All operations logged to audit device.
- **Backup System**: Automated daily snapshots to S3 with AES-256-GCM encryption.
- **Monitoring**: Prometheus metrics, PagerDuty alerts.
