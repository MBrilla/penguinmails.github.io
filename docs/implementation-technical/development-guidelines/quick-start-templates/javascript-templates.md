---
title: "JavaScript Templates"
last_modified_date: "2025-12-15"
level: "2"
persona: "Developers"
---

# JavaScript Templates

SDK integration, Express, and React/Next.js templates for PenguinMails.

## Basic Email Campaign Script

```javascript
import { PenguinMails } from '@penguinmails/sdk';

async function createBasicCampaign() {
  const client = new PenguinMails({
    apiKey: "YOUR_API_KEY",
    environment: process.env.NODE_ENV === 'production' ? 'production' : 'sandbox'
  });

  const campaign = {
    name: "Welcome Campaign",
    subject: "Welcome to PenguinMails!",
    templateId: "welcome-template",
    audience: {
      segmentId: "new_subscribers"
    },
    schedule: {
      type: "immediate"
    }
  };

  const result = await client.campaigns.create(campaign);
  return result;
}

const campaign = await createBasicCampaign();
console.log(`Campaign created: ${campaign.id}`);
```

## Node.js Express Integration

```javascript
const express = require('express');
const { PenguinMails } = require('@penguinmails/sdk');

const app = express();
app.use(express.json());

const client = new PenguinMails({
  apiKey: process.env.PENGUINMAILS_API_KEY
});

// Send notification email
app.post('/api/send-notification', async (req, res) => {
  const { email, message, type } = req.body;

  try {
    const result = await client.campaigns.sendImmediate({
      templateId: 'notification-template',
      recipient: email,
      variables: {
        message,
        type,
        timestamp: new Date().toISOString()
      }
    });

    res.json({ success: true, campaignId: result.id });
  } catch (error) {
    res.status(400).json({ success: false, error: error.message });
  }
});

// Campaign creation endpoint
app.post('/api/create-campaign', async (req, res) => {
  const { name, subject, templateId, segmentId } = req.body;

  try {
    const campaign = await client.campaigns.create({
      name,
      subject,
      templateId,
      audience: { segmentId },
      schedule: {
        type: 'scheduled',
        time: new Date(Date.now() + 24 * 60 * 60 * 1000)
      }
    });

    res.json({ success: true, campaign });
  } catch (error) {
    res.status(400).json({ success: false, error: error.message });
  }
});

app.listen(3000, () => {
  console.log('PenguinMails integration server running on port 3000');
});
```

## Next.js Component Example

```jsx
'use client';

import { useState } from 'react';

export default function EmailCampaignManager() {
  const [campaign, setCampaign] = useState({
    name: '',
    subject: '',
    templateId: '',
    scheduledDate: ''
  });
  const [isLoading, setIsLoading] = useState(false);

  const handleSubmit = async (e) => {
    e.preventDefault();
    setIsLoading(true);

    try {
      const response = await fetch('/api/create-campaign', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(campaign)
      });

      const result = await response.json();
      if (result.success) {
        alert('Campaign created successfully!');
      } else {
        alert('Error creating campaign: ' + result.error);
      }
    } catch (error) {
      alert('Error: ' + error.message);
    }

    setIsLoading(false);
  };

  return (
    <div className="max-w-md mx-auto p-6">
      <h1 className="text-2xl font-bold mb-4">Create Email Campaign</h1>
      <form onSubmit={handleSubmit} className="space-y-4">
        <div>
          <label className="block text-sm font-medium mb-1">Campaign Name</label>
          <input
            type="text"
            value={campaign.name}
            onChange={(e) => setCampaign({...campaign, name: e.target.value})}
            className="w-full border rounded px-3 py-2"
            required
          />
        </div>
        <div>
          <label className="block text-sm font-medium mb-1">Subject</label>
          <input
            type="text"
            value={campaign.subject}
            onChange={(e) => setCampaign({...campaign, subject: e.target.value})}
            className="w-full border rounded px-3 py-2"
            required
          />
        </div>
        <button
          type="submit"
          disabled={isLoading}
          className="w-full bg-blue-500 text-white py-2 rounded hover:bg-blue-600"
        >
          {isLoading ? 'Creating...' : 'Create Campaign'}
        </button>
      </form>
    </div>
  );
}
```

---

[← Back to Quick Start Templates](./README)
