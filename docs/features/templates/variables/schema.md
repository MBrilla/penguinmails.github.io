---
title: Template Variables Database Schema
description: PostgreSQL schema design for template variable definitions, usage tracking, and recipient values
last_modified_date: 2025-01-15
level: 3
persona: ["developer"]
keywords: [database-schema, drizzle-orm, postgresql, multi-tenant]
---

# Template Variables Database Schema

Template variable definitions and usage tracking are stored in PostgreSQL using Drizzle ORM with multi-tenant isolation.

## template_variables Table

Stores variable definitions available for use in templates:

```typescript
import { pgTable, text, timestamp, uuid, jsonb, boolean } from 'drizzle-orm/pg-core';

export const templateVariables = pgTable('template_variables', {
  id: uuid('id').defaultRandom().primaryKey(),
  tenantId: uuid('tenant_id').notNull().references(() => tenants.id),
  
  // Variable identification
  name: text('name').notNull(),
  displayName: text('display_name').notNull(),
  category: text('category').notNull(), // 'standard', 'custom', 'system'
  
  // Type and validation
  dataType: text('data_type').notNull(), // 'string', 'number', 'boolean', 'date', 'array', 'object'
  defaultValue: jsonb('default_value'),
  validation: jsonb('validation'), // regex, min, max, enum values
  
  // Metadata
  description: text('description'),
  isRequired: boolean('is_required').default(false),
  isActive: boolean('is_active').default(true),
  
  // Audit fields
  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().notNull(),
  createdBy: uuid('created_by').references(() => users.id),
});
```

## variable_usage Table

Tracks which variables are used in each template for analytics and validation:

```typescript
export const variableUsage = pgTable('variable_usage', {
  id: uuid('id').defaultRandom().primaryKey(),
  tenantId: uuid('tenant_id').notNull().references(() => tenants.id),
  
  // References
  templateId: uuid('template_id').notNull().references(() => templates.id),
  variableId: uuid('variable_id').references(() => templateVariables.id),
  
  // Usage details
  variableName: text('variable_name').notNull(),
  usageCount: integer('usage_count').default(1),
  contexts: jsonb('contexts'), // Where in template: subject, body, etc.
  
  // Tracking
  firstUsed: timestamp('first_used').defaultNow().notNull(),
  lastUsed: timestamp('last_used').defaultNow().notNull(),
});
```

## variable_values Table

Stores recipient-specific variable values for custom fields:

```typescript
export const variableValues = pgTable('variable_values', {
  id: uuid('id').defaultRandom().primaryKey(),
  tenantId: uuid('tenant_id').notNull().references(() => tenants.id),
  
  // References
  recipientId: uuid('recipient_id').notNull().references(() => recipients.id),
  variableId: uuid('variable_id').notNull().references(() => templateVariables.id),
  
  // Value storage
  value: jsonb('value').notNull(),
  
  // Audit
  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().notNull(),
});
```

## Database Indexes

Optimize common query patterns with targeted indexes:

```typescript
// Optimize variable lookups by tenant
createIndex('idx_template_variables_tenant')
  .on(templateVariables)
  .using('btree', templateVariables.tenantId);

// Speed up usage queries
createIndex('idx_variable_usage_template')
  .on(variableUsage)
  .using('btree', variableUsage.templateId);

// Fast recipient value lookups
createIndex('idx_variable_values_recipient')
  .on(variableValues)
  .using('btree', variableValues.recipientId, variableValues.variableId);
```

## Related Documentation

- [VariableParser Service](./parser) - Template parsing implementation
- [API Endpoints](./api) - REST API for variable management
- [Schema Guide](/docs/implementation-technical/database-infrastructure/oltp-database/schema/) - Database design patterns
