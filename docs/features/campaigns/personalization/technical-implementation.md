---
title: "Personalization Technical Implementation"
description: "Templating engine, database schema, validation, and performance optimization for personalization"
level: "3"
status: "PLANNED"
roadmap_timeline: "Q1 2026"
priority: "High"
keywords: [templating-engine, database-schema, validation, performance, technical]
---

# Personalization Technical Implementation

Technical implementation details for the Personalization System including templating engine, database schema, validation, and performance optimization.

## Templating Engine

Uses Liquid-based syntax for compatibility:

```typescript
import { Liquid } from 'liquidjs';

const engine = new Liquid({
  cache: true,
  strictFilters: true,
  strictVariables: false, // Allow missing vars with fallbacks
});

// Register custom filters
engine.registerFilter('phoneFormat', (phone: string) => {
  return phone.replace(/(\d{3})(\d{3})(\d{4})/, '($1) $2-$3');
});

engine.registerFilter('currency', (amount: number, currency = 'USD') => {
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency,
  }).format(amount);
});
```

## Merge Tag Resolution

```typescript
interface PersonalizationContext {
  contact: Contact;
  campaign: Campaign;
  customFields: Record<string, any>;
  computed: Record<string, any>;
}

async function renderPersonalizedEmail(
  template: string,
  contact: Contact
): Promise<string> {
  // Build context
  const context: PersonalizationContext = {
    contact: {
      firstName: contact.firstName,
      lastName: contact.lastName,
      email: contact.email,
      company: contact.company,
      // ... other standard fields
    },
    customFields: contact.customFields || {},
    computed: await this.computeDynamicFields(contact),
  };

  // Render template
  try {
    const rendered = await engine.parseAndRender(template, context);
    return rendered;
  } catch (error) {
    logger.error('Personalization error:', error);
    // Fallback to unpersonalized template
    return this.stripMergeTags(template);
  }
}

async function computeDynamicFields(contact: Contact): Promise<Record<string, any>> {
  return {
    daysSinceSignup: differenceInDays(new Date(), contact.createdAt),
    isRecentCustomer: contact.lastPurchaseDate &&
      differenceInDays(new Date(), contact.lastPurchaseDate) < 90,
    recommendedProducts: await this.getRecommendations(contact),
  };
}
```

## Database Schema

```sql
-- Contact custom fields (JSONB for flexibility)
CREATE TABLE contacts (
  id UUID PRIMARY KEY,
  tenant_id UUID NOT NULL,

  -- Standard fields
  email VARCHAR(255) UNIQUE NOT NULL,
  first_name VARCHAR(100),
  last_name VARCHAR(100),
  company VARCHAR(255),
  job_title VARCHAR(100),
  city VARCHAR(100),
  state VARCHAR(100),
  country VARCHAR(100),

  -- Custom fields (schemaless)
  custom_fields JSONB DEFAULT '{}',

  -- Metadata
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Index for custom field queries
CREATE INDEX idx_contacts_custom_fields ON contacts USING GIN(custom_fields);

-- Custom field definitions (for UI/validation)
CREATE TABLE custom_field_definitions (
  id UUID PRIMARY KEY,
  tenant_id UUID NOT NULL,

  field_name VARCHAR(100) NOT NULL,
  field_type VARCHAR(50), -- text, number, date, boolean, dropdown
  field_label VARCHAR(255),

  -- Validation
  is_required BOOLEAN DEFAULT FALSE,
  default_value TEXT,
  options JSONB, -- For dropdown fields

  -- Metadata
  created_at TIMESTAMP DEFAULT NOW(),

  UNIQUE(tenant_id, field_name)
);
```

## Personalization Validation

{% raw %}

```typescript
class PersonalizationValidator {
  async validateTemplate(template: string, tenantId: string): Promise<ValidationResult> {
    const errors: ValidationError[] = [];
    const warnings: ValidationWarning[] = [];

    // Find all merge tags
    const mergeTags = this.extractMergeTags(template);

    for (const tag of mergeTags) {
      // Check if field exists
      const fieldExists = await this.fieldExists(tag, tenantId);

      if (!fieldExists) {
        warnings.push({
          tag,
          message: `Field "${tag}" not found in contact schema. Will use fallback.`,
        });
      }

      // Check for proper fallback syntax
      if (!tag.includes('|') && !fieldExists) {
        warnings.push({
          tag,
          message: `Consider adding fallback: {{${tag}|default value}}`,
        });
      }
    }

    // Find conditional blocks
    const conditionals = this.extractConditionals(template);

    for (const conditional of conditionals) {
      try {
        await engine.parse(conditional);
      } catch (error) {
        errors.push({
          block: conditional,
          message: `Syntax error: ${error.message}`,
        });
      }
    }

    return {
      isValid: errors.length === 0,
      errors,
      warnings,
    };
  }

  private extractMergeTags(template: string): string[] {
    const regex = /\{\{([^}]+)\}\}/g;
    const matches = template.matchAll(regex);
    return Array.from(matches).map(m => m[1].trim().split('|')[0]);
  }
}
```

{% endraw %}

## Performance Optimization

```typescript
// Batch personalization for bulk sends
async function batchPersonalize(
  template: string,
  contacts: Contact[]
): Promise<Map<string, string>> {
  const resultsMap = new Map<string, string>();

  // Parse template once
  const parsedTemplate = await engine.parse(template);

  // Render for each contact (parallel)
  await Promise.all(
    contacts.map(async (contact) => {
      const context = await this.buildContext(contact);
      const rendered = await engine.render(parsedTemplate, context);
      resultsMap.set(contact.id, rendered);
    })
  );

  return resultsMap;
}

// Cache compiled templates
const templateCache = new LRU<string, Template>({
  max: 1000,
  ttl: 1000 * 60 * 60, // 1 hour
});

async function renderCached(templateId: string, context: any): Promise<string> {
  let compiled = templateCache.get(templateId);

  if (!compiled) {
    const template = await db.templates.findById(templateId);
    compiled = await engine.parse(template.content);
    templateCache.set(templateId, compiled);
  }

  return engine.render(compiled, context);
}
```

## Related Documentation

- [Personalization Overview](/docs/features/campaigns/personalization/overview) - Basic merge tags and quick start
- [Advanced Personalization](/docs/features/campaigns/personalization/advanced-personalization) - Conditional content
- [Template Management](/docs/features/templates/template-management) - Build reusable templates

---

**Last Updated:** November 25, 2025
**Status:** Planned - MVP Feature (Level 3)
**Target Release:** Q1 2026
**Owner:** Campaigns Team
