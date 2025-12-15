---
title: VariableParser Service
description: TypeScript implementation for template parsing, variable extraction, and content rendering
last_modified_date: 2025-01-15
level: 3
persona: ["developer"]
keywords: [parser, typescript, template-engine, filters, conditionals]
---

{% raw %}

# VariableParser Service

The VariableParser service handles template parsing, variable extraction, and content rendering. It processes merge tags, conditional blocks, and loops to produce personalized output.

## Type Definitions

```typescript
interface VariableContext {
  // Standard recipient fields
  first_name?: string;
  last_name?: string;
  full_name?: string;
  email: string;
  company_name?: string;
  job_title?: string;
  city?: string;
  country?: string;
  phone?: string;
  
  // Custom fields stored in jsonb
  custom?: Record<string, unknown>;
  
  // System variables
  today?: Date;
  now?: Date;
  campaign_name?: string;
  sender_name?: string;
  sender_email?: string;
  
  // Dynamic data for loops
  [key: string]: unknown;
}

interface ParseOptions {
  throwOnMissingVariable?: boolean;
  strictMode?: boolean;
  customFilters?: Record<string, FilterFunction>;
  maxLoopIterations?: number;
}

interface ParseResult {
  content: string;
  usedVariables: string[];
  missingVariables: string[];
  errors: ParseError[];
}

interface ParseError {
  type: 'missing_variable' | 'invalid_syntax' | 'filter_error' | 'loop_error';
  message: string;
  position: { line: number; column: number };
  variableName?: string;
}

type FilterFunction = (value: unknown, ...args: string[]) => string;
```

## Core Parser Implementation

```typescript
export class VariableParser {
  private filters: Map<string, FilterFunction>;
  private readonly MERGE_TAG_REGEX = /\{\{([^}]+)\}\}/g;
  private readonly CONDITIONAL_REGEX = /\{%\s*(if|elsif|else|endif)\s*([^%]*)\s*%\}/g;
  private readonly LOOP_REGEX = /\{%\s*for\s+(\w+)\s+in\s+(\w+)(?:\s+limit:\s*(\d+))?(?:\s+offset:\s*(\d+))?\s*%\}/g;
  private readonly ENDFOR_REGEX = /\{%\s*endfor\s*%\}/g;

  constructor(customFilters?: Record<string, FilterFunction>) {
    this.filters = new Map();
    this.registerDefaultFilters();
    
    if (customFilters) {
      Object.entries(customFilters).forEach(([name, fn]) => {
        this.filters.set(name, fn);
      });
    }
  }

  parse(template: string, context: VariableContext, options: ParseOptions = {}): ParseResult {
    const usedVariables: string[] = [];
    const missingVariables: string[] = [];
    const errors: ParseError[] = [];
    
    let content = template;
    
    // Process loops first
    content = this.parseLoops(content, context, options, usedVariables, errors);
    
    // Process conditionals
    content = this.parseConditionals(content, context, errors);
    
    // Process merge tags
    content = this.parseMergeTags(content, context, options, usedVariables, missingVariables, errors);
    
    return { content, usedVariables, missingVariables, errors };
  }

  extractVariables(template: string): string[] {
    const variables = new Set<string>();
    
    // Extract from merge tags
    let match;
    while ((match = this.MERGE_TAG_REGEX.exec(template)) !== null) {
      const [variablePath] = match[1].trim().split('|');
      variables.add(variablePath.trim());
    }
    
    // Extract from conditionals
    const conditionalVarRegex = /\{%\s*(?:if|elsif)\s+([^%]+)\s*%\}/g;
    while ((match = conditionalVarRegex.exec(template)) !== null) {
      const condition = match[1];
      const varMatch = condition.match(/^(\w+(?:\.\w+)*)/);
      if (varMatch) variables.add(varMatch[1]);
    }
    
    return Array.from(variables);
  }
}
```

## Default Filters

The parser includes built-in filters for common transformations:

