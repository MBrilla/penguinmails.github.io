---
title: Template Variables API
description: REST API endpoints for template parsing, variable extraction, and validation
last_modified_date: 2025-01-15
level: 3
persona: ["developer"]
keywords: [api, rest, endpoints, parsing, validation]
---

{% raw %}

# Template Variables API

RESTful endpoints for variable management, template parsing, and validation.

## Parse Template

Render a template with provided variable values.

**Endpoint**: `POST /api/templates/parse`

```typescript
interface ParseRequest {
  template: string;
  variables: VariableContext;
  options?: ParseOptions;
}

interface ParseResponse {
  content: string;
  usedVariables: string[];
  missingVariables: string[];
  errors: ParseError[];
}

export async function POST(request: Request) {
  const { template, variables, options } = await request.json();
  
  const parser = new VariableParser();
  const result = parser.parse(template, variables, options);
  
  return Response.json(result);
}
```

## Extract Variables

Get list of variables used in a template.

**Endpoint**: `POST /api/templates/variables/extract`

```typescript
interface ExtractRequest {
  template: string;
}

interface ExtractResponse {
  variables: string[];
  categories: {
    standard: string[];
    custom: string[];
    system: string[];
  };
}

export async function POST(request: Request) {
  const { template } = await request.json();
  
  const parser = new VariableParser();
  const variables = parser.extractVariables(template);
  
  const categorized = {
    standard: variables.filter(v => !v.includes('.') && !v.startsWith('custom.')),
    custom: variables.filter(v => v.startsWith('custom.')),
    system: variables.filter(v => ['today', 'now', 'campaign_name', 'sender_name'].includes(v)),
  };
  
  return Response.json({ variables, categories: categorized });
}
```

## Validate Template

Check template syntax and variable availability.

**Endpoint**: `POST /api/templates/validate`

```typescript
interface ValidateRequest {
  template: string;
  tenantId: string;
  requiredVariables?: string[];
}

interface ValidateResponse {
  valid: boolean;
  errors: ValidationError[];
  warnings: ValidationWarning[];
  availableVariables: string[];
  missingVariables: string[];
}

export async function POST(request: Request) {
  const { template, tenantId, requiredVariables } = await request.json();
  
  const parser = new VariableParser();
  const usedVariables = parser.extractVariables(template);
  
  // Get available variables for tenant
  const availableVariables = await db
    .select()
    .from(templateVariables)
    .where(eq(templateVariables.tenantId, tenantId));
  
  const availableNames = new Set(availableVariables.map(v => v.name));
  const missing = usedVariables.filter(v => !availableNames.has(v.split('.')[0]));
  
  // Syntax validation
  const testResult = parser.parse(template, {}, { throwOnMissingVariable: false });
  
  return Response.json({
    valid: testResult.errors.length === 0 && missing.length === 0,
    errors: testResult.errors,
    warnings: [],
    availableVariables: Array.from(availableNames),
    missingVariables: missing,
  });
}
```

## List Available Variables

Get all variables available for a tenant.

**Endpoint**: `GET /api/templates/variables?tenantId={tenantId}`

```typescript
export async function GET(request: Request) {
  const { searchParams } = new URL(request.url);
  const tenantId = searchParams.get('tenantId');
  
  const variables = await db
    .select()
    .from(templateVariables)
    .where(
      and(
        eq(templateVariables.tenantId, tenantId),
        eq(templateVariables.isActive, true)
      )
    )
    .orderBy(templateVariables.category, templateVariables.name);
  
  const grouped = variables.reduce((acc, v) => {
    if (!acc[v.category]) acc[v.category] = [];
    acc[v.category].push({
      name: v.name,
      displayName: v.displayName,
      dataType: v.dataType,
      description: v.description,
      defaultValue: v.defaultValue,
      isRequired: v.isRequired,
    });
    return acc;
  }, {} as Record<string, typeof variables>);
  
  return Response.json({ variables: grouped });
}
```

## Performance Considerations

Template parsing performance is critical for high-volume email sending.

**Parser Caching**: Pre-parse templates and cache the parsed AST to avoid repeated parsing for each recipient.

**Batch Processing**: Process multiple recipients in parallel using worker pools.

**Variable Preloading**: Load all recipient variables in bulk rather than per-recipient queries.

**Lazy Evaluation**: Only resolve variables that are actually used in the template.

```typescript
async function renderBatch(
  template: string,
  recipients: Recipient[],
  batchSize: number = 100
): Promise<Map<string, string>> {
  const parser = new VariableParser();
  const results = new Map<string, string>();
  
  for (let i = 0; i < recipients.length; i += batchSize) {
    const batch = recipients.slice(i, i + batchSize);
    
    await Promise.all(batch.map(async (recipient) => {
      const context = await buildContext(recipient);
      const { content } = parser.parse(template, context);
      results.set(recipient.id, content);
    }));
  }
  
  return results;
}
```

## Error Handling

Implement robust error handling for production use.

```typescript
class TemplateParseError extends Error {
  constructor(
    message: string,
    public readonly errors: ParseError[],
    public readonly template: string
  ) {
    super(message);
    this.name = 'TemplateParseError';
  }
}

function parseWithValidation(
  template: string,
  context: VariableContext
): string {
  const parser = new VariableParser();
  const result = parser.parse(template, context, { strictMode: true });
  
  if (result.errors.length > 0) {
    throw new TemplateParseError(
      `Template parsing failed with ${result.errors.length} errors`,
      result.errors,
      template
    );
  }
  
  return result.content;
}
```

## Related Documentation

- [Database Schema](./schema) - Table definitions and indexes
- [VariableParser Service](./parser) - Parser implementation details
- [Quick Start](./quick-start) - Basic merge tag usage

{% endraw %}
