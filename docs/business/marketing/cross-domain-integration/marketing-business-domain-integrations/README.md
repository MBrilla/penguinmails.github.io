---
title: Marketing-Business Domain Integration
description: Integration specifications between marketing systems and business domains
last_modified_date: 2025-01-15
level: 2
persona: ["developer", "analyst"]
keywords: [integration, cross-domain, marketing, CRM, analytics]
---

# Marketing-Business Domain Integration

Detailed specifications for integrating marketing systems with other business domains including Sales, Product, Customer Success, and Finance.

## Integration Architecture

Marketing systems connect with business domains through real-time event streaming and scheduled batch synchronization. All integrations follow consistent patterns for data validation, error handling, and performance monitoring.

## Domain Integration Topics

- [Sales Domain](./sales-domain) - CRM synchronization and campaign attribution
- [Product Domain](./product-domain) - Feature adoption and feedback integration
- [Customer Success](./customer-success) - Health monitoring and retention campaigns
- [Finance Domain](./finance-domain) - ROI tracking and budget optimization
- [Data Synchronization](./data-synchronization) - Event streaming, batch sync, and error handling

## Integration Patterns Overview

| Domain | Primary Pattern | Data Flow |
|--------|----------------|-----------|
| Sales | Real-time bidirectional | Marketing ↔ CRM |
| Product | Analytics-driven | Product → Marketing |
| Customer Success | Health score-driven | All domains → CS |
| Finance | Attribution-based | Marketing → Finance |

## Technical Requirements

All integrations follow these standards:

- Event-driven architecture using PostgreSQL + Redis
- Multi-touch attribution modeling
- Real-time SLA monitoring
- Automated error recovery with exponential backoff

## Related Documentation

- [Sales Domain Integration](./sales-domain)
- [Finance Domain Integration](./finance-domain)
- [Data Synchronization Patterns](./data-synchronization)
