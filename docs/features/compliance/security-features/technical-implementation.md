---
title: "Technical Implementation"
last_modified_date: "2025-12-15"
level: "4"
persona: "Developers"
---

# Technical Implementation

Code examples and middleware for security implementation.

## Database Encryption

### Field-Level Encryption

```typescript
import { encrypt, decrypt } from '@/lib/encryption';

class User {
  @Column()
  email: string;

  @Column({ type: 'text' })
  private _passwordHash: string;

  @Column({ type: 'text', nullable: true })
  private _apiKey: string;

  get apiKey(): string | null {
    return this._apiKey ? decrypt(this._apiKey) : null;
  }

  set apiKey(value: string | null) {
    this._apiKey = value ? encrypt(value) : null;
  }

  async setPassword(password: string): Promise<void> {
    const isBreached = await checkPasswordBreach(password);
    if (isBreached) {
      throw new Error('Password found in breach database');
    }
    this._passwordHash = await bcrypt.hash(password, 12);
  }

  async verifyPassword(password: string): Promise<boolean> {
    return bcrypt.compare(password, this._passwordHash);
  }
}
```

### Encryption Key Management

```typescript
import { pbkdf2Sync, randomBytes } from 'crypto';

const MASTER_KEY = process.env.ENCRYPTION_MASTER_KEY;

function deriveKey(purpose: string, salt?: Buffer): Buffer {
  const keySalt = salt || randomBytes(32);
  return pbkdf2Sync(MASTER_KEY, keySalt, 100000, 32, 'sha256');
}

function encrypt(data: string): string {
  const iv = randomBytes(16);
  const key = deriveKey('field-encryption');
  const cipher = crypto.createCipheriv('aes-256-gcm', key, iv);

  let encrypted = cipher.update(data, 'utf8', 'hex');
  encrypted += cipher.final('hex');

  const authTag = cipher.getAuthTag();
  return iv.toString('hex') + ':' + authTag.toString('hex') + ':' + encrypted;
}

function decrypt(encryptedData: string): string {
  const [ivHex, authTagHex, encrypted] = encryptedData.split(':');

  const iv = Buffer.from(ivHex, 'hex');
  const authTag = Buffer.from(authTagHex, 'hex');
  const key = deriveKey('field-encryption');

  const decipher = crypto.createDecipheriv('aes-256-gcm', key, iv);
  decipher.setAuthTag(authTag);

  let decrypted = decipher.update(encrypted, 'hex', 'utf8');
  decrypted += decipher.final('utf8');

  return decrypted;
}
```

## SMTP Security Implementation

### TLS Configuration

```nginx
# Postfix configuration for secure SMTP
smtpd_tls_security_level = may
smtpd_tls_cert_file = /etc/letsencrypt/live/mail.domain.com/fullchain.pem
smtpd_tls_key_file = /etc/letsencrypt/live/mail.domain.com/privkey.pem
smtpd_tls_protocols = !SSLv2, !SSLv3, !TLSv1, !TLSv1.1
smtpd_tls_ciphers = high
smtpd_tls_mandatory_ciphers = high
smtpd_tls_mandatory_protocols = !SSLv2, !SSLv3, !TLSv1, !TLSv1.1
smtpd_tls_auth_only = yes

# Client TLS (outbound)
smtp_tls_security_level = may
smtp_tls_protocols = !SSLv2, !SSLv3, !TLSv1, !TLSv1.1
smtp_tls_ciphers = high
```

### DKIM Signing

```typescript
import { DKIMSigner } from 'dkim-signer';

async function signEmail(email: Email, domain: Domain): Promise<string> {
  const dkimKey = await getDKIMPrivateKey(domain.id);

  const signer = new DKIMSigner({
    domainName: domain.name,
    selector: 'default',
    privateKey: dkimKey.privateKey,
    headerCanonicalization: 'relaxed',
    bodyCanonicalization: 'relaxed',
  });

  return await signer.sign(email.raw);
}

async function generateDKIMKeys(domainId: string): Promise<DKIMKeyPair> {
  const { publicKey, privateKey } = await generateKeyPair('rsa', {
    modulusLength: 2048,
    publicKeyEncoding: { type: 'spki', format: 'pem' },
    privateKeyEncoding: { type: 'pkcs8', format: 'pem' },
  });

  await db.dkimKeys.create({
    domainId,
    selector: 'default',
    publicKey,
    privateKeyEncrypted: encrypt(privateKey),
    createdAt: new Date(),
    expiresAt: addMonths(new Date(), 3),
  });

  return { publicKey, privateKey };
}
```

