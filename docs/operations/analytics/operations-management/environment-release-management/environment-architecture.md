---
title: "Environment Architecture"
description: Environment specifications for development, staging, and production
last_modified_date: 2025-01-15
level: 3
persona: ["devops", "operations"]
keywords: [environments, infrastructure, configuration, development, staging, production]
---

# Environment Architecture

Detailed specifications for each environment tier.

## Development Environment

```yaml
environment: development
purpose: Feature development and testing
infrastructure:
  - VPS: 2 vCPUs, 4GB RAM
  - Database: NileDB dev instance
  - Storage: Local file system
  - Networking: Internal only

data:
  - Source: Synthetic/anonymized data
  - Retention: 30 days
  - Backup: Daily snapshots

access:
  - Developers: Full access
  - QA: Read-only access
  - Security: Automated scanning
```

## Staging Environment

```yaml
environment: staging
purpose: Pre-production validation
infrastructure:
  - VPS: 4 vCPUs, 8GB RAM
  - Database: NileDB staging instance
  - Storage: Cloud storage with CDN
  - Networking: Restricted external access

data:
  - Source: Production-like data (anonymized)
  - Retention: 90 days
  - Backup: Hourly snapshots

access:
  - Developers: Limited access
  - QA: Full testing access
  - Product: Demo access
  - Security: Full monitoring
```

## Production Environment

```yaml
environment: production
purpose: Live user-facing system
infrastructure:
  - VPS: 8-16 vCPUs, 32-64GB RAM (auto-scaling)
  - Database: NileDB production cluster
  - Storage: Geo-redundant cloud storage
  - Networking: Global CDN with WAF

data:
  - Source: Live user data
  - Retention: Per data classification policy
  - Backup: Continuous replication

access:
  - Users: Application access
  - Support: Limited debugging access
  - Operations: Administrative access
  - Security: 24/7 monitoring
```

## Environment Comparison

| Aspect | Development | Staging | Production |
|--------|-------------|---------|------------|
| Purpose | Feature dev | Validation | Live system |
| Resources | 2 vCPU, 4GB | 4 vCPU, 8GB | 8-16 vCPU, 32-64GB |
| Data | Synthetic | Anonymized | Live |
| Backup | Daily | Hourly | Continuous |
| Access | Full for devs | Limited | Restricted |
