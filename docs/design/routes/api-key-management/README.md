---
title: "API Key Management Routes"
description: Frontend route specifications for tenant API key management
last_modified_date: 2025-01-15
level: 2
persona: ["frontend-developer"]
keywords: [routes, API-keys, Vault, authentication, UI]
---

# API Key Management Routes

Frontend routes for tenant API key management, enabling self-service creation, viewing, regeneration, and revocation of API keys for programmatic access to PenguinMails.

## Routes Summary

| Route | Purpose | Authentication | Permission |
|-------|---------|----------------|------------|
| `/dashboard/settings/developers/api-keys` | API key list and management | Required | Tenant user |
| `/dashboard/settings/developers/api-keys/create` | Create new API key (modal) | Required | Tenant user |
| `/dashboard/settings/developers/api-keys/{key_id}` | View API key details (modal) | Required | Tenant user |

## Topic Pages

- [Key List](./key-list) - API key list route, table structure, empty state
- [Create Modal](./create-modal) - Create API key modal and form fields
- [Success Modal](./success-modal) - API key created modal with code examples
- [Details Modal](./details-modal) - API key details with usage tabs
- [Confirmation Modals](./confirmation-modals) - Regenerate and revoke confirmations
- [State & Security](./state-security) - State management, errors, security, accessibility

## Status

- **Document Version**: 1.0
- **Status**: PLANNED
- **Priority**: P0 - Critical

## Related Documentation

- [Tenant API Key System](/docs/features/integrations/vault-api-keys/overview) - Feature documentation
- [Frontend Routing Map](/docs/design/frontend-routing-map) - Master route registry
- [Component Library](/docs/design/component-library) - Reusable UI components
- [Design System](/docs/design/design-system) - Design tokens and patterns
