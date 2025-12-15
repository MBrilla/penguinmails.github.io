---
title: "Security Implementation"
last_modified_date: "2025-12-15"
level: "3"
persona: "Developers"
---

# Security Implementation

End-to-end encryption and access control for GDPR compliance.

## End-to-End Encryption

```javascript
// encryption-service.js
import crypto from 'crypto';

export class EncryptionService {
    constructor() {
        this.algorithm = 'aes-256-gcm';
        this.key = process.env.ENCRYPTION_KEY;
    }

    encrypt(data) {
        const iv = crypto.randomBytes(16);
        const cipher = crypto.createCipher(this.algorithm, this.key);
        cipher.setAAD(Buffer.from('penguinmails-gdpr'));

        let encrypted = cipher.update(JSON.stringify(data), 'utf8', 'hex');
        encrypted += cipher.final('hex');

        const authTag = cipher.getAuthTag();

        return {
            encryptedData: encrypted,
            iv: iv.toString('hex'),
            authTag: authTag.toString('hex')
        };
    }

    decrypt(encryptedPackage) {
        const decipher = crypto.createDecipher(this.algorithm, this.key);
        decipher.setAAD(Buffer.from('penguinmails-gdpr'));
        decipher.setAuthTag(Buffer.from(encryptedPackage.authTag, 'hex'));

        let decrypted = decipher.update(encryptedPackage.encryptedData, 'hex', 'utf8');
        decrypted += decipher.final('utf8');

        return JSON.parse(decrypted);
    }
}
```

## Access Control Implementation

```javascript
// access-control.js
export class AccessControlService {
    constructor() {
        this.roles = {
            admin: ['*'],
            compliance_officer: ['consent_read', 'audit_read', 'compliance_write'],
            campaign_manager: ['campaign_read', 'campaign_write', 'consent_verify'],
            support_agent: ['contact_read', 'contact_update', 'rights_process']
        };
    }

    async checkPermission(userId, resource, action) {
        const user = await this.getUser(userId);
        const userRoles = user.roles;

        for (const role of userRoles) {
            const permissions = this.roles[role] || [];
            if (permissions.includes('*') || permissions.includes(`${resource}_${action}`)) {
                return true;
            }
        }

        return false;
    }

    async logAccess(userId, resource, action, result) {
        await this.database.query(`
            INSERT INTO access_audit_log (user_id, resource, action, result, timestamp)
            VALUES ($1, $2, $3, $4, $5)
        `, [userId, resource, action, result, new Date()]);
    }
}
```

---

[← Back to Technical Implementation](./README)
