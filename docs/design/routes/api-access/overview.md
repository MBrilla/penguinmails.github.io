---
title: "API Access Routes Overview"
description: "Route specifications for API key management and developer settings"
last_modified_date: "2025-11-25"
level: "3"
persona: "Developers"
---

# API Access Routes

## Purpose and Context

API access routes enable developers to generate and manage API keys for programmatic access to PenguinMails. These routes are accessed occasionally during integration setup and key rotation.

**Feature References**:

- [API Access](/docs/features/integrations/api-access)
- [API Documentation](/docs/implementation-technical/api/README)

## UI Patterns and Components

**Core Components**:

- `APIKeyCard`: Display API key with masked value and copy button
- `KeyGenerationModal`: Create new API key with name and permissions
- `RateLimitIndicator`: Real-time rate limit usage display
- `CodeSnippet`: Syntax-highlighted code examples
- `PermissionMatrix`: Visual permission selector for API keys
- `UsageChart`: API usage analytics over time

**Analytics Patterns**: Request volume, endpoint usage, error rates.

**Layout**: Settings Layout with sidebar navigation.

## Route Specifications

| Route | Access | Purpose | State/Data Requirements |
|---|---|---|---|
| `/dashboard/settings/developers` | Admin, Developer | API Overview | List of API keys, rate limits, quick start guide |
| `/dashboard/settings/developers/keys` | Admin, Developer | Key Management | Create, view, revoke API keys |
| `/dashboard/settings/developers/docs` | Admin, Developer | API Docs | Interactive API documentation with examples |
| `/dashboard/settings/developers/usage` | Admin, Developer | Usage Analytics | API request metrics, endpoint usage, error logs |
| `/dashboard/settings/developers/webhooks` | Admin, Developer | Webhook Config | Webhook endpoints (links to ESP webhooks route) |

## Detailed Route Documentation

For detailed specifications of each route, see:

- [Developer Overview Route](/docs/design/routes/api-access/developer-overview-route) - Quick start and API keys summary
- [Key Management](/docs/design/routes/api-access/key-management) - Create, view, and revoke API keys
- [Docs and Usage](/docs/design/routes/api-access/docs-and-usage) - Interactive docs and usage analytics
- [Technical Reference](/docs/design/routes/api-access/technical-reference) - API endpoints and implementation details

---

**Last Updated:** November 25, 2025
**API Version:** v1
**SDKs:** Coming Q1 2026
