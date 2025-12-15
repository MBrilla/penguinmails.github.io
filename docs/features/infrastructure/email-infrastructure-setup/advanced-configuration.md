---
title: "Advanced Configuration"
last_modified_date: "2025-12-15"
level: "2"
persona: "All Users"
---

# Advanced Configuration

VPS sizing, DNS best practices, and multi-workspace infrastructure options.

## VPS Configuration Options

### Server Sizing

| Size | Specs | Capacity |
|------|-------|----------|
| **Starter** | 1 CPU, 2GB RAM | Up to 5K emails/day |
| **Professional** | 2 CPU, 4GB RAM | Up to 25K emails/day |
| **Business** | 4 CPU, 8GB RAM | Up to 100K emails/day |
| **Enterprise** | Custom sizing | High-volume needs |

### Server Location

- Choose geographic region for optimal deliverability
- Consider GDPR data residency requirements
- Multi-region deployment available (Enterprise)

## SMTP Server Optimization

### Performance Tuning

```yaml
# Email sending limits
max_connections: 50
max_messages_per_connection: 100
connection_timeout: 300s
retry_attempts: 3
retry_delay: 60s

# Queue configuration
queue_lifetime: 5d
bounce_queue_lifetime: 5d
defer_transports: true
```

### Security Hardening

- TLS 1.2+ enforcement
- Strong cipher suite configuration
- SMTP authentication required
- IP-based access restrictions (optional)
- Rate limiting per account

## DNS Best Practices

### SPF Configuration Strategies

```text
# Simple (single server)
v=spf1 ip4:123.45.67.89 ~all

# Multiple servers
v=spf1 ip4:123.45.67.89 ip4:123.45.67.90 ~all

# Include third-party (e.g., Postmark for transactional)
v=spf1 ip4:123.45.67.89 include:spf.postmarkapp.com ~all
```

### DKIM Key Rotation

- Rotate DKIM keys quarterly for security
- Support for multiple simultaneous keys
- Automated rotation available (Enterprise)

### DMARC Policy Progression

```text
# Phase 1: Monitor only
v=DMARC1; p=none; rua=mailto:dmarc@yourdomain.com

# Phase 2: Quarantine (after 30 days)
v=DMARC1; p=quarantine; pct=10; rua=mailto:dmarc@yourdomain.com

# Phase 3: Reject (after 90 days)
v=DMARC1; p=reject; rua=mailto:dmarc@yourdomain.com
```

## Multi-Workspace Infrastructure

For agencies managing multiple clients:

### Workspace Isolation Options

**1. Shared VPS** (Cost-effective):
- Multiple domains on single VPS
- Separate DKIM keys per domain
- Shared IP reputation
- Best for: Startups, small agencies

**2. Dedicated VPS per Workspace** (Recommended):
- Complete isolation between clients
- Independent IP reputation
- Custom server configuration per workspace
- Best for: Agencies, enterprise clients

**3. IP Pooling** (Enterprise):
- Multiple IPs per VPS
- Intelligent IP rotation
- Dedicated IPs for high-volume senders
- Best for: High-volume sending

---

[← Back to Email Infrastructure Setup](./README)
