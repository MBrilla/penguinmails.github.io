---
title: "Current Analytics Dependencies"
description: "PostHog, Stripe, and Loop.so integration details for MVP analytics"
last_modified_date: "2025-01-15"
level: "3"
persona: "Technical Teams, Developers"
---

# Current Analytics Dependencies

Detailed information on third-party services currently active in the Analytics & Reporting system.

## PostHog - Product Analytics & Event Tracking

**Purpose:** Comprehensive analytics, logging, and monitoring platform

### Usage

- User behavior analytics and funnel analysis
- Event tracking across all platform features
- Error tracking and performance monitoring
- System logs and operational analytics
- Feature flag management
- Session replay for debugging

**Cost:** Free tier (1M events/month), then usage-based pricing

**Status:** Active (MVP)

### Why PostHog

Consolidates multiple tools (analytics, logging, error tracking) into a single platform, reducing complexity and cost. Provides comprehensive analytics capabilities without requiring separate logging and error tracking services.

**Self-Hosting Option:** PostHog can be self-hosted if needed for data residency or cost optimization at scale.

## Stripe - Payment Processing

**Purpose:** Payment processing and financial analytics

### Usage

- Subscription management and billing
- Webhook integration for subscription events
- MRR (Monthly Recurring Revenue) calculation
- Revenue tracking and financial analytics
- Payment method management

**Cost:** Transaction fees (2.9% + $0.30 per transaction)

**Status:** Active

**Integration:** Webhook-based real-time event processing for subscription lifecycle events.

### Events Tracked

- `customer.subscription.created`
- `customer.subscription.updated`
- `customer.subscription.deleted`
- `invoice.payment_succeeded`
- `invoice.payment_failed`

### Data Synchronization

Real-time webhook processing triggers automatic MRR recalculation and financial analytics updates.

## Loop.so - Transactional Email Service

**Purpose:** Scheduled report delivery via email

### Usage

- Send weekly/monthly analytics reports to users
- Automated report distribution
- Email template management for reports

**Cost:** $29/month (up to 50K emails)

**Status:** Active

### Report Types

- Weekly performance summaries
- Monthly executive reports
- Custom scheduled reports
- On-demand exports

### Replacement Plan (Q3 2026)

Migrate to in-house SMTP server for cost savings ($29/month → $0).

**Migration Complexity:** High (2-3 weeks)

Migration steps:
1. Build central SMTP server
2. Implement template management
3. Add delivery tracking
4. Migrate report delivery workflows
5. Deprecate Loop.so integration

## Cost Analysis

### Current Monthly Costs

| Service | Cost | Notes |
|---------|------|-------|
| PostHog | $0 | Free tier (1M events/month) |
| Stripe | Variable | Transaction fees (2.9% + $0.30) |
| Loop.so | $29 | Up to 50K emails/month |
| **Total** | **~$29/month** | Plus Stripe transaction fees |

### Cost Efficiency

PostHog's free tier consolidates analytics, logging, and error tracking into a single platform, significantly reducing infrastructure costs while maintaining comprehensive monitoring capabilities.

---

**Related Documents:**
- [Overview](overview.md)
- [Future Dependencies](future-dependencies.md)
- [Integration Architecture](integration-architecture.md)
