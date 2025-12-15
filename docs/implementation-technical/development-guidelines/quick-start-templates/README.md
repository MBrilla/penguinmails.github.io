---
title: "Developer Quick Start Templates"
description: "Technology-specific templates for PenguinMails integration"
last_modified_date: "2025-12-15"
level: "2"
persona: "Developers"
---

# Developer Quick Start Templates

Production-ready code templates for integrating PenguinMails into your applications.

## Overview

These templates provide ready-to-use code examples for common integration scenarios across different technologies and frameworks.

## Topic Guides

| Guide | Technology | Description |
|-------|------------|-------------|
| [JavaScript Templates](./javascript-templates) | JS/TS, Node.js, Next.js | SDK integration, Express, React |
| [Email Templates](./email-templates) | CSS, HTML | Responsive email styles and layouts |
| [Integration Patterns](./integration-patterns) | Multi-language | Common integration patterns |

## Quick Start

### Basic SDK Usage

```javascript
import { PenguinMails } from '@penguinmails/sdk';

const client = new PenguinMails({
  apiKey: process.env.PENGUINMAILS_API_KEY,
  environment: 'production'
});

// Create a campaign
const campaign = await client.campaigns.create({
  name: "Welcome Campaign",
  subject: "Welcome to PenguinMails!",
  templateId: "welcome-template",
  audience: { segmentId: "new_subscribers" },
  schedule: { type: "immediate" }
});
```

### Express Integration

```javascript
const express = require('express');
const { PenguinMails } = require('@penguinmails/sdk');

const app = express();
const client = new PenguinMails({ apiKey: process.env.PENGUINMAILS_API_KEY });

app.post('/api/send-notification', async (req, res) => {
  const { email, message } = req.body;
  const result = await client.campaigns.sendImmediate({
    templateId: 'notification-template',
    recipient: email,
    variables: { message }
  });
  res.json({ success: true, campaignId: result.id });
});
```

---

## Related Documentation

- [API Reference](/docs/implementation-technical/api/platform-api/overview)
- [Development Guidelines](/docs/implementation-technical/development-guidelines/README)
