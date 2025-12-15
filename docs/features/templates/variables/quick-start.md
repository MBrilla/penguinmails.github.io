---
title: Template Variables Quick Start
description: Basic merge tags, default values, and simple conditional logic for email personalization
last_modified_date: 2025-01-15
level: 1
persona: ["email-marketer"]
keywords: [merge-tags, personalization, defaults, conditionals]
---

{% raw %}

# Template Variables Quick Start

This guide covers the essential merge tag syntax and basic personalization features. You'll learn to insert recipient data, handle missing values gracefully, and create simple conditional content.

## Basic Merge Tags

Merge tags insert recipient-specific data into your email content using double curly braces. The system replaces these placeholders with actual values when sending each email.

### Standard Variables

Common recipient fields are available as simple variable names:

```liquid
Hello {{first_name}},

Thank you for your interest in {{company_name}}.
We noticed you're based in {{city}}, {{country}}.

Best regards,
{{sender_name}}
{{sender_title}}
```

### Available Standard Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `{{first_name}}` | Recipient's first name | Sarah |
| `{{last_name}}` | Recipient's last name | Johnson |
| `{{full_name}}` | Combined first and last name | Sarah Johnson |
| `{{email}}` | Email address | sarah@example.com |
| `{{company_name}}` | Company/organization name | Acme Corp |
| `{{job_title}}` | Professional title | Marketing Director |
| `{{city}}` | City location | San Francisco |
| `{{country}}` | Country | United States |
| `{{phone}}` | Phone number | +1-555-0123 |

## Default Values

When recipient data is missing, default values prevent awkward blank spaces or broken personalization. Use the pipe character followed by `default:` to specify fallback text.

### Basic Default Syntax

```liquid
Hello {{first_name | default: "there"}},

Your role as {{job_title | default: "a professional"}} at 
{{company_name | default: "your organization"}} caught our attention.
```

### Chained Defaults

Try multiple fields before falling back to a static default:

```liquid
Hi {{first_name | default: nickname | default: "Friend"}},
```

The system checks `first_name` first, then `nickname`, and finally uses "Friend" if both are empty.

### Empty Value Handling

By default, missing variables render as empty strings. Use defaults to maintain natural-sounding copy:

| Without Default | With Default |
|-----------------|--------------|
| "Hello ," | "Hello there," |
| "Your role at caught..." | "Your role at your company caught..." |

## Conditional Content

Show or hide content blocks based on recipient data. Conditional logic uses `{% if %}` tags to wrap content that should only appear when conditions are met.

### Basic If Statement

```liquid
{% if company_name %}
I noticed you work at {{company_name}} and thought you might be interested in our enterprise solutions.
{% endif %}
```

The content inside the `{% if %}` block only appears when `company_name` has a value.

### If-Else Statements

Provide alternative content when conditions aren't met:

```liquid
{% if job_title %}
As a {{job_title}}, you understand the importance of efficient workflows.
{% else %}
As a professional, you understand the importance of efficient workflows.
{% endif %}
```

### Multiple Conditions with Elsif

Handle multiple scenarios with chained conditions:

```liquid
{% if industry == "technology" %}
Tech companies like yours are seeing 40% efficiency gains.
{% elsif industry == "healthcare" %}
Healthcare organizations are improving patient communication by 60%.
{% elsif industry == "finance" %}
Financial services firms are reducing compliance risks significantly.
{% else %}
Companies in your industry are transforming their outreach.
{% endif %}
```

### Comparison Operators

Conditionals support standard comparison operators:

| Operator | Description | Example |
|----------|-------------|---------|
| `==` | Equals | `{% if status == "active" %}` |
| `!=` | Not equals | `{% if plan != "free" %}` |
| `>` | Greater than | `{% if score > 50 %}` |
| `<` | Less than | `{% if days_since_signup < 30 %}` |
| `>=` | Greater or equal | `{% if employees >= 100 %}` |
| `<=` | Less or equal | `{% if revenue <= 1000000 %}` |

### Checking for Empty Values

Use `blank` to check if a field has no value:

```liquid
{% if phone != blank %}
Call us at {{phone}} for immediate assistance.
{% endif %}

{% if notes == blank %}
We'd love to learn more about your specific needs.
{% endif %}
```

## Practical Examples

### Personalized Introduction

```liquid
Hi {{first_name | default: "there"}},

{% if company_name %}
I came across {{company_name}} while researching companies in the {{industry | default: "your"}} space.
{% else %}
I came across your profile and was impressed by your background.
{% endif %}

{% if job_title %}
As a {{job_title}}, you're probably familiar with the challenges of managing outreach at scale.
{% endif %}
```

### Call-to-Action Personalization

```liquid
{% if plan == "free" %}
Upgrade to Pro today and unlock unlimited campaigns.
{% elsif plan == "pro" %}
You're getting great results! Consider Enterprise for team features.
{% else %}
Schedule a demo to see what PenguinMails can do for you.
{% endif %}
```

### Location-Based Content

```liquid
{% if country == "United States" %}
Join us at our upcoming conference in San Francisco.
{% elsif country == "United Kingdom" %}
Meet our team at the London marketing summit next month.
{% else %}
Attend our virtual event from anywhere in the world.
{% endif %}
```

## Best Practices

**Always use defaults** for critical personalization points like greetings to avoid awkward blank spaces.

**Test with missing data** by previewing emails with incomplete recipient records to catch personalization failures.

**Keep conditionals simple** at this level. Complex logic with multiple AND/OR conditions is covered in the [Advanced Usage](./advanced-usage) guide.

**Use consistent variable naming** across your templates to simplify maintenance and reduce errors.

## Next Steps

Once comfortable with basic merge tags and conditionals, explore [Advanced Usage](./advanced-usage) for:

- Custom field access with nested properties
- Complex conditional logic with AND/OR operators
- Date formatting and calculations
- Loops for repeating content
- String and number manipulation

{% endraw %}
