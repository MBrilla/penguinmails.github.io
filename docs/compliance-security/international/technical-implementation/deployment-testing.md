---
title: "Deployment & Testing"
last_modified_date: "2025-12-15"
level: "3"
persona: "DevOps engineers"
---

# Deployment & Testing

Docker configuration, testing, and performance optimization.

## Docker Configuration for GDPR Compliance

```dockerfile
# Dockerfile.gdpr-compliant
FROM node:18-alpine

# Install security updates
RUN apk update && apk upgrade

# Create non-root user for security
RUN addgroup -g 1001 -S nodejs && adduser -S penguinmails -u 1001

# Set up encrypted volumes for sensitive data
VOLUME ["/app/encrypted-data"]
VOLUME ["/app/logs/audit"]

COPY --chown=nodejs:nodejs . /app
WORKDIR /app

RUN npm ci --only=production

USER penguinmails

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD node healthcheck.js

EXPOSE 3000
CMD ["npm", "start"]
```

## Environment Configuration

```yaml
# docker-compose.gdpr.yml
version: '3.8'
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile.gdpr-compliant
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://user:pass@postgres:5432/penguinmails_gdpr
      - ENCRYPTION_KEY=${ENCRYPTION_KEY}
      - GDPR_COMPLIANCE_MODE=true
      - AUDIT_LOGGING=true
    volumes:
      - encrypted_data:/app/encrypted-data
      - audit_logs:/app/logs/audit

  postgres:
    image: postgres:15
    environment:
      - POSTGRES_DB=penguinmails_gdpr
      - POSTGRES_USER=gdpr_user
      - POSTGRES_PASSWORD=${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./sql/init-gdpr.sql:/docker-entrypoint-initdb.d/init.sql

volumes:
  encrypted_data:
  audit_logs:
  postgres_data:
```

## Compliance Testing Framework

```javascript
// compliance-test-suite.js
describe('GDPR Compliance Tests', () => {
    test('should verify consent before sending emails', async () => {
        const emailData = { to: 'test@example.com', subject: 'Test' };

        await expect(emailService.sendEmail(emailData))
            .rejects.toThrow('Consent verification failed');
    });

    test('should implement right to erasure', async () => {
        const userEmail = 'user@example.com';

        await dataRightsService.processErasureRequest(userEmail, 'user_requested');

        const remainingData = await database.query(`
            SELECT * FROM contacts WHERE email = $1
        `, [userEmail]);

        expect(remainingData.length).toBe(0);
    });

    test('should encrypt personal data at rest', async () => {
        const sensitiveData = { email: 'test@example.com', name: 'Test User' };
        const encrypted = encryptionService.encrypt(sensitiveData);

        expect(encrypted.encryptedData).toBeDefined();
        expect(encrypted.encryptedData).not.toContain('test@example.com');
    });
});
```

## Security Audit Procedures

```javascript
// security-audit.js
export class SecurityAudit {
    async performComplianceAudit() {
        const auditResults = {
            encryptionStatus: await this.checkEncryptionCompliance(),
            accessControlStatus: await this.checkAccessControlCompliance(),
            auditLoggingStatus: await this.checkAuditLoggingCompliance(),
            dataRetentionStatus: await this.checkDataRetentionCompliance(),
            consentManagementStatus: await this.checkConsentManagementCompliance()
        };

        return {
            overallCompliance: this.calculateOverallCompliance(auditResults),
            criticalIssues: this.identifyCriticalIssues(auditResults),
            recommendations: this.generateRecommendations(auditResults),
            auditDate: new Date().toISOString()
        };
    }
}
```

## Performance Optimization

### Database Optimization

```sql
-- Indexes for consent management performance
CREATE INDEX idx_contacts_consent_status ON contacts(consent_status, consent_timestamp);
CREATE INDEX idx_consent_history_email_timestamp ON consent_history(email, timestamp DESC);
CREATE INDEX idx_audit_logs_timestamp ON access_audit_log(timestamp DESC);

-- Partitioning for large audit logs
CREATE TABLE access_audit_log_2025 PARTITION OF access_audit_log
FOR VALUES FROM ('2025-01-01') TO ('2026-01-01');
```

### Caching Strategy

```javascript
// consent-cache.js
import Redis from 'ioredis';

export class ConsentCache {
    constructor() {
        this.redis = new Redis(process.env.REDIS_URL);
        this.cacheTTL = 3600; // 1 hour
    }

    async getCachedConsentStatus(email) {
        const cached = await this.redis.get(`consent:${email}`);
        if (cached) {
            return JSON.parse(cached);
        }

        const consentStatus = await this.consentService.verifyConsent(email);
        await this.redis.setex(
            `consent:${email}`,
            this.cacheTTL,
            JSON.stringify(consentStatus)
        );

        return consentStatus;
    }

    async invalidateConsentCache(email) {
        await this.redis.del(`consent:${email}`);
    }
}
```

---

[← Back to Technical Implementation](./README)
