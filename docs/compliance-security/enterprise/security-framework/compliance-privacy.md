---
title: "Compliance & Privacy"
last_modified_date: "2025-12-15"
level: "2"
persona: "Documentation Users"
---

# Compliance & Privacy

GDPR compliance and data retention policies.

## GDPR Compliance

### Data Processing Rights

```javascript
// GDPR compliance functions
const gdprCompliance = {
  // Right to be forgotten
  deleteUserData: async (userId, tenantId) => {
    await nileDB.transaction(async (trx) => {
      await trx('user_sessions').where({ user_id: userId }).del();
      await trx('tenant_users').where({ user_id: userId, tenant_id: tenantId }).del();
      await trx('users').where({ id: userId }).del();
    });
  },

  // Data export
  exportUserData: async (userId, tenantId) => {
    const userData = {
      profile: await nileDB('users').where({ id: userId }).first(),
      tenant_access: await nileDB('tenant_users').where({ user_id: userId }),
      activity_log: await nileDB('audit_log').where({ user_id: userId })
    };

    return userData;
  },

  // Data portability
  exportDataJSON: (data) => {
    return JSON.stringify(data, null, 2);
  }
};
```

## Data Retention Policies

### Retention Schedule

| Data Type | Retention Period | Deletion Method |
|-----------|------------------|-----------------|
| User Sessions | 30 days | Automated purge |
| Audit Logs | 7 years | Secure deletion |
| Email Campaigns | 3 years | Tenant option |
| Payment Records | 7 years | Regulatory compliance |
| System Logs | 90 days | Automated rotation |

---

[← Back to Security Framework](./README)
