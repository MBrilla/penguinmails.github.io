---
title: Template Variables Advanced Usage
description: Custom fields, complex conditionals, loops, date formatting, and dynamic content manipulation
last_modified_date: 2025-01-15
level: 2
persona: ["email-marketer", "power-user"]
keywords: [custom-fields, loops, conditionals, formatting, dynamic-content]
---

{% raw %}

# Template Variables Advanced Usage

This guide covers advanced personalization techniques including custom field access, complex conditional logic, loops for repeating content, and data formatting. These features enable sophisticated, data-driven email personalization.

## Custom Fields

Access custom recipient data stored in your CRM or uploaded via CSV. Custom fields use dot notation to access nested properties.

### Basic Custom Field Access

```liquid
{{custom.customer_id}}
{{custom.account_manager}}
{{custom.renewal_date}}
```

### Nested Property Access

For complex data structures, chain property names with dots:

```liquid
{{custom.subscription.plan_name}}
{{custom.subscription.seats_used}}
{{custom.company.address.street}}
{{custom.company.address.postal_code}}
```

### Custom Fields with Defaults

Apply defaults to custom fields the same way as standard variables:

```liquid
Your account manager is {{custom.account_manager | default: "our support team"}}.
Your plan: {{custom.subscription.plan_name | default: "Standard"}}
```

### Available Custom Field Types

| Type | Storage | Example Value |
|------|---------|---------------|
| Text | String | "Enterprise Plan" |
| Number | Integer/Float | 42, 99.95 |
| Boolean | true/false | true |
| Date | ISO 8601 | "2025-03-15" |
| List | Array | ["tag1", "tag2"] |
| Object | Nested JSON | `{city: "NYC", zip: "10001"}` |

## Advanced Conditional Logic

Build complex conditions using AND/OR operators and nested logic.

### AND Conditions

All conditions must be true for the block to render:

```liquid
{% if plan == "enterprise" and employees > 100 %}
As a large enterprise customer, you qualify for dedicated support.
{% endif %}

{% if industry == "technology" and country == "United States" and revenue > 1000000 %}
You're eligible for our US Tech Enterprise program.
{% endif %}
```

### OR Conditions

Any condition being true renders the block:

```liquid
{% if plan == "pro" or plan == "enterprise" %}
Thank you for being a premium customer!
{% endif %}

{% if country == "United States" or country == "Canada" or country == "Mexico" %}
Take advantage of our North American pricing.
{% endif %}
```

### Combined AND/OR Logic

Use parentheses for complex combinations:

```liquid
{% if (plan == "enterprise" or plan == "pro") and status == "active" %}
Your premium features are ready to use.
{% endif %}

{% if industry == "healthcare" and (employees > 500 or revenue > 10000000) %}
Explore our healthcare enterprise solutions.
{% endif %}
```

### Negation with NOT

Invert conditions using `not`:

```liquid
{% if not opted_out %}
Subscribe to our newsletter for weekly tips.
{% endif %}

{% if plan != "free" and not trial_expired %}
Your premium access is active.
{% endif %}
```

### Contains Operator

Check if a string contains a substring or array contains an element:

```liquid
{% if email contains "@enterprise.com" %}
Welcome, enterprise team member!
{% endif %}

{% if tags contains "vip" %}
As a VIP customer, you get priority access.
{% endif %}
```

## Dynamic Links and Images

Generate personalized URLs and image sources based on recipient data.

### Personalized Links

```liquid
<a href="https://app.penguinmails.com/dashboard?user={{email}}&source=email">
  View Your Dashboard
</a>

<a href="https://calendly.com/{{custom.account_manager_calendar}}">
  Schedule a Call
</a>
```

### Tracking Parameters

Add recipient data to tracking URLs:

```liquid
<a href="https://example.com/offer?
  utm_campaign={{campaign_name}}&
  utm_content={{first_name}}&
  customer_id={{custom.customer_id}}">
  Claim Your Offer
</a>
```

### Dynamic Images

