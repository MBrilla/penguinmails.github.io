---
title: "Email Authentication"
last_modified_date: "2025-12-15"
level: "1"
persona: "Documentation Users"
---

# Email Authentication (SPF, DKIM, DMARC)

Sender verification protocols that protect your domain reputation and ensure email deliverability.

## SPF (Sender Policy Framework)

SPF specifies which mail servers can send email on behalf of your domain.

**Setup** (Automated during infrastructure setup):

```text
TXT Record: @ or yourdomain.com
Value: v=spf1 ip4:YOUR_VPS_IP ~all
```

**Best Practices:**

- Use `~all` (soft fail) initially, upgrade to `-all` (hard fail) after testing
- Include all authorized sending IPs
- Keep SPF record under 255 characters
- Avoid more than 10 DNS lookups

## DKIM (DomainKeys Identified Mail)

DKIM provides cryptographic signature proving email authenticity.

**Setup** (Automated):

```text
TXT Record: default._domainkey.yourdomain.com
Value: v=DKIM1; k=rsa; p=YOUR_PUBLIC_KEY
```

**Best Practices:**

- Rotate DKIM keys quarterly
- Use 2048-bit keys minimum
- Monitor for validation failures
- Keep private keys secure and encrypted

## DMARC (Domain-based Message Authentication)

DMARC instructs receivers how to handle failed SPF/DKIM checks.

**Staged Rollout (Recommended):**

```text
Phase 1 (Monitoring):
TXT Record: _dmarc.yourdomain.com
Value: v=DMARC1; p=none; rua=mailto:dmarc@yourdomain.com

Phase 2 (Quarantine - after 30 days):
Value: v=DMARC1; p=quarantine; pct=10; rua=mailto:dmarc@yourdomain.com

Phase 3 (Reject - after 90 days):
Value: v=DMARC1; p=reject; rua=mailto:dmarc@yourdomain.com
```

**Best Practices:**

- Start with `p=none` to monitor
- Gradually increase `pct` (percentage)
- Review aggregate reports weekly
- Achieve 100% SPF+DKIM alignment before `p=reject`

## SSL/TLS Certificates

**Automatic Configuration:**

- Let's Encrypt SSL certificates auto-installed
- Auto-renewal 30 days before expiration
- TLS 1.2+ enforcement
- Strong cipher suite configuration

**Coverage:**

- SMTP connections (ports 465, 587)
- HTTPS for web dashboard
- API endpoints

**Monitoring:**

- 30-day expiration warning
- 7-day expiration alert
- 1-day expiration critical alert

---

[← Back to Security Features](./README)
