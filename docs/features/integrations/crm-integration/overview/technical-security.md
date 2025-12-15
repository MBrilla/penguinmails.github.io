---
title: "Technical Stack & Security"
description: "Components, encryption, compliance, and security for CRM integration"
last_modified_date: "2025-01-20"
level: 3
persona: Developers
keywords: security, encryption, OAuth, compliance, GDPR
---

# Technical Stack & Security

This document covers the technical components, security measures, and compliance requirements for CRM integrations.

## Components

- **Backend**: Node.js integration microservice
- **OAuth**: Passport.js with OAuth 2.0 strategies
- **Database**: PostgreSQL for integration credentials and mappings
- **Queue**: Background job processing for batch syncs
- **Webhooks**: Real-time event processing from CRMs
- **Encryption**: AES-256 for credential storage

## Security Measures

- OAuth 2.0 for secure authentication
- Encrypted credential storage (AES-256)
- Scoped API permissions (minimal access)
- Audit logging for all sync operations
- GDPR compliance for data handling
- Token rotation every 30 days
- Webhook signature verification

## OAuth Token Management

```typescript
class OAuthTokenManager {
  private readonly encryptionKey: string;

  async storeTokens(
    integrationId: string,
    accessToken: string,
    refreshToken: string,
    expiresAt: Date
  ): Promise<void> {
    const encryptedAccess = await this.encrypt(accessToken);
    const encryptedRefresh = await this.encrypt(refreshToken);

    await db.integrations.update(integrationId, {
      accessTokenEncrypted: encryptedAccess,
      refreshTokenEncrypted: encryptedRefresh,
      tokenExpiresAt: expiresAt,
    });
  }

  async refreshTokenIfNeeded(integrationId: string): Promise<string> {
    const integration = await db.integrations.findById(integrationId);

    // Refresh if expires within 5 minutes
    const expiresIn = integration.tokenExpiresAt.getTime() - Date.now();
    if (expiresIn < 5 * 60 * 1000) {
      return await this.performTokenRefresh(integration);
    }

    return await this.decrypt(integration.accessTokenEncrypted);
  }

  private async performTokenRefresh(integration: Integration): Promise<string> {
    const refreshToken = await this.decrypt(integration.refreshTokenEncrypted);
    
    const newTokens = await this.oauthClient.refreshToken(refreshToken);
    
    await this.storeTokens(
      integration.id,
      newTokens.accessToken,
      newTokens.refreshToken,
      new Date(Date.now() + newTokens.expiresIn * 1000)
    );

    return newTokens.accessToken;
  }
}
```

## Audit Logging

All sync operations are logged for compliance and debugging:

```typescript
interface SyncAuditLog {
  id: string;
  integrationId: string;
  action: 'sync_to_crm' | 'sync_from_crm' | 'conflict_resolved';
  recordType: 'contact' | 'lead' | 'activity';
  recordId: string;
  crmRecordId: string;
  changes: Record<string, { old: any; new: any }>;
  timestamp: Date;
  userId?: string;
}
```

## Data Privacy Compliance

### GDPR Requirements

- Right to deletion: CRM sync respects deletion requests
- Data portability: Export includes CRM sync status
- Consent tracking: Sync only consented contacts
- Audit trail: Complete sync history available

### Data Handling

- PII encrypted at rest and in transit
- Minimal data exposure to third parties
- User consent required before CRM connection
- Clear data flow documentation

## Rate Limiting

```typescript
const rateLimits = {
  salesforce: {
    apiCallsPerDay: 15000,
    apiCallsPerHour: 1000,
    batchSize: 200,
  },
  hubspot: {
    apiCallsPerDay: 250000,
    apiCallsPerSecond: 10,
    batchSize: 100,
  },
};

class RateLimiter {
  async checkLimit(crm: string): Promise<boolean> {
    const usage = await this.getUsage(crm);
    const limit = rateLimits[crm];
    
    return usage.callsToday < limit.apiCallsPerDay;
  }

  async waitForSlot(crm: string): Promise<void> {
    while (!(await this.checkLimit(crm))) {
      await sleep(1000);
    }
  }
}
```

## Related Documentation

- [Salesforce Integration](./salesforce-integration.md)
- [HubSpot Integration](./hubspot-integration.md)
- [Sync Logic](./sync-logic.md)
- [Setup & Roadmap](./setup-roadmap.md)
