---
title: "Quick Setup Guide"
last_modified_date: "2025-12-15"
level: "1"
persona: "All Users"
---

# Quick Setup Guide

5-minute setup process for professional email infrastructure.

## Step 1: Domain Verification

```text
1. Add your domain to PenguinMails
2. System generates DNS verification record
3. Add TXT record to your DNS provider
4. Click "Verify Domain" (usually instant)
```

**Required Information:**
- Domain name (e.g., `yourdomain.com`)
- DNS provider access (for record creation)

## Step 2: Infrastructure Provisioning

Once domain is verified, click **"Launch Infrastructure"**:

```text
[Automated Process - 2-3 minutes]
✓ VPS server provisioned via Hostwind API
✓ MailU SMTP server installed and configured
✓ SSL certificates generated (Let's Encrypt)
✓ Firewall rules configured
✓ Initial system hardening applied
```

**What Happens Behind the Scenes:**
- VPS created with Ubuntu LTS + optimized email settings
- MailU installed with production configuration
- Port 25, 465, 587, 993 opened and secured
- Fail2ban installed for brute-force protection

## Step 3: DNS Configuration

System displays required DNS records:

```text
Add these records to your DNS provider:

MX Record:
  Host: @
  Value: mail.yourdomain.com
  Priority: 10

A Record:
  Host: mail
  Value: [Your VPS IP]

TXT Record (SPF):
  Host: @
  Value: v=spf1 ip4:[Your VPS IP] ~all

TXT Record (DKIM):
  Host: default._domainkey
  Value: [Generated DKIM public key]

TXT Record (DMARC):
  Host: _dmarc
  Value: v=DMARC1; p=quarantine; rua=mailto:postmaster@yourdomain.com
```

**Automation Option:** For supported DNS providers (Cloudflare, Route53, etc.), click **"Auto-Configure DNS"** to apply all records automatically.

## Step 4: Create Email Account

```text
1. Click "Create Email Account"
2. Enter email address (e.g., sales@yourdomain.com)
3. Set secure password (or auto-generate)
4. Click "Create Account"
```

**Result:** Email account ready to send/receive in 30 seconds.

## Step 5: Verification & Testing

System automatically runs verification:

```text
✓ SMTP connection test
✓ SPF record validation
✓ DKIM signature validation
✓ DMARC policy check
✓ SSL certificate validation
✓ Deliverability score (initial)
```

**Test Email:** Send test email to verify configuration.

## Success Criteria

Infrastructure setup is complete when:

- ✅ All DNS records validated
- ✅ SMTP server responding on ports 25, 465, 587
- ✅ SSL certificate valid
- ✅ Test email delivered successfully
- ✅ Deliverability score > 80%

> **Important**: Infrastructure is just the first step. Perfect DNS records won't save you if your sending behavior is flagged.

---

[← Back to Email Infrastructure Setup](./README)
