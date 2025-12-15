---
title: "Integrations"
description: "Third-party system connections including CRM, webhooks, and external platforms"
last_modified_date: "2025-12-16"
level: "1"
persona: "All Users"
---

# Integrations

Third-party system connections including CRM, webhooks, and external platforms.

**[📖 Read Full Integrations Overview](/docs/features/integrations/overview)** - Comprehensive guide to PenguinMails integration capabilities including API-first strategy, Vault secrets management, and implementation roadmap.

## MVP Status: API-First Approach

**Strategy**: Build general-purpose REST API + webhooks first (MVP), add vendor-specific integrations later (Post-MVP)

**Current State**: Foundation features documented, implementation in progress

**MVP Focus (Q1 2026)**:

- ✅ API Key Management System with Vault storage

- ✅ Core REST API endpoints (contacts, campaigns, emails, analytics)

- ✅ Webhook system for real-time event notifications

- ✅ API documentation (OpenAPI/Swagger)

**Post-MVP (Q1-Q2 2026)**:

- Salesforce/HubSpot CRM integrations (Q1 2026)

- Zapier integration for 5,000+ apps (Q2 2026)

- ESP integration UI (Q3 2026)

**Rationale**: API-first approach enables customers to build custom integrations immediately while we develop pre-built vendor integrations on top of stable API foundation.

## Features in This Category

### In Development (MVP - Q1 2026)

- **[API Key Management](/docs/features/integrations/vault-api-keys/overview)** - Secure API key generation, Vault storage, permission scopes, rate limiting (P0 - Critical)

- **[REST API Endpoints](/docs/features/integrations/api-access)** - Core API for contacts, campaigns, emails, analytics with OpenAPI documentation (P0 - Critical)

- **[Webhook System](/docs/features/integrations/webhook-system)** - Real-time event notifications with retry logic and signature verification (P1 - High)

### Available Now

- **[API Access Documentation](/docs/features/integrations/api-access)** - RESTful API specification for custom integrations

- **[Vault API Keys Documentation](/docs/features/integrations/vault-api-keys/overview)** - Secure API key management architecture with Vault storage and bcrypt hashing

- **[ESP Integration](/docs/features/integrations/esp-integration)** - External email service providers (Postmark, Mailgun) - Active integrations

- **[Webhook System Documentation](/docs/features/integrations/webhook-system)** - Real-time event notification architecture

### �� Coming Soon (Post-MVP)

- **[CRM Integration](/docs/features/integrations/crm-integration/overview)** - Salesforce and HubSpot bi-directional sync (Q1 2026)

- **Zapier Integration** - Connect with 5,000+ apps via no-code automation (Q2 2026)

- **ESP Integration UI** - User interface for Postmark/Mailgun configuration (Q3 2026)

- **Custom Webhook Builder** - Advanced filtering and conditional logic (Q3 2026)

- **Additional CRM Integrations** - Pipedrive, Zoho, Close.io (Q2-Q3 2026)

## Missing Features & Roadmap

### MVP Gaps (Q1 2026)

## Critical (P0) - Blocks programmatic integrations

1. **API Key Management UI** (5-7 days) - Self-service API key creation, viewing, regeneration, revocation

2. **Core REST API Implementation** (2-3 weeks) - Contacts, campaigns, emails, analytics endpoints

3. **API Documentation** (1 week) - OpenAPI/Swagger with interactive explorer and code examples

## High Priority (P1) - Enables real-time integrations

1. **Webhook System Backend** (1-2 weeks) - Event delivery engine with retry logic and signature verification

2. **Webhook Configuration UI** (1-2 weeks) - Self-service webhook setup with event selection and testing

**Total MVP Effort**: 6-9 weeks

### Post-MVP Roadmap

### Q1 2026: Advanced CRM Integrat

- Salesforce integration with OAuth and bi-directional sync (20-30 days)

- HubSpot integration with field mapping and workflow triggers (20-30 days)

### Q2 2026: No-Code Integrations

