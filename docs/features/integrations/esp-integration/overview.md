---
title: "ESP Integration Overview"
description: "External Email Service Provider integration overview for enhanced deliverability"
last_modified_date: "2025-01-15"
level: "2"
persona: "Operations Teams, Developers"
---

# ESP Integration Overview

Connect external Email Service Providers (ESPs) for enhanced deliverability and specialized sending capabilities.

## Overview

While PenguinMails has built-in email sending capabilities via your own SMTP infrastructure, you can optionally integrate with premium ESPs like Postmark and Mailgun for higher deliverability rates, dedicated IP addresses, advanced sending infrastructure, and specialized transactional or bulk email handling.

### Supported ESPs

| ESP | Best For | Pricing | Setup Time |
|-----|----------|---------|------------|
| [Postmark](https://postmarkapp.com) | Transactional emails | Pay-as-you-go | 5 min |
| [Mailgun](https://www.mailgun.com) | Bulk marketing emails | Monthly plans | 10 min |

## When to Use an ESP

### Built-in SMTP (via Hostwind VPS)

Advantages:
- Full control over infrastructure
- No per-email costs
- Custom configuration

Considerations:
- Requires warmup and IP reputation management
- Shared IP reputation risk

### External ESP (Postmark/Mailgun)

Advantages:
- Pre-warmed infrastructure
- Dedicated deliverability team
- Advanced analytics and tracking
- Better inbox placement rates

Considerations:
- Per-email costs
- Less direct control

### Recommended Approach

Use multiple providers for different purposes:

- **Postmark** → Transactional emails (password resets, receipts, alerts)
- **Mailgun** → Marketing campaigns (newsletters, promotions)
- **Built-in SMTP** → High-volume cold outreach, custom scenarios

## Document Structure

This ESP integration framework is organized into focused implementation guides:

| Document | Focus Area |
|----------|------------|
| [Postmark Integration](postmark.md) | Premium transactional email delivery setup |
| [Mailgun Integration](mailgun.md) | Bulk marketing email delivery configuration |
| [Advanced Configuration](advanced-config.md) | Routing rules, webhooks, monitoring |
| [Technical Implementation](technical-implementation.md) | API integration, failover, cost optimization |

## Coming Soon

Additional ESP integrations planned for future releases:

- **SendGrid** - Enterprise-grade email delivery
- **Amazon SES** - AWS-native email service

---

**Related Documents:**
- [API Access](/docs/features/integrations/api-access)
- [Webhook System](/docs/features/integrations/webhook-system)
- [Email Infrastructure Setup](/docs/features/infrastructure/email-infrastructure-setup)
