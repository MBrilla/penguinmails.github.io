---
title: "Data Security"
last_modified_date: "2025-12-15"
level: "2"
persona: "Documentation Users"
---

# Data Security

Multi-tenant data isolation and encryption practices for the PenguinMails platform.

## Multi-Tenant Data Isolation

### Database Security

```sql
-- Row Level Security Example
ALTER TABLE tenant_data ENABLE ROW LEVEL SECURITY;

CREATE POLICY tenant_isolation ON tenant_data
  USING (tenant_id = current_setting('app.current_tenant_id')::bigint);

-- Session-based tenant context
SET app.current_tenant_id = '12345';
```

### API Security

```javascript
// Tenant context middleware
const setTenantContext = async (req, res, next) => {
  try {
    const tenantId = req.params.tenantId || req.user.default_tenant_id;

    // Verify user has access to this tenant
    const hasAccess = await nileDB.tenants.verifyAccess({
      userId: req.user.id,
      tenantId: tenantId
    });

    if (!hasAccess) {
      return res.status(403).json({ error: 'Tenant access denied' });
    }

    // Set tenant context for database queries
    await nileDB.query('SET app.current_tenant_id = $1', [tenantId]);

    req.tenantId = tenantId;
    next();
  } catch (error) {
    logger.error('Tenant context error:', error);
    res.status(500).json({ error: 'Failed to set tenant context' });
  }
};
```

## Data Encryption

### Encryption at Rest

- **Database**: PostgreSQL TDE (Transparent Data Encryption)
- **File Storage**: Encrypted backups and log files
- **Configuration**: Encrypted environment variables

### Encryption in Transit

```javascript
// HTTPS enforcement
const enforceHTTPS = (req, res, next) => {
  if (req.headers['x-forwarded-proto'] !== 'https') {
    const secureUrl = `https://${req.headers.host}${req.url}`;
    return res.redirect(301, secureUrl);
  }
  next();
};

// API security headers
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      scriptSrc: ["'self'"],
      imgSrc: ["'self'", "data:", "https:"],
    },
  },
}));
```

### API Key Management

```javascript
// Secure API key handling
const apiKeyManager = {
  generateKey: () => {
    return crypto.randomBytes(32).toString('hex');
  },

  hashKey: (key) => {
    return crypto.createHash('sha256').update(key).digest('hex');
  },

  encryptSensitive: (data) => {
    const cipher = crypto.createCipher('aes-256-gcm', process.env.ENCRYPTION_KEY);
    return cipher.update(JSON.stringify(data), 'utf8', 'hex') + cipher.final('hex');
  }
};
```

---

[← Back to Security Framework](./README)
