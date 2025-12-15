---
title: "API Key Created Success Modal"
description: Success modal after API key creation with code examples
last_modified_date: 2025-01-15
level: 3
persona: ["frontend-developer"]
keywords: [modal, API-keys, success, code-examples, copy]
---

# API Key Created Success Modal

**Trigger:** After successful API key creation
**Modal Type:** Centered modal with backdrop (cannot close by clicking backdrop)

## Modal Structure

```text
┌─────────────────────────────────────────────────┐
│ ⚠️ API Key Created                        [X]   │
├─────────────────────────────────────────────────┤
│                                                 │
│ Store this key securely. It will not be shown  │
│ again. If you lose it, you'll need to          │
│ regenerate a new key.                           │
│                                                 │
│ Your API Key                                    │
│ ┌─────────────────────────────────────────────┐│
│ │ pm_live_a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6  📋││
│ └─────────────────────────────────────────────┘│
│                                                 │
│ [Copy Key]  [Download .env]                    │
│                                                 │
│ Quick Start                                     │
│ ┌─────────────────────────────────────────────┐│
│ │ cURL | Node.js | Python | PHP              ▼││
│ │ curl -X POST https://api.penguinmails.com  ││
│ │   -H "Authorization: Bearer pm_live_..."   ││
│ └─────────────────────────────────────────────┘│
│                                                 │
│ [View Documentation]              [Done]       │
└─────────────────────────────────────────────────┘
```

## UI Components

### Warning Banner

- **Icon**: ⚠️ Warning icon
- **Text**: "Store this key securely. It will not be shown again."
- **Style**: Yellow background, prominent

### API Key Display

- Full API key value (plaintext)
- Copy button (clipboard icon)
- Monospace font
- Select-all on click

### Action Buttons

| Button | Action |
|--------|--------|
| Copy Key | Copies API key to clipboard, shows toast |
| Download .env | Downloads file with `PENGUINMAILS_API_KEY=pm_live_...` |

### Code Examples

Tabbed interface with syntax highlighting:

**cURL:**

```bash
curl -X POST https://api.penguinmails.com/api/v1/emails/send \
  -H "Authorization: Bearer pm_live_a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6" \
  -H "Content-Type: application/json" \
  -d '{
    "to": "recipient@example.com",
    "subject": "Test Email",
    "body": "Hello from PenguinMails API"
  }'
```

**Node.js:**

```javascript
const axios = require('axios');

const apiKey = 'pm_live_a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6';

axios.post('https://api.penguinmails.com/api/v1/emails/send', {
  to: 'recipient@example.com',
  subject: 'Test Email',
  body: 'Hello from PenguinMails API'
}, {
  headers: {
    'Authorization': `Bearer ${apiKey}`,
    'Content-Type': 'application/json'
  }
}).then(response => {
  console.log('Email sent:', response.data);
}).catch(error => {
  console.error('Error:', error.response.data);
});
```

### Footer Actions

| Button | Action |
|--------|--------|
| View Documentation | Opens API docs in new tab |
| Done | Closes modal, returns to API key list |

## Security Notes

- Disable backdrop click to close
- Require explicit "Done" button click
- Show confirmation if user tries to close without copying