Personalize images based on recipient attributes:

```liquid
<img src="https://cdn.example.com/banners/{{industry | default: 'general'}}.png" 
     alt="Industry-specific banner">

{% if country == "United States" %}
<img src="https://cdn.example.com/flags/us.png" alt="US Flag">
{% elsif country == "United Kingdom" %}
<img src="https://cdn.example.com/flags/uk.png" alt="UK Flag">
{% endif %}
```

## Date Formatting

Transform date values into readable formats using date filters.

### Basic Date Formatting

```liquid
Account created: {{created_at | date: "%B %d, %Y"}}
<!-- Output: Account created: March 15, 2025 -->

Renewal: {{custom.renewal_date | date: "%m/%d/%Y"}}
<!-- Output: Renewal: 03/15/2025 -->
```

### Format Codes

| Code | Description | Example |
|------|-------------|---------|
| `%Y` | 4-digit year | 2025 |
| `%y` | 2-digit year | 25 |
| `%m` | Month (01-12) | 03 |
| `%B` | Full month name | March |
| `%b` | Abbreviated month | Mar |
| `%d` | Day of month (01-31) | 15 |
| `%e` | Day of month (1-31) | 15 |
| `%A` | Full weekday | Saturday |
| `%a` | Abbreviated weekday | Sat |
| `%H` | Hour (00-23) | 14 |
| `%I` | Hour (01-12) | 02 |
| `%M` | Minutes (00-59) | 30 |
| `%p` | AM/PM | PM |

### Common Date Formats

```liquid
{{date | date: "%B %d, %Y"}}          <!-- March 15, 2025 -->
{{date | date: "%m/%d/%Y"}}            <!-- 03/15/2025 -->
{{date | date: "%d-%m-%Y"}}            <!-- 15-03-2025 -->
{{date | date: "%A, %B %e"}}           <!-- Saturday, March 15 -->
{{date | date: "%b %e at %I:%M %p"}}   <!-- Mar 15 at 02:30 PM -->
```

### Relative Date References

Use special keywords for dynamic dates:

```liquid
Today is {{today | date: "%B %d, %Y"}}.
This email was sent on {{now | date: "%A at %I:%M %p"}}.
```

## Loops

Iterate over lists to generate repeating content sections.

### Basic Loop Syntax

```liquid
{% for product in recommended_products %}
- {{product.name}}: ${{product.price}}
{% endfor %}
```

### Loop with Index

Access the current iteration number:

```liquid
Your top 3 recommendations:
{% for item in recommendations %}
{{forloop.index}}. {{item.title}} - {{item.description}}
{% endfor %}
```

### Loop Properties

| Property | Description |
|----------|-------------|
| `forloop.index` | Current iteration (1-based) |
| `forloop.index0` | Current iteration (0-based) |
| `forloop.first` | True if first iteration |
| `forloop.last` | True if last iteration |
| `forloop.length` | Total items in loop |

### Conditional Content in Loops

```liquid
{% for activity in recent_activities %}
{% if forloop.first %}
Your most recent activity:
{% endif %}
- {{activity.date | date: "%b %d"}}: {{activity.action}}
{% if forloop.last %}
View all activity in your dashboard.
{% endif %}
{% endfor %}
```

### Limiting Loop Iterations

```liquid
{% for product in products limit: 5 %}
{{product.name}}
{% endfor %}

{% for product in products offset: 2 limit: 3 %}
<!-- Skip first 2, show next 3 -->
{{product.name}}
{% endfor %}
```

### Empty Loop Handling

```liquid
{% for notification in pending_notifications %}
- {{notification.message}}
{% else %}
You have no pending notifications.
{% endfor %}
```

## String Manipulation

Transform text values using string filters.

### Case Transformation

```liquid
{{first_name | upcase}}      <!-- SARAH -->
{{first_name | downcase}}    <!-- sarah -->
{{first_name | capitalize}}  <!-- Sarah -->
```

### String Trimming

