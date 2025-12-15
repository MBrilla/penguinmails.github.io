---
title: "ESP Integration Technical Reference"
description: "API endpoints, data strategy, error handling, and component architecture for ESP integration routes"
last_modified_date: "2025-11-25"
level: "3"
persona: "Developers"
---

# ESP Integration Technical Reference

## Related API Endpoints

| Route | Related API | Description |
|---|---|---|
| `/settings/integrations/esp` | [ESP API](/docs/implementation-technical/api/tenant-api/esp-config) | `GET /api/v1/tenant/esp/providers` (List configured ESPs) |
| `/settings/integrations/esp/postmark` | [ESP API](/docs/implementation-technical/api/tenant-api/esp-config) | `POST /api/v1/tenant/esp/postmark` (Configure Postmark) |
| `/settings/integrations/esp/mailgun` | [ESP API](/docs/implementation-technical/api/tenant-api/esp-config) | `POST /api/v1/tenant/esp/mailgun` (Configure Mailgun) |
| `/settings/integrations/esp/routing` | [ESP API](/docs/implementation-technical/api/tenant-api/esp-config) | `PUT /api/v1/tenant/esp/routing` (Update routing rules) |
| `/settings/integrations/esp/webhooks` | [Webhook API](/docs/implementation-technical/api/platform-api/webhooks) | `POST /webhooks/postmark`, `POST /webhooks/mailgun` (Receive events) |
| `/settings/integrations/esp/analytics` | [Analytics API](/docs/implementation-technical/api/tenant-api/analytics) | `GET /api/v1/tenant/analytics/esp` (ESP performance metrics) |

## Data Strategy

### Fetching Methods

- **ESP List**: Server Component (cached for 5 minutes)
- **Configuration**: Server Component with real-time validation
- **Analytics**: Client Component with SWR for real-time updates

### Caching

- **Provider Status**: Cached for 5 minutes
- **Analytics**: Cached for 15 minutes
- **Webhook Logs**: No caching (real-time)

### Security

- **API Keys**: Encrypted at rest, never exposed to client
- **Webhook Signatures**: Verified on every request
- **Admin Only**: All ESP routes require admin role

## Edge Cases and Error Handling

| Error Condition | User Message |
|-----------------|--------------|
| Invalid API Key | "Invalid API key. Please check your credentials and try again." |
| Domain Not Verified | "Domain verification required. [View DNS records →]" (Block sending until verified) |
| ESP Service Down | "Unable to connect to [Provider]. Using fallback provider." |
| Webhook Signature Mismatch | Log security warning, reject webhook, alert admin |
| Rate Limit Exceeded | "ESP rate limit reached. Emails queued for retry." |
| Routing Conflict | "Rule priority determines which ESP is used." (Warn if multiple rules match) |

## Component Architecture

### Page Components

**`ESPOverview`** (Server)
- Features: Provider cards, quick stats, routing summary

**`PostmarkConfig`** (Client)
- Features: API token input, domain verification, test email

**`MailgunConfig`** (Client)
- Features: API key input, region selection, email validation

**`RoutingBuilder`** (Client)
- Features: Drag-and-drop rules, visual routing diagram

**`WebhookLogs`** (Client)
- Features: Real-time event stream, filtering, export

**`ESPAnalytics`** (Client)
- Features: Interactive charts, provider comparison, cost analysis

### Shared Components

- **`ProviderCard`**: Reused across ESP overview and analytics
- **`ConnectionTest`**: Reused in Postmark and Mailgun config
- **`DeliverabilityChart`**: Reused in analytics and campaign views

## Related Documentation

### Feature Documentation

- [ESP Integration](/docs/features/integrations/esp-integration) - Complete ESP integration guide
- [Webhook System](/docs/features/integrations/webhook-system) - Webhook configuration and events
- [Email Infrastructure](/docs/features/infrastructure/email-infrastructure-setup) - SMTP and infrastructure setup

### Technical Documentation

- [ESP API Reference](/docs/implementation-technical/api/tenant-api/esp-config) - API endpoints for ESP configuration
- [Webhook API Reference](/docs/implementation-technical/api/platform-api/webhooks) - Webhook endpoint specifications
- [Analytics API Reference](/docs/implementation-technical/api/tenant-api/analytics) - ESP analytics endpoints

### User Journeys

- Operations Team Journey - ESP setup and monitoring workflows
- Developer Journey - API integration and webhook setup

---

**Last Updated:** November 25, 2025
**Supported ESPs:** Postmark, Mailgun
**Coming Soon:** SendGrid, Amazon SES

ESP integration routes provide comprehensive configuration and monitoring for external email service providers, enabling optimized deliverability and cost management.
