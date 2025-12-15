---
title: "Application Security"
last_modified_date: "2025-12-15"
level: "2"
persona: "Documentation Users"
---

# Application Security

Input validation, XSS prevention, and rate limiting for the application layer.

## Input Validation & Sanitization

### SQL Injection Prevention

```javascript
// Parameterized queries
const getUserData = async (userId, tenantId) => {
  const query = `
    SELECT * FROM users
    WHERE id = $1 AND tenant_id = $2
  `;

  const result = await nileDB.query(query, [userId, tenantId]);
  return result.rows[0];
};

// Query builder with validation
const campaignQuery = nileDB('campaigns')
  .where({ tenant_id: tenantId, status: 'active' })
  .select(['id', 'name', 'status']);
```

### XSS Prevention

```javascript
// Input sanitization
const sanitizeInput = (input) => {
  return input
    .replace(/[<>'"]/g, '') // Remove dangerous characters
    .trim()
    .substring(0, 1000); // Limit length
};

// Output encoding
const escapeHTML = (unsafe) => {
  return unsafe
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#039;');
};
```

## Rate Limiting

### API Rate Limiting

```javascript
// Redis-based rate limiter
const rateLimiter = {
  check: async (identifier, limit, window) => {
    const key = `rate_limit:${identifier}`;
    const current = await redis.get(key) || 0;

    if (current >= limit) {
      return { allowed: false, remaining: 0 };
    }

    await redis.multi()
      .incr(key)
      .expire(key, window)
      .exec();

    return { allowed: true, remaining: limit - current - 1 };
  }
};

// Middleware usage
app.use('/api', async (req, res, next) => {
  const identifier = `${req.ip}:${req.user?.id || 'anonymous'}`;
  const result = await rateLimiter.check(identifier, 100, 3600); // 100 requests per hour

  if (!result.allowed) {
    return res.status(429).json({ error: 'Rate limit exceeded' });
  }

  res.setHeader('X-RateLimit-Remaining', result.remaining);
  next();
});
```

---

[← Back to Security Framework](./README)