- Zapier integration for 5,000+ apps (2-3 weeks)

- Pre-built Zaps for common workflows

### Q3 2026: Enhanced Features

- ESP Integration UI for Postmark/Mailgun configuration (10-14 days)

- Custom Webhook Builder with advanced filtering (3-4 weeks)

- Additional CRM integrations (Pipedrive, Zoho, Close.io)

## Q4 2026+: Ecosystem Expansion

- Integration Marketplace for third-party developers (8-12 weeks)

- Real-Time Sync with sub-1-minute latency (6-8 weeks)

**Detailed Roadmap**: [Integrations Roadmap](/docs/features/integrations/roadmap)

## Vault Integration & Secrets Management

**Status**: ✅ Fully Documented - Architecture and implementation workflows defined

PenguinMails uses HashiCorp Vault for centralized secrets management, providing enhanced security, automated rotation, and disaster recovery capabilities.

**Key Features**:

- **VPS SSH Key Management**: Admin and tenant SSH keys stored in Vault with 90-day rotation

- **SMTP Credentials Storage**: MailU admin credentials encrypted and stored in Vault with 180-day rotation

- **Tenant API Key System**: API keys hashed with bcrypt and stored in Vault with on-demand rotation

- **Disaster Recovery**: Automated daily backups enable rapid recovery from VPS compromise or Vault failure

**Documentation**:

- [Vault Integration Architecture](/docs/.kiro/specs/feature-completeness-review/findings/vault-integration-architecture)

- [VPS SSH Key Management](/docs/features/infrastructure/vault-ssh-management)

- [SMTP Credentials Storage](/docs/features/infrastructure/vault-smtp/overview)

- [Tenant API Keys](/docs/features/integrations/vault-api-keys/overview)

- [Disaster Recovery Procedures](/docs/features/infrastructure/vault-disaster-recovery/)

## Third-Party Dependencies

**Critical Services (P0)**:

- **HashiCorp Vault** - Secrets management (HCP Vault: $22/month MVP, self-hosted Post-MVP)

- **Stripe** - Payment processing (2.9% + $0.30 per transaction, no migration planned)

- **Hostwind API** - VPS provisioning ($5-50/month per VPS, add alternatives Q2 2026)

**Active Integrations**:

- **Postmark** - Transactional email delivery ($1.25 per 1,000 emails)

- **Mailgun** - Bulk marketing email delivery ($35/month + $1/1,000 emails)

- **Loop.so** - Transactional email service ($29/month, migrate to MailU Q3 2026)

**Planned Integrations (Post-MVP)**:

- **Salesforce** - CRM integration (Q1 2026)

- **HubSpot** - CRM integration (Q1 2026)

- **Zapier** - 5,000+ app connections (Q2 2026)

**Detailed Analysis**: [Third-Party Dependencies Report](/docs/.kiro/specs/feature-completeness-review/findings/integrations-third-party-dependencies)

## Cross-References

**Related Features**:

- [Authentication & Security](/docs/features/authentication/user-management) - API key authentication, OAuth flows

- [Infrastructure Management](/docs/features/infrastructure/README) - VPS provisioning, SSH key management, SMTP setup

- [Domain Management](/docs/features/domains/README) - DKIM key storage in Vault

- [Email Operations](/docs/features/campaigns/README) - Campaign API endpoints, webhook events

**Implementation**:

- [API Documentation](/docs/implementation-technical/api/README) - Technical API specifications

- [Vault Architecture](/docs/.kiro/specs/feature-completeness-review/findings/vault-integration-architecture) - Secrets management architecture

**Business Strategy**:

- [Integration Strategy](/docs/.kiro/specs/feature-completeness-review/findings/integration-strategy) - API-first approach rationale

- [Product Roadmap](/docs/operations/roadmap/product-roadmap/overview) - Integration timeline and priorities

- [Executive Roadmap](/docs/business/roadmap/executive-roadmap) - Strategic integration investments

---

[← Back to All Features](https://github.com/penguinmails/penguinmails.github.io/blob/main)
