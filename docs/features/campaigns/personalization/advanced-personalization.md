---
title: "Advanced Personalization"
description: "Conditional content, behavioral personalization, dynamic CTAs, and localization"
level: "2"
status: "PLANNED"
roadmap_timeline: "Q1 2026"
priority: "High"
keywords: [conditional-content, behavioral, localization, dynamic-cta, personalization]
---

# Advanced Personalization

Advanced personalization techniques including conditional content blocks, behavioral targeting, dynamic CTAs, and location-based content.

## Conditional Content

Show different content blocks based on contact attributes:

### Basic Conditional

```markdown
{% if company %}
  We've helped companies like {{company}} achieve...
{% else %}
  We've helped businesses like yours achieve...
{% endif %}
```

### Multiple Conditions

```markdown
{% if leadScore > 50 %}
  You're a valued customer! Here's an exclusive offer...
{% elsif leadScore > 20 %}
  Thanks for your interest! Let us show you more...
{% else %}
  Welcome! Here's how we can help...
{% endif %}
```

### Conditional by Industry

```markdown
{% if customField.industry == "SaaS" %}
  Our platform integrates seamlessly with your tech stack.
{% elsif customField.industry == "E-commerce" %}
  Boost your cart abandonment recovery by 35%.
{% else %}
  Discover how email automation can grow your business.
{% endif %}
```

## Personalized CTAs

Adapt call-to-action based on contact status:

```text
{% if customField.trialUser %}
  <a href="{{upgradeLink}}">Upgrade to Premium</a>
{% elsif customField.freeUser %}
  <a href="{{trialLink}}">Start Your Free Trial</a>
{% else %}
  <a href="{{signupLink}}">Get Started Free</a>
{% endif %}
```

## Date-Based Personalization

Use date fields for dynamic content:

{% raw %}

```markdown
{% if customField.subscriptionRenewalDate < now + 30days %}
  Your subscription renews on {{customField.subscriptionRenewalDate|date('F j, Y')}}.
  Renew now and save 20%!
{% endif %}
```

{% endraw %}

## Behavioral Personalization

Personalize based on past actions:

{% raw %}

```markdown
{% if customField.lastPurchaseDate > now - 90days %}
  Thanks for your recent purchase! Here's what's new...
{% else %}
  We miss you! Come back and save 15%...
{% endif %}
```

{% endraw %}

## Dynamic Product Recommendations

Show relevant products:

```text
Based on your interest in {{customField.lastViewedProduct}},
you might also like:

{% for product in relatedProducts %}
  - {{product.name}} - ${{product.price}}
{% endfor %}
```

## Localization

Adapt content by location:

```text
{% if country == "United States" %}
  Free shipping on orders over $50!
{% elsif country == "Canada" %}
  Free shipping on orders over CAD $75!
{% elsif country == "United Kingdom" %}
  Free shipping on orders over £40!
{% endif %}
```

## A/B Testing with Personalization

Combine with A/B testing:

```text
Test A: Hi {{firstName}},
Test B: Hi {{firstName}}, fellow {{customField.industry}} professional,
```

## Related Documentation

- [Personalization Overview](/docs/features/campaigns/personalization/overview) - Basic merge tags and quick start
- [Technical Implementation](/docs/features/campaigns/personalization/technical-implementation) - Engine and database
- [A/B Testing](/docs/features/campaigns/ab-testing) - Test personalized variants

---

**Last Updated:** November 25, 2025
**Status:** Planned - MVP Feature (Level 2)
**Target Release:** Q1 2026