```liquid
{{input | strip}}            <!-- Remove leading/trailing whitespace -->
{{input | lstrip}}           <!-- Remove leading whitespace -->
{{input | rstrip}}           <!-- Remove trailing whitespace -->
```

### Text Replacement

```liquid
{{email | replace: "@", " at "}}
<!-- sarah at example.com -->

{{phone | remove: "-"}}
<!-- 5550123 -->
```

### String Truncation

```liquid
{{description | truncate: 50}}
<!-- First 50 characters with ellipsis -->

{{title | truncatewords: 10}}
<!-- First 10 words with ellipsis -->
```

### Splitting and Joining

```liquid
{% assign tags = tag_string | split: "," %}
{% for tag in tags %}
- {{tag | strip}}
{% endfor %}

{{tag_array | join: ", "}}
<!-- tag1, tag2, tag3 -->
```

## Number Formatting

Format numeric values for display.

### Basic Number Formatting

```liquid
Total: ${{amount | money}}
<!-- Total: $1,234.56 -->

{{percentage}}%
<!-- 75% -->
```

### Decimal Places

```liquid
{{price | round: 2}}         <!-- 19.99 -->
{{average | floor}}          <!-- 19 -->
{{score | ceil}}             <!-- 20 -->
```

### Number with Separators

```liquid
{{revenue | number_with_delimiter}}
<!-- 1,234,567 -->

{{value | number_with_precision: 2}}
<!-- 1234.50 -->
```

## Practical Examples

### Personalized Product Recommendations

```liquid
Hi {{first_name}},

Based on your recent activity, we think you'll love these:

{% for product in recommended_products limit: 3 %}
**{{product.name}}**
{{product.description | truncate: 100}}
${{product.price | round: 2}} - [View Product]({{product.url}})

{% endfor %}

{% if recommended_products.size > 3 %}
[See all {{recommended_products.size}} recommendations →](https://app.example.com/recommendations)
{% endif %}
```

### Account Summary Email

```liquid
Hi {{first_name}},

Here's your account summary for {{today | date: "%B %Y"}}:

**Plan**: {{custom.subscription.plan_name | default: "Free"}}
**Emails Sent**: {{custom.usage.emails_sent | number_with_delimiter}}
**Open Rate**: {{custom.usage.open_rate | round: 1}}%

{% if custom.usage.emails_sent > custom.subscription.limit * 0.8 %}
⚠️ You've used {{custom.usage.emails_sent}}/{{custom.subscription.limit}} emails this month.
[Upgrade your plan](https://app.example.com/upgrade) to avoid interruptions.
{% endif %}

{% for alert in account_alerts %}
- {{alert.message}}
{% else %}
✓ No issues with your account.
{% endfor %}
```

### Event Invitation with Location Logic

```liquid
{% if custom.event_registered %}
You're registered for {{event_name}} on {{event_date | date: "%A, %B %e"}}.

{% if custom.attendance_type == "in-person" %}
**Venue**: {{venue_name}}
{{venue_address}}

[Get Directions](https://maps.google.com/?q={{venue_address | url_encode}})
{% else %}
**Virtual Event**
Join link: {{virtual_event_url}}

The event starts at {{event_time}} in your timezone ({{custom.timezone | default: "UTC"}}).
{% endif %}

{% else %}
You're invited to {{event_name}}!

{% if spots_remaining < 10 %}
Only {{spots_remaining}} spots left - [Register Now]({{registration_url}})
{% else %}
[Reserve Your Spot]({{registration_url}})
{% endif %}
{% endif %}
```

## Best Practices

**Validate custom field paths** before deployment to ensure the data structure matches your template expectations.

**Use loops sparingly** for email content since very long lists impact readability and deliverability.

**Test edge cases** including empty arrays, missing nested properties, and null values.

**Keep formatting consistent** across templates for brand coherence.

**Document complex logic** with comments for future template maintainers.

## Next Steps

For database schema details, parser implementation, and API integration, see [Technical Implementation](./technical).

{% endraw %}
