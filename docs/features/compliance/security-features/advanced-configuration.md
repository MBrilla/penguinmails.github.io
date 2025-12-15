---
title: "Advanced Security Configuration"
last_modified_date: "2025-12-15"
level: "2"
persona: "Documentation Users"
---

# Advanced Security Configuration

Firewall, session security, and audit logging for production environments.

## Firewall & Network Security

### VPS Firewall Rules

**Allowed Inbound:**

```text
Port 25   (SMTP)       - Email reception
Port 465  (SMTPS)      - Secure SMTP
Port 587  (Submission) - Email submission
Port 993  (IMAPS)      - Secure IMAP
Port 443  (HTTPS)      - Web traffic
Port 22   (SSH)        - Admin access (restricted IPs only)
```

**Blocked by Default:**

- All other ports
- Outbound connections restricted to necessary services

**IP Whitelisting:**

- SSH access limited to admin IPs
- API access can be IP-restricted per tenant
- SMTP relay can be IP-restricted

### DDoS Protection

**Rate Limiting:**

```typescript
// API endpoints
- 100 requests/minute per IP (anonymous)
- 1000 requests/minute per authenticated user
- 10,000 requests/minute per API key

// SMTP
- 50 connections/minute per IP
- 500 emails/hour per account (configurable)
```

**Fail2ban Configuration:**

- Auto-ban after 5 failed auth attempts
- Ban duration: 1 hour (first), 24 hours (repeat)
- Email notification on bans

## Advanced Authentication Security

### Session Security

```typescript
session: {
  secret: process.env.SESSION_SECRET, // Cryptographically random
  maxAge: 24 * 60 * 60 * 1000,       // 24 hours
  rolling: true,                      // Extend on activity
  httpOnly: true,                     // Prevent XSS
  secure: true,                       // HTTPS only
  sameSite: 'strict',                 // CSRF protection
}
```

**Session Management:**

- Automatic logout after 24 hours inactivity
- Concurrent session limit (5 per user)
- Session revocation on password change
- "Logout all devices" functionality

### Password Policies

**Requirements:**

- Minimum 12 characters
- Must contain: uppercase, lowercase, number, special character
- Cannot contain email address or common words
- Password history: prevent reuse of last 5 passwords
- Maximum age: 90 days (configurable)

**Breach Detection:**

- Check against Have I Been Pwned database
- Warn users of compromised passwords
- Force reset if breach detected

### Two-Factor Authentication (2FA)

**Status:** Planned (Level 4 - Enterprise Features)

**Future Implementation:**

- TOTP (Time-based One-Time Password)
- SMS verification
- Email verification codes
- Hardware security keys (U2F/WebAuthn)

## Audit Logging

### What Gets Logged

**User Actions:**

- Login/logout events
- Password changes
- Profile updates
- Permission changes
- Resource creation/deletion

**Email Events:**

- Email sent/received
- Campaign launches
- Template modifications
- Domain changes

**Security Events:**

- Failed login attempts
- API authentication failures
- Permission denied events
- Suspicious activity detection

**Infrastructure Events:**

- VPS provisioning/deletion
- DNS record changes
- SSL certificate renewals
- Service restarts

### Log Structure

```typescript
interface AuditLog {
  id: string;
  tenantId: string;
  userId?: string;
  timestamp: Date;
  eventType: string;
  eventCategory: string;
  action: string;
  resource: string;
  resourceId?: string;
  ipAddress: string;
  userAgent: string;
  requestId: string;
  changeset?: {
    before: object;
    after: object;
  };
  severity: 'info' | 'warning' | 'critical';
  tags: string[];
}
```

### Log Retention

| Tier | Retention |
|------|-----------|
| Default | 90 days |
| Enterprise | 1 year |
| Compliance Mode | 7 years (GDPR/CCPA) |

### Log Access

- Tenant admins can view own tenant logs
- Platform admins can view all logs
- Export to CSV/JSON available
- Integration with SIEM tools (Enterprise)

## Security Headers

All HTTP responses include security headers:

```text
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Frame-Options: DENY
X-Content-Type-Options: nosniff
X-XSS-Protection: 1; mode=block
Content-Security-Policy: default-src 'self'; script-src 'self' 'unsafe-inline'
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: geolocation=(), microphone=(), camera=()
```

## Vulnerability Scanning

**Automated Scans:**

- Weekly dependency vulnerability scans (npm audit, Snyk)
- Monthly infrastructure scans
- Quarterly penetration testing (planned)

**Security Updates:**

- Critical patches applied within 24 hours
- High-priority patches within 7 days
- Regular updates within 30 days

---

[← Back to Security Features](./README)
