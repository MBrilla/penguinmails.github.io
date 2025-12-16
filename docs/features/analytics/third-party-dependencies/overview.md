---
title: "Analytics Third-Party Dependencies Overview"
description: "Overview of third-party services used by the Analytics & Reporting system"
last_modified_date: "2025-01-15"
level: "3"
persona: "Technical Teams, Developers"
---

# Analytics Third-Party Dependencies Overview

This document outlines all third-party services and integrations used by the Analytics & Reporting system, including current MVP dependencies, planned enhancements, and future considerations.

## Architecture Philosophy

We use a consolidated, cost-effective approach with PostHog as our primary analytics platform, supplemented by targeted services for specific needs.

## Current State Summary

| Service | Purpose | Monthly Cost | Status |
|---------|---------|--------------|--------|
| PostHog | Product analytics, logging, monitoring | $0 (free tier) | Active |
| Stripe | Payment processing, financial analytics | Transaction fees | Active |
| Loop.so | Transactional email for reports | $29 | Active |

**Total Monthly Cost:** ~$29/month (plus Stripe transaction fees)

## Future Roadmap

| Service | Purpose | Target Timeline |
|---------|---------|-----------------|
| Gemini AI | Predictive analytics, ML insights | Q1 2026 |
| In-house SMTP | Replace Loop.so | Q3 2026 |
| Data Warehouse | Enterprise integration | Q3 2026 (customer-driven) |

## Document Structure

This dependencies framework is organized into focused sections:

| Document | Focus Area |
|----------|------------|
| [Current Dependencies](current-dependencies.md) | PostHog, Stripe, Loop.so details |
| [Future Dependencies](future-dependencies.md) | Planned services and investigations |
| [Integration Architecture](integration-architecture.md) | Technical implementation and security |

## Strategic Benefits

**Cost Optimization:** Minimal monthly costs with PostHog free tier, consolidating analytics, logging, and error tracking.

**Simplified Architecture:** Single platform (PostHog) reduces complexity compared to multiple specialized tools.

**Demand-Driven Approach:** Future integrations validated by customer demand before implementation.

**Flexibility:** Clear migration paths and minimal vendor lock-in risk across all dependencies.

---

**See Also:**
- [Platform-Wide Third-Party Services](/docs/technical/architecture/third-party-services) - Comprehensive analysis of all platform dependencies
- [Core Analytics Overview](/docs/features/analytics/core-analytics/overview) - Analytics system overview
