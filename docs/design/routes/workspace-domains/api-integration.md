---
title: "API Integration & Data Strategy"
description: "Related API endpoints, data strategy, and component architecture"
last_modified_date: "2025-01-09"
level: "2"
persona: "Technical Teams"
---

# API Integration & Data Strategy

## Related API Endpoints

| Route | Related API | Description |
|-------|-------------|-------------|
| `/domains` | [Domains API](/docs/implementation-technical/api/tenant-smtp/domains) | `GET /api/v1/tenant/domains` (List), `POST` (Add) |
| `/domains/[id]` | [Domains API](/docs/implementation-technical/api/tenant-smtp/domains) | `GET /api/v1/tenant/domains/{id}` (Details & DNS Status) |
| `/domains/[id]/emails` | [Domains API](/docs/implementation-technical/api/tenant-smtp/domains) | `POST /api/v1/tenant/domains/{id}/accounts` (Add Email) |
| `/domains` (Health) | [Reputation API](/docs/implementation-technical/api/central-smtp/reputation) | `GET /api/v1/reputation/domains/{domain}` (Health Score) |

---

## Data Strategy

### Fetching Method

- **Domain List**: Server Component
- **DNS Status**: Hybrid - Initial load via Server Component, "Re-check" button triggers Client-side fetch

### Caching

- **Domain List**: Cached for 1 minute
- **Reputation Scores**: Cached for 24 hours (updated via nightly job)

### Optimistic Updates

- "Delete Domain" removes row immediately

### Streaming

- **DNS Verification**: When adding a domain, the verification step uses a streaming response or polling to show progress of record checks

---

## Edge Cases & Error Handling

### DNS Propagation Delay

Explicitly warn users that verification can take 24-48h.

### Domain Blacklisted

Show critical alert with specific blacklist provider and delisting link.

### SMTP Connection Fail

During email account setup, if credentials fail, show specific error (e.g., "Authentication Failed" vs "Timeout").

### Rate Limited

If sending limit reached, account status changes to "Rate Limited" until next window.

---

## Component Architecture

### Page Components

**DomainList** (Server):
- Props: `domains: Domain[]`
- Dependencies: `HealthScoreRing`

**DNSStatusTable** (Client):
- Props: `records: DNSRecord[]`
- Features: Polling for verification status, copy-to-clipboard buttons

**InstructionModal** (Client):
- Props: `provider: 'GoDaddy' | 'Namecheap' | 'Cloudflare'`
- Content: Provider-specific step-by-step guides

### Shared Components

**HealthScoreRing**: SVG circular progress with color coding (Green/Yellow/Red)

**CopyButton**: Reused globally for API keys and DNS values

---

## Technical Integration Notes

### Domain List View

- **Real-time DNS Checks**: Button triggers backend DNS lookup (via DNS resolver API)
- **Blacklist Monitoring**: Daily cron job checks against known blacklists
- **Reputation Calculation**: Aggregated from bounce rates, spam complaints, and blacklist status

### Add Domain Wizard

- **DNS Validation**: Backend queries DNS records using DNS resolver
- **Async Verification**: Polling mechanism checks DNS propagation status

### Email Account Setup

- **Authentication**: SMTP credentials encrypted at rest using AES-256
- **Connection Testing**: Validates SMTP/IMAP credentials before saving (sends test email)
- **Auto-warmup**: Newly added account automatically enters warmup sequence

### Warmup Management

- **Automated Progression**: Background job checks daily and adjusts limits
- **Health-based Pausing**: If health score drops, warmup automatically pauses
- **Email Distribution**: Warmup emails distributed across time windows for natural patterns

---

## Related Documentation

- [Domain Setup Guide](/docs/features/warmup/email-warmups/overview)
- [DNS Configuration](/docs/technical/infrastructure/email-deliverability)
- [Domain Setup Tutorial](/docs/operations/freelancer-management/support/tutorials/domain-setup)
- [DNS Provider Guides](/docs/technical/infrastructure/dns-providers)
- [Email Account Health](/docs/features/warmup/email-warmups/overview)
- [Free Mailbox Creation](/docs/features/infrastructure/free-mailbox-creation/overview)
- [Security Best Practices](/docs/compliance-security/enterprise/overview)
- [Warmup Strategies](/docs/features/warmup/email-warmups/overview)

---

## Related Sections

- [Overview](/docs/design/routes/workspace-domains/overview) - Workspace domains overview
- [Domain Setup](/docs/design/routes/workspace-domains/domain-setup) - Domain wizard and configuration
- [Email Accounts](/docs/design/routes/workspace-domains/email-accounts) - Email account management and warmup
