---
title: "Personalization System"
description: "Dynamic email personalization with merge tags, conditional content, and custom fields"
level: "2"
status: "PLANNED"
roadmap_timeline: "Q1 2026"
priority: "High"
keywords: [personalization, merge-tags, conditional-content, dynamic-emails]
related_features:
  - campaigns/campaign-management/overview
  - campaigns/ab-testing
  - templates/template-management
  - leads/leads-management
---

# Personalization System

**Quick Access**: Create dynamic, personalized emails using merge tags, conditional content blocks, and custom data fields.

## Overview

The Personalization System enables 1:1 email customization at scale by dynamically inserting contact data, displaying conditional content, and adapting messaging based on recipient attributes and behavior.

### Key Capabilities

| Capability | Description |
|------------|-------------|
| Merge Tags | Insert contact data ({{firstName}}, {{company}}, etc.) |
| Conditional Content | Show/hide blocks based on conditions |
| Custom Fields | Use any contact attribute for personalization |
| Fallback Values | Default content when data is missing |
| Preview & Testing | Test personalization before sending |

## Quick Start Guide

### Basic Merge Tags

Insert contact information directly into your emails:

```markdown
Hi {{firstName}},

I noticed you're from {{company}} in {{city}}.
We've helped companies like yours increase email
performance by {{customField.industryBenchmark}}%.

Best regards,
{{senderName}}
```

### Standard Merge Tags

{% raw %}

| Tag | Description |
|-----|-------------|
| `{{firstName}}` | Contact's first name |
| `{{lastName}}` | Contact's last name |
| `{{email}}` | Email address |
| `{{company}}` | Company name |
| `{{jobTitle}}` | Job title |
| `{{city}}` | City |
| `{{state}}` | State/Province |
| `{{country}}` | Country |

{% endraw %}

### Fallback Values

Provide defaults when data is missing:

{% raw %}

```markdown
Hi {{firstName|there}},

{{company|"Your company"}} could benefit from...
```

{% endraw %}

**Result:**
- If firstName exists: "Hi Sarah,"
- If firstName missing: "Hi there,"

### Using Custom Fields

Access any custom field from your contact records:

```markdown
{{customField.subscribedDate}}
{{customField.leadScore}}
{{customField.industry}}
{{customField.lastPurchaseAmount}}
```

### Preview Personalization

Before sending, preview how emails appear to different contacts:

1. Click "Preview Personalization"
2. Select contact or enter test data
3. View rendered email
4. Test multiple contacts to ensure formatting

## Related Documentation

- [Advanced Personalization](/docs/features/campaigns/personalization/advanced-personalization) - Conditional content, behavioral
- [Technical Implementation](/docs/features/campaigns/personalization/technical-implementation) - Engine, database, validation
- [Campaign Management](/docs/features/campaigns/campaign-management/hub) - Create personalized campaigns
- [A/B Testing](/docs/features/campaigns/ab-testing) - Test personalized variants

---

**Last Updated:** November 25, 2025
**Status:** Planned - MVP Feature (Level 2)
**Target Release:** Q1 2026
**Owner:** Campaigns Team
