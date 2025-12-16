---
title: "Vault SSH Key Management Overview"
description: "Overview of secure SSH credential management for VPS infrastructure using HashiCorp Vault"
last_modified_date: "2025-11-26"
level: 2
persona: "Technical Teams, Operations"
status: "BACKLOG"
priority: "HIGH"
roadmap_milestone: "Q1_2026"
keywords: ["vault", "ssh", "security", "infrastructure", "credentials"]
---

# Vault SSH Key Management Overview

Vault SSH Key Management provides secure, centralized storage and automated rotation of SSH credentials for tenant VPS infrastructure. By storing SSH keys in HashiCorp Vault instead of ENV files on VPS instances, PenguinMails enables rapid disaster recovery and eliminates credential exposure risks from VPS compromise.

## Problem Statement

Traditional SSH key management approaches create security vulnerabilities:

- **ENV File Exposure**: SSH keys stored in ENV files on VPS are exposed if VPS is compromised
- **Manual Rotation**: Rotating SSH keys requires manual updates across multiple VPS instances
- **No Audit Trail**: Difficult to track who accessed SSH keys and when
- **Disaster Recovery**: VPS compromise requires manual credential recovery and rotation
- **Scattered Management**: SSH keys managed individually per VPS instead of centrally

## Solution

Vault SSH Key Management centralizes SSH credential storage and automates rotation:

1. **Centralized Storage**: All SSH keys stored in Vault, not on VPS
2. **Automated Rotation**: SSH keys automatically rotated every 90 days with 24-hour grace period
3. **Audit Logging**: All SSH key access logged with timestamp, user, and action
4. **Rapid Recovery**: VPS compromise doesn't expose secrets; abandon VPS and provision new one
5. **Tenant Access**: Tenants can download their SSH private key for VPS access

## Key Features Summary

### Dual SSH Key System

Separate admin and tenant SSH keys provide isolation between PenguinMails infrastructure management and customer VPS access.

### Secure Key Download

One-time private key download with audit logging ensures credentials are only accessed when needed.

### Automated Key Rotation

SSH keys automatically rotated every 90 days with 24-hour grace period for seamless transitions.

### Key Revocation and Regeneration

Immediate revocation capability with easy regeneration for compromised credentials.

### Comprehensive Audit Logging

All SSH key operations logged including downloads, rotations, revocations, and failed authentication attempts.

## Feature Status

| Aspect | Details |
|--------|---------|
| Status | BACKLOG |
| Priority | HIGH (MVP dependency) |
| Roadmap | Q1 2026 |
| Estimated Effort | Large (10-15 days) |

## Related Documentation

- **[Key Management](key-management)** - Dual key system, rotation, and revocation details
- **[User Workflows](user-workflows)** - Tenant and admin workflow guides
- **[Technical & Security](technical-security)** - Architecture, API, and compliance
- **[Vault API Keys](/docs/features/integrations/vault-api-keys/overview)** - Tenant API key system
- **[Email Infrastructure Setup](/docs/features/infrastructure/email-infrastructure-setup)** - Infrastructure provisioning
