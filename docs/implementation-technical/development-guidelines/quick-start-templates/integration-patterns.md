---
title: "Integration Patterns"
last_modified_date: "2025-12-15"
level: "2"
persona: "Developers"
---

# Integration Patterns

Common integration patterns for PenguinMails across different use cases.

## Webhook Integration

```javascript
const express = require('express');
const crypto = require('crypto');

const app = express();
app.use(express.raw({ type: 'application/json' }));

// Webhook endpoint for PenguinMails events
app.post('/webhooks/penguinmails', (req, res) => {
  // Verify webhook signature
  const signature = req.headers['x-penguinmails-signature'];
  const expectedSignature = crypto
    .createHmac('sha256', process.env.WEBHOOK_SECRET)
    .update(req.body)
    .digest('hex');

  if (signature !== expectedSignature) {
    return res.status(401).json({ error: 'Invalid signature' });
  }

  const event = JSON.parse(req.body);

  switch (event.type) {
    case 'email.delivered':
      handleDelivery(event.data);
      break;
    case 'email.opened':
      handleOpen(event.data);
      break;
    case 'email.clicked':
      handleClick(event.data);
      break;
    case 'email.bounced':
      handleBounce(event.data);
      break;
    case 'contact.unsubscribed':
      handleUnsubscribe(event.data);
      break;
  }

  res.status(200).json({ received: true });
});
```

## Batch Processing

```javascript
import { PenguinMails } from '@penguinmails/sdk';

const client = new PenguinMails({ apiKey: process.env.PENGUINMAILS_API_KEY });

async function processBatchContacts(contacts) {
  const BATCH_SIZE = 100;
  const results = [];

  for (let i = 0; i < contacts.length; i += BATCH_SIZE) {
    const batch = contacts.slice(i, i + BATCH_SIZE);
    
    const batchResult = await client.contacts.bulkCreate({
      contacts: batch,
      updateExisting: true
    });
    
    results.push(...batchResult.results);
    
    // Rate limiting - wait between batches
    if (i + BATCH_SIZE < contacts.length) {
      await new Promise(resolve => setTimeout(resolve, 1000));
    }
  }

  return results;
}
```

## Error Handling Pattern

```javascript
import { PenguinMails, PenguinMailsError } from '@penguinmails/sdk';

async function sendEmailWithRetry(emailData, maxRetries = 3) {
  const client = new PenguinMails({ apiKey: process.env.PENGUINMAILS_API_KEY });

  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      const result = await client.campaigns.sendImmediate(emailData);
      return result;
    } catch (error) {
      if (error instanceof PenguinMailsError) {
        // Don't retry client errors
        if (error.statusCode >= 400 && error.statusCode < 500) {
          throw error;
        }
        
        // Retry server errors with exponential backoff
        if (attempt < maxRetries) {
          const delay = Math.pow(2, attempt) * 1000;
          await new Promise(resolve => setTimeout(resolve, delay));
          continue;
        }
      }
      throw error;
    }
  }
}
```

## Queue-Based Processing

```javascript
import { Queue, Worker } from 'bullmq';
import { PenguinMails } from '@penguinmails/sdk';

// Create queue for email jobs
const emailQueue = new Queue('email-sending', {
  connection: { host: 'localhost', port: 6379 }
});

// Add email to queue
async function queueEmail(emailData) {
  await emailQueue.add('send-email', emailData, {
    attempts: 3,
    backoff: { type: 'exponential', delay: 2000 }
  });
}

// Process email queue
const worker = new Worker('email-sending', async (job) => {
  const client = new PenguinMails({ apiKey: process.env.PENGUINMAILS_API_KEY });
  return await client.campaigns.sendImmediate(job.data);
}, {
  connection: { host: 'localhost', port: 6379 },
  concurrency: 5
});
```

---

[← Back to Quick Start Templates](./README)
