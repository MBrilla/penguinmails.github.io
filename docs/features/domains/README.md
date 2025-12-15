---
title: "Domain Management"
description: "Domain DNS management, queries, mutations, and sender authentication"
last_modified_date: "2025-12-16"
level: "1"
persona: "All Users"
---

# Domain Management

General domain DNS management, queries, mutations, and sender authentication.

## Purpose

Manage domain configuration, DNS settings, and sender authentication separately from warmup operations.

## MVP Status

## Domain Management is MVP-Complete

All essential domain verification, DNS management, and authentication features are implemented and documented. The platform provides:

- ✅ Complete domain verification workflow

- ✅ Comprehensive DNS record management (SPF, DKIM, DMARC, CNAME, MX)

- ✅ Industry-standard authentication protocols

- ✅ Real-time health monitoring and reputation tracking

- ✅ Multi-domain support per workspace

- ✅ Email account management per domain

- ✅ Custom tracking domains with SSL

- ✅ Full API access for programmatic management

**No MVP-blocking gaps identified.** See [Domain Management Roadmap](/docs/features/domains/roadmap) for Post-MVP enhancements.

**Last Reviewed:** November 26, 2025 - Comprehensive feature completeness review confirmed MVP-ready status

## Features in This Category

### Available Now

- **[Domain Management](/docs/features/domains/domain-management)** - Complete DNS verification and sender authentication

  - Custom domain addition and verification

  - Automated DNS record generation (SPF, DKIM, DMARC, CNAME, MX)

  - Multi-step wizard for domain setup

  - DNS status monitoring with real-time verification

  - Provider-specific setup guides (GoDaddy, Cloudflare, Namecheap)

- **Domain DNS Management** - Complete DNS configuration interface

  - DNS record display with copy-to-clipboard

  - DNS status indicators (Verified/Pending/Failed)

  - Manual DNS re-check capability

  - Raw DNS query results view

- **Sender Authentication** - SPF, DKIM, DMARC configuration

  - Automated SPF record generation with include mechanism

  - 2048-bit RSA DKIM key pair generation

  - DKIM rotation (manual and automated every 90 days)

  - DMARC policy configuration (none/quarantine/reject)

  - DMARC report analysis

- **Reputation Monitoring** - Domain health tracking

  - Domain reputation score (0-100) with trend tracking

  - Blacklist monitoring (URIBL, SURBL)

  - Google Postmaster Tools integration

  - Bounce rate and spam complaint tracking

  - Daily health checks via cron job

- **Domain Verification** - Automated domain ownership verification

  - DNS verification system with retry logic

  - Real-time DNS status checking

  - DNS propagation delay warnings (24-48 hours)

### �� Post-MVP Enhancements

See [Domain Management Roadmap](/docs/features/domains/roadmap) for detailed timeline and features:

- **Q1 2026:** Subdomain support with reputation isolation

- **Q2 2026:** Bulk domain onboarding for agencies, DNS instruction presets

- **Q3 2026:** Real-time DNS validation with misconfiguration alerts

- **Q4 2026:** Multi-region sending pools with geographic routing

- **Q1 2027:** Reputation prediction with proactive recommendations

## Related Documentation

### Planning & Strategy

- [Domain Management Roadmap](/docs/features/domains/roadmap) - Post-MVP feature timeline

- [Product Roadmap](/docs/operations/roadmap/product-roadmap/overview) - Overall product direction

### Implementation & Technical

- [Domain Routes](/docs/design/routes/workspace-domains) - UI specifications (7 routes)

- [Infrastructure Management](/docs/features/infrastructure/README) - VPS provisioning and mailbox creation

- [Email Operations](/docs/features/email-operations/README) - Email sending pipeline

- [Warmup Features](/docs/features/warmup/README) - Email warmup system

### Security & Compliance

- [Vault Integration](/docs/features/integrations/overview#vault-integration) - DKIM key storage in HashiCorp Vault

- [Vault SSH Management](/docs/features/infrastructure/vault-ssh-management) - SSH key storage for VPS access

- [Vault SMTP Credentials](/docs/features/infrastructure/vault-smtp/overview) - SMTP credentials storage

- [Compliance & Security](/docs/compliance-security/overview) - Security architecture

### APIs

- Domain Management API - `POST /api/v1/domains`, `GET /api/v1/domains/{id}/dns`

- See [Domain Management](/docs/features/domains/domain-management) for API examples

---

[← Back to All Features](https://github.com/penguinmails/penguinmails.github.io/blob/main)