```typescript
private registerDefaultFilters(): void {
  // String filters
  this.filters.set('upcase', (v) => String(v).toUpperCase());
  this.filters.set('downcase', (v) => String(v).toLowerCase());
  this.filters.set('capitalize', (v) => {
    const str = String(v);
    return str.charAt(0).toUpperCase() + str.slice(1).toLowerCase();
  });
  this.filters.set('strip', (v) => String(v).trim());
  this.filters.set('truncate', (v, length) => {
    const str = String(v);
    const len = parseInt(length) || 50;
    return str.length > len ? str.slice(0, len - 3) + '...' : str;
  });
  this.filters.set('replace', (v, search, replace) => 
    String(v).replace(new RegExp(search, 'g'), replace || '')
  );
  this.filters.set('remove', (v, search) => 
    String(v).replace(new RegExp(search, 'g'), '')
  );
  
  // Number filters
  this.filters.set('round', (v, precision) => {
    const num = parseFloat(String(v));
    const prec = parseInt(precision) || 0;
    return isNaN(num) ? v : num.toFixed(prec);
  });
  this.filters.set('floor', (v) => Math.floor(parseFloat(String(v))).toString());
  this.filters.set('ceil', (v) => Math.ceil(parseFloat(String(v))).toString());
  
  // Date filters
  this.filters.set('date', (v, format) => this.formatDate(v, format));
  
  // Default filter
  this.filters.set('default', (v, defaultVal) => 
    v === null || v === undefined || v === '' ? defaultVal : String(v)
  );
  
  // Array filters
  this.filters.set('join', (v, separator) => 
    Array.isArray(v) ? v.join(separator || ', ') : String(v)
  );
  this.filters.set('first', (v) => Array.isArray(v) ? v[0] : v);
  this.filters.set('last', (v) => Array.isArray(v) ? v[v.length - 1] : v);
  this.filters.set('size', (v) => 
    Array.isArray(v) ? v.length.toString() : String(v).length.toString()
  );
}
```

## Date Formatting

Transform date values using format tokens:

```typescript
private formatDate(value: unknown, format: string): string {
  const date = value instanceof Date ? value : new Date(String(value));
  if (isNaN(date.getTime())) return String(value);
  
  const tokens: Record<string, string> = {
    '%Y': date.getFullYear().toString(),
    '%y': date.getFullYear().toString().slice(-2),
    '%m': (date.getMonth() + 1).toString().padStart(2, '0'),
    '%d': date.getDate().toString().padStart(2, '0'),
    '%e': date.getDate().toString(),
    '%H': date.getHours().toString().padStart(2, '0'),
    '%I': (date.getHours() % 12 || 12).toString().padStart(2, '0'),
    '%M': date.getMinutes().toString().padStart(2, '0'),
    '%p': date.getHours() >= 12 ? 'PM' : 'AM',
    '%B': date.toLocaleString('en-US', { month: 'long' }),
    '%b': date.toLocaleString('en-US', { month: 'short' }),
    '%A': date.toLocaleString('en-US', { weekday: 'long' }),
    '%a': date.toLocaleString('en-US', { weekday: 'short' }),
  };
  
  let result = format;
  Object.entries(tokens).forEach(([token, value]) => {
    result = result.replace(new RegExp(token, 'g'), value);
  });
  
  return result;
}
```

## Merge Tag Processing

Parse and replace merge tags with values from context:

```typescript
private parseMergeTags(
  template: string,
  context: VariableContext,
  options: ParseOptions,
  usedVariables: string[],
  missingVariables: string[],
  errors: ParseError[]
): string {
  return template.replace(this.MERGE_TAG_REGEX, (match, expression) => {
    const [variablePath, ...filterChain] = expression.trim().split('|').map((s: string) => s.trim());
    
    // Get variable value
    let value = this.getNestedValue(context, variablePath);
    usedVariables.push(variablePath);
    
    // Track missing variables
    if (value === undefined || value === null) {
      missingVariables.push(variablePath);
    }
    
    // Apply filters
    for (const filterExpr of filterChain) {
      const [filterName, ...args] = filterExpr.split(':').map((s: string) => s.trim());
      const filter = this.filters.get(filterName);
      
      if (filter) {
        value = filter(value, ...args.map(a => a.replace(/^["']|["']$/g, '')));
      } else {
        errors.push({
          type: 'filter_error',
          message: `Unknown filter: ${filterName}`,
          position: { line: 0, column: 0 },
        });
      }
    }
    
    return value === undefined || value === null ? '' : String(value);
  });
}

private getNestedValue(obj: Record<string, unknown>, path: string): unknown {
  const parts = path.split('.');
  let current: unknown = obj;
  
  for (const part of parts) {
    if (current === null || current === undefined) return undefined;
    if (typeof current !== 'object') return undefined;
    current = (current as Record<string, unknown>)[part];
  }
  
  return current;
}
```

