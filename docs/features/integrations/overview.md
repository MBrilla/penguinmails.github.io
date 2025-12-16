---
title: "Integrations Overview"
description: "API-first integration platform connecting email infrastructure, CRMs, and external services"
level: "2"
status: "PLANNED"
roadmap_timeline: "MVP 2025"
priority: "High"
keywords: [integrations, api, webhooks, crm, esp, vault]
---

# Integrations Overview

**Quick Access**: Connect PenguinMails with external services through our API-first architecture, webhooks, and native integrations.

## API-First Philosophy

Every feature in PenguinMails is built API-first, meaning all functionality is accessible programmatically before any UI is created. This ensures third-party integrations have the same capabilities as our native interface.

### Design Principles

| Principle | Implementation |
|-----------|----------------|
| API-First | All features via REST/GraphQL before UI |
| Webhook-Driven | Real-time events for all state changes |
| OAuth 2.0 | Secure third-party authentication |
| Rate Limiting | Tiered limits per plan |
| Versioning | API versions with deprecation policy |

## Core Integration Features

### API Access

Full programmatic access to all PenguinMails features:

- **REST API**: Standard HTTP/JSON endpoints
- **GraphQL**: Flexible queries for complex data needs
- **API Keys**: Scoped keys with granular permissions
- **SDKs**: Official libraries for Python, Node.js, Ruby

### Webhooks

Real-time event notifications:

- Email sent, opened, clicked, bounced
- Campaign status changes
- Contact subscription changes
- Infrastructure health alerts

### CRM Integrations

Native connectors for popular CRMs:

- **Salesforce**: Bi-directional sync
- **HubSpot**: Contact and activity sync
- **Pipedrive**: Deal-based email triggers
- **Custom**: Webhook-based integration

### ESP Integrations

Email service provider connections:

- **Postmark**: Transactional email
- **Mailgun**: Bulk sending
- **SendGrid**: Marketing automation
- **Amazon SES**: High-volume sending

## HashiCorp Vault Integration

Secure secrets management for email infrastructure credentials. See [Vault Secrets Management](/docs/features/integrations/vault-secrets-management) for complete details.

### Supported Secret Types

| Secret Type | Description |
|-------------|-------------|
| SMTP Credentials | Mail server authentication |
| API Keys | ESP and service API keys |
| DKIM Keys | Domain authentication |
| OAuth Tokens | Third-party app access |
| SSH Keys | Server access credentials |

## Planned Integrations

| Integration | Timeline | Status |
|-------------|----------|--------|
| Stripe | MVP 2025 | Development |
| Slack | Q1 2026 | Planned |
| Zapier | Q2 2026 | Planned |
| Make (Integromat) | Q2 2026 | Planned |

## Related Documentation

- [Vault Secrets Management](/docs/features/integrations/vault-secrets-management) - Secure credential storage
- [Third-Party Roadmap](/docs/features/integrations/third-party-roadmap) - Integration timeline
- [API Documentation](/docs/developers/api-reference) - Full API reference
- [Webhooks Guide](/docs/developers/webhooks) - Event notifications

---

**Last Updated:** November 25, 2025
**Status:** Planned - MVP Feature (Level 2)
**Target Release:** MVP 2025 (Core), Q2 2026 (Extended)
**Owner:** Platform Team
