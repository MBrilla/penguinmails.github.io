---
title: "ESP Integration Routes Overview"
description: "Route specifications for External Email Service Provider (ESP) configuration and management"
last_modified_date: "2025-11-25"
level: "3"
persona: "Developers, Operations Teams"
---

# ESP Integration Routes

## Purpose and Context

ESP integration routes enable configuration and management of external Email Service Provider integrations for enhanced deliverability. These routes are accessed occasionally during initial setup and when adding new ESPs.

**Feature References**:

- [ESP Integration](/docs/features/integrations/esp-integration)
- [Email Infrastructure](/docs/features/infrastructure/email-infrastructure-setup)

## UI Patterns and Components

**Core Components**:

- `Form`: Configuration forms with validation
- `ConnectionTest`: Test ESP connection with real-time feedback
- `ProviderCard`: Visual card for each ESP with status indicators
- `RoutingRulesBuilder`: Drag-and-drop interface for email routing logic
- `WebhookEndpoint`: Display webhook URLs with copy button
- `DeliverabilityChart`: Comparative analytics across providers

**Analytics Patterns**: Provider performance comparison, delivery metrics.

**Layout**: Settings Layout with sidebar navigation.

## Route Specifications

| Route | Access | Purpose | State/Data Requirements |
|---|---|---|---|
| `/dashboard/settings/integrations/esp` | Admin | ESP Overview | List of configured ESPs with status, quick actions |
| `/dashboard/settings/integrations/esp/postmark` | Admin | Postmark Config | Form: API Token, Default Sender, Tracking Options |
| `/dashboard/settings/integrations/esp/mailgun` | Admin | Mailgun Config | Form: API Key, Domain, Region, Tracking Options |
| `/dashboard/settings/integrations/esp/routing` | Admin | Routing Rules | Visual builder for email type → ESP mapping |
| `/dashboard/settings/integrations/esp/webhooks` | Admin | Webhook Config | Webhook URLs, event subscriptions, delivery logs |
| `/dashboard/settings/integrations/esp/analytics` | Admin | ESP Analytics | Comparative deliverability metrics across providers |

## Detailed Route Documentation

For detailed specifications of each route, see:

- [ESP Overview Route](/docs/design/routes/esp-integration/esp-overview-route) - Provider cards and quick actions
- [Provider Configuration](/docs/design/routes/esp-integration/postmark-mailgun) - Postmark and Mailgun setup
- [Routing and Webhooks](/docs/design/routes/esp-integration/routing-webhooks) - Email routing rules and webhook configuration
- [ESP Analytics](/docs/design/routes/esp-integration/analytics) - Performance comparison dashboard
- [Technical Reference](/docs/design/routes/esp-integration/technical-reference) - API endpoints and implementation details

---

**Last Updated:** November 25, 2025
**Supported ESPs:** Postmark, Mailgun
**Coming Soon:** SendGrid, Amazon SES