## Conditional Processing

Handle if/elsif/else/endif blocks:

```typescript
private parseConditionals(
  template: string,
  context: VariableContext,
  errors: ParseError[]
): string {
  let result = template;
  let iterations = 0;
  const maxIterations = 100;
  
  while (result.includes('{%') && iterations < maxIterations) {
    iterations++;
    result = this.processConditionalBlock(result, context, errors);
  }
  
  return result;
}

private evaluateCondition(condition: string, context: VariableContext): boolean {
  const comparisonMatch = condition.match(/(\S+)\s*(==|!=|>|<|>=|<=|contains)\s*(.+)/);
  
  if (comparisonMatch) {
    const [, left, operator, right] = comparisonMatch;
    const leftValue = this.resolveValue(left.trim(), context);
    const rightValue = this.resolveValue(right.trim(), context);
    
    switch (operator) {
      case '==': return leftValue == rightValue;
      case '!=': return leftValue != rightValue;
      case '>': return Number(leftValue) > Number(rightValue);
      case '<': return Number(leftValue) < Number(rightValue);
      case '>=': return Number(leftValue) >= Number(rightValue);
      case '<=': return Number(leftValue) <= Number(rightValue);
      case 'contains': return String(leftValue).includes(String(rightValue));
    }
  }
  
  const value = this.getNestedValue(context, condition);
  return value !== null && value !== undefined && value !== '' && value !== false;
}

private resolveValue(expr: string, context: VariableContext): unknown {
  if ((expr.startsWith('"') && expr.endsWith('"')) || 
      (expr.startsWith("'") && expr.endsWith("'"))) {
    return expr.slice(1, -1);
  }
  if (!isNaN(Number(expr))) return Number(expr);
  if (expr === 'true') return true;
  if (expr === 'false') return false;
  if (expr === 'blank') return '';
  return this.getNestedValue(context, expr);
}
```

## Loop Processing

Iterate over arrays to generate repeated content:

```typescript
private parseLoops(
  template: string,
  context: VariableContext,
  options: ParseOptions,
  usedVariables: string[],
  errors: ParseError[]
): string {
  const maxIterations = options.maxLoopIterations || 100;
  
  return template.replace(
    /\{%\s*for\s+(\w+)\s+in\s+(\w+)(?:\s+limit:\s*(\d+))?(?:\s+offset:\s*(\d+))?\s*%\}([\s\S]*?)\{%\s*endfor\s*%\}/g,
    (match, itemVar, arrayVar, limit, offset, content) => {
      const array = this.getNestedValue(context, arrayVar);
      usedVariables.push(arrayVar);
      
      if (!Array.isArray(array)) {
        errors.push({
          type: 'loop_error',
          message: `Variable ${arrayVar} is not an array`,
          position: { line: 0, column: 0 },
          variableName: arrayVar,
        });
        return '';
      }
      
      let items = array;
      if (offset) items = items.slice(parseInt(offset));
      if (limit) items = items.slice(0, parseInt(limit));
      items = items.slice(0, maxIterations);
      
      return items.map((item, index) => {
        const loopContext = {
          ...context,
          [itemVar]: item,
          forloop: {
            index: index + 1,
            index0: index,
            first: index === 0,
            last: index === items.length - 1,
            length: items.length,
          },
        };
        
        return this.parseMergeTags(content, loopContext, options, usedVariables, [], errors);
      }).join('');
    }
  );
}
```

## Related Documentation

- [Database Schema](./schema) - Table definitions and indexes
- [API Endpoints](./api) - REST API for template operations
- [Advanced Usage](./advanced-usage) - User-facing template syntax

{% endraw %}