## API Security Middleware

### Authentication Middleware

```typescript
import { Request, Response, NextFunction } from 'express';
import { verifyJWT, verifyAPIKey } from '@/lib/auth';

export async function authenticate(
  req: Request,
  res: Response,
  next: NextFunction
) {
  try {
    const authHeader = req.headers.authorization;
    if (authHeader?.startsWith('Bearer ')) {
      const token = authHeader.substring(7);
      const user = await verifyJWT(token);
      req.user = user;
      return next();
    }

    const apiKey = req.headers['x-api-key'];
    if (apiKey) {
      const apiUser = await verifyAPIKey(apiKey as string);
      req.user = apiUser;
      return next();
    }

    return res.status(401).json({ error: 'Authentication required' });
  } catch (error) {
    logger.error('Authentication error:', error);
    return res.status(401).json({ error: 'Invalid credentials' });
  }
}
```

### Rate Limiting Middleware

```typescript
import rateLimit from 'express-rate-limit';
import RedisStore from 'rate-limit-redis';

export const apiLimiter = rateLimit({
  store: new RedisStore({
    client: redisClient,
    prefix: 'rl:api:',
  }),
  windowMs: 60 * 1000,
  max: async (req) => {
    if (req.user?.apiKey) return 10000;
    if (req.user) return 1000;
    return 100;
  },
  message: 'Too many requests, please try again later',
  standardHeaders: true,
  legacyHeaders: false,
});

export const authLimiter = rateLimit({
  store: new RedisStore({
    client: redisClient,
    prefix: 'rl:auth:',
  }),
  windowMs: 15 * 60 * 1000,
  max: 5,
  skipSuccessfulRequests: true,
  message: 'Too many login attempts, please try again later',
});
```

### Security Headers Middleware

```typescript
import helmet from 'helmet';

export const securityHeaders = helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", "'unsafe-inline'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      imgSrc: ["'self'", 'data:', 'https:'],
      connectSrc: ["'self'"],
      fontSrc: ["'self'"],
      objectSrc: ["'none'"],
      mediaSrc: ["'self'"],
      frameSrc: ["'none'"],
    },
  },
  hsts: {
    maxAge: 31536000,
    includeSubDomains: true,
    preload: true,
  },
  referrerPolicy: {
    policy: 'strict-origin-when-cross-origin',
  },
});
```

## Security Monitoring

### Suspicious Activity Detection

```typescript
async function detectSuspiciousActivity(event: AuditLog): Promise<void> {
  const checks = [
    checkBruteForce(event),
    checkRapidActions(event),
    checkUnusualLocation(event),
    checkUnusualTime(event),
    checkMassDelete(event),
  ];

  const alerts = (await Promise.all(checks)).filter(Boolean);

  if (alerts.length > 0) {
    await createSecurityAlert(event.tenantId, event.userId, alerts);
  }
}

async function checkBruteForce(event: AuditLog): Promise<SecurityAlert | null> {
  if (event.eventType !== 'user.login.failed') return null;

  const recentFailures = await db.auditLogs.count({
    where: {
      eventType: 'user.login.failed',
      ipAddress: event.ipAddress,
      timestamp: { gte: subMinutes(new Date(), 15) },
    },
  });

  if (recentFailures >= 5) {
    return {
      type: 'brute_force',
      severity: 'critical',
      message: `${recentFailures} failed login attempts from ${event.ipAddress}`,
    };
  }

  return null;
}
```

---

[← Back to Security Features](./README)
