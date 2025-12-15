---
title: "Architecture Overview"
description: "Comprehensive technical architecture for PenguinMails multi-tenant platform"
last_modified_date: "2025-12-15"
level: "2"
persona: "Technical Leads, Architects"
---

# Architecture Overview

Multi-tenant, microservices architecture designed for cold email infrastructure management.

## Executive Summary

PenguinMails is built on a multi-tenant, microservices architecture designed specifically for cold email infrastructure management. The system combines automated infrastructure provisioning, real-time deliverability monitoring, and intelligent campaign management.

### Key Architectural Decisions

| Decision | Rationale |
|----------|-----------|
| Multi-tenant by design | Complete data isolation with efficient infrastructure sharing |
| Email infrastructure specialization | Built for cold email deliverability, not general marketing |
| Automation-first approach | Minimize manual operations through intelligent automation |
| Compliance built-in | GDPR, CAN-SPAM, international compliance as core features |
| Real-time monitoring | Continuous monitoring of deliverability and system health |

## Topic Guides

| Guide | Description |
|-------|-------------|
| [System Components](./system-components) | User interface, API gateway, core services |
| [Infrastructure Layer](./infrastructure-layer) | VPS, SMTP, DNS, database architecture |
| [External Integrations](./external-integrations) | Hostwinds, MailU, Stripe, NileDB |
| [Data Flow](./data-flow) | Request routing, email processing, analytics |

## High-Level Architecture

```text
┌─────────────────────────────────────────────────────────────┐
│                    PENGUINMAILS PLATFORM                    │
└─────────────────────────────────────────────────────────────┘
                             │
         ┌───────────────────┼───────────────────┐
         │                   │                   │
┌────────▼────────┐ ┌────────▼────────┐ ┌───────▼────────┐
│  USER INTERFACE │ │   API GATEWAY   │ │   EXTERNAL     │
│                 │ │                 │ │   SERVICES     │
│ • Dashboard     │ │ • Auth          │ │ • Hostwind VPS │
│ • Admin Panel   │ │ • Rate Limiting │ │ • MailU SMTP   │
│ • Knowledge Base│ │ • Load Balance  │ │ • Stripe       │
└────────┬────────┘ └────────┬────────┘ └───────┬────────┘
         │                   │                   │
         └───────────────────┼───────────────────┘
                             │
              ┌──────────────▼──────────────┐
              │        CORE SERVICES        │
              │ • User Management           │
              │ • Tenant Management         │
              │ • Infrastructure Management │
              │ • Campaign Engine           │
              │ • Email Processor           │
              │ • Analytics                 │
              └──────────────┬──────────────┘
                             │
              ┌──────────────▼──────────────┐
              │    INFRASTRUCTURE LAYER     │
              │ • PostgreSQL + Redis        │
              │ • VPS Management            │
              │ • SMTP Servers              │
              │ • Monitoring                │
              └─────────────────────────────┘
```

---

## Related Documentation

- [Database Migration Guide](/docs/implementation-technical/database-infrastructure/operations/database-migration-guide)
- [Backup & Recovery](/docs/implementation-technical/database-infrastructure/operations/backup-recovery-procedures)
- [Development Guidelines](/docs/implementation-technical/development-guidelines/README)
