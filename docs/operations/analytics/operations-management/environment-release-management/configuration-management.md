---
title: "Configuration Management"
description: Environment configuration, secrets management, and feature flags
last_modified_date: 2025-01-15
level: 3
persona: ["devops", "developer"]
keywords: [configuration, secrets, feature-flags, environment-variables]
---

# Configuration Management

Environment configuration, secrets management, and feature flag implementation.

## Environment Configuration

```typescript
interface EnvironmentConfig {
  environment: string;
  database: DatabaseConfig;
  cache: CacheConfig;
  storage: StorageConfig;
  email: EmailConfig;
  monitoring: MonitoringConfig;
  security: SecurityConfig;
  features: FeatureFlags;
}

interface DatabaseConfig {
  host: string;
  port: number;
  database: string;
  ssl: boolean;
  connectionPool: {
    min: number;
    max: number;
    idleTimeoutMillis: number;
  };
}
```

## Secrets Management

```yaml
# Secret management strategy
secrets:
  strategy: 'vault'  # HashiCorp Vault
  rotation:
    automatic: true
    frequency: '30d'
    grace_period: '7d'
  access:
    principle: 'role-based'
    audit: true
    encryption: 'AES-256-GCM'
```

### Secret Categories

| Category | Rotation | Storage | Access Control |
|----------|----------|---------|----------------|
| API Keys | 30 days | Vault | Role-based |
| Database Credentials | 90 days | Vault | Service accounts |
| Encryption Keys | 365 days | HSM | Admin only |
| OAuth Secrets | 180 days | Vault | Application |

## Feature Flags

```typescript
interface FeatureFlag {
  name: string;
  description: string;
  enabled: boolean;
  rollout: {
    strategy: 'immediate' | 'gradual' | 'user_segment' | 'percentage';
    percentage?: number;
    userSegments?: string[];
    conditions?: FeatureCondition[];
  };
  monitoring: {
    metrics: string[];
    alerts: AlertConfig[];
  };
}

// Feature flag usage
const isFeatureEnabled = (flagName: string, userId?: string): boolean => {
  const flag = featureFlags[flagName];
  if (!flag || !flag.enabled) return false;

  switch (flag.rollout.strategy) {
    case 'immediate':
      return true;
    case 'percentage':
      return (hash(userId || 'anonymous') % 100) < flag.rollout.percentage!;
    case 'user_segment':
      return flag.rollout.userSegments!.includes(getUserSegment(userId));
    case 'gradual':
      return Date.now() > flag.rollout.conditions![0].date!;
    default:
      return false;
  }
};
```

## Rollout Strategies

| Strategy | Description | Use Case |
|----------|-------------|----------|
| Immediate | Enable for all users | Bug fixes, low-risk features |
| Percentage | Random sample of users | A/B testing |
| User Segment | Specific user groups | Beta programs |
| Gradual | Time-based expansion | High-risk features |

## Configuration Best Practices

1. **Environment Separation**: Never share secrets between environments
2. **Least Privilege**: Grant minimum necessary permissions
3. **Audit Logging**: Track all configuration access and changes
4. **Version Control**: Manage non-secret config in version control
5. **Validation**: Validate configuration at application startup
