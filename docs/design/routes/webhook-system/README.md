---
title: "Webhook System Routes"
description: "Route specifications for custom webhook configuration and event management"
last_modified_date: "2025-01-20"
level: 3
persona: Developers
keywords: webhooks, routes, event notifications, API integration
---

# Webhook System Routes

This section provides route specifications for the webhook management interface, enabling developers to configure custom webhooks for real-time event notifications from PenguinMails.

## Routes Overview

| Route | Access | Purpose |
|-------|--------|---------|
| `/dashboard/settings/webhooks` | Admin, Developer | Webhook Overview |
| `/dashboard/settings/webhooks/create` | Admin, Developer | Create Webhook |
| `/dashboard/settings/webhooks/:id` | Admin, Developer | Webhook Details |
| `/dashboard/settings/webhooks/:id/deliveries` | Admin, Developer | Delivery Log |
| `/dashboard/settings/webhooks/:id/test` | Admin, Developer | Test Webhook |
| `/dashboard/settings/webhooks/events` | Admin, Developer | Event Reference |

## Topic Pages

**[Overview Route](./overview.md)** covers the main webhook list page with status cards, health metrics, and quick filters. Users monitor all configured webhooks and their delivery status from this central dashboard.

**[Create Webhook Route](./create-webhook.md)** details the multi-step form for webhook creation including endpoint configuration, event selection, filtering options, and security settings.

**[Webhook Details Route](./webhook-details.md)** describes the configuration summary page with health statistics, recent deliveries, and management actions like pause, resume, and secret rotation.

**[Delivery Log Route](./delivery-log.md)** documents the detailed delivery history interface with filtering, request/response inspection, and retry capabilities.

**[Test Webhook Route](./test-webhook.md)** covers the testing interface with payload preview, code examples for signature verification, and result display.

**[Event Reference Route](./event-reference.md)** presents the documentation page for all available webhook events, payload schemas, and usage patterns.

**[Technical Details](./technical-details.md)** contains API endpoints, data strategy, edge cases, error handling, and component architecture.

## Related Documentation

- [Webhook System](/docs/features/integrations/webhook-system) - Feature documentation
- [API Access](/docs/features/integrations/api-access) - API authentication
- [Webhooks API Reference](/docs/implementation-technical/api/platform-api/webhooks) - API endpoints
