---
title: Template Variables
description: Dynamic content insertion using merge tags and personalization variables
last_modified_date: 2025-01-15
level: 1
persona: ["email-marketer", "developer"]
keywords: [merge-tags, personalization, dynamic-content, template-variables]
---

# Template Variables

Template variables enable dynamic content insertion in email templates through merge tags, conditional logic, and data-driven personalization. This system transforms static templates into personalized communications that adapt to each recipient's context.

## Quick Navigation

Start with basic merge tags then progress to advanced features as your needs grow.

| Guide | Focus | Audience |
|-------|-------|----------|
| [Quick Start](./quick-start) | Basic merge tags, defaults, simple conditionals | Email marketers |
| [Advanced Usage](./advanced-usage) | Custom fields, loops, formatting, dynamic content | Power users |

### Technical Documentation

| Guide | Focus |
|-------|-------|
| [Database Schema](./schema) | PostgreSQL tables, indexes, Drizzle ORM definitions |
| [VariableParser Service](./parser) | TypeScript parser implementation, filters, conditionals |
| [API Endpoints](./api) | REST API for parsing, validation, variable management |

## Core Capabilities

Template variables support three categories of dynamic content:

**Merge Tags** insert recipient-specific data like names, company details, and custom fields directly into email content using `{{variable}}` syntax.

**Conditional Logic** shows or hides content blocks based on recipient data, enabling targeted messaging without maintaining separate templates.

**Data Loops** iterate over collections like product lists or recent activities, generating repeated content sections dynamically.

## Implementation Status

Template variables are planned for Q2 2026 as part of the personalization engine rollout. The feature depends on the core template system and recipient data infrastructure.

## Related Documentation

- [Template System Overview](/docs/features/templates/)
- [Personalization Engine](/docs/implementation-technical/marketing/personalization/engine/)
- [Email Campaigns](/docs/features/campaigns/)
