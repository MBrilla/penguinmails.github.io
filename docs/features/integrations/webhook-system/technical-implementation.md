---
title: Webhook Technical Implementation
description: Database schema, delivery service, and API endpoints
last_modified_date: 2025-01-15
level: 3
persona: ["developer"]
keywords: [webhook, database, schema, delivery, API]
---

{% raw %}

# Webhook Technical Implementation

Database schema, delivery service architecture, and API endpoints.

## Database Schema

```sql
-- Webhook endpoints
CREATE TABLE webhooks (
  id UUID PRIMARY KEY,
  tenant_id UUID NOT NULL REFERENCES tenants(id),

  -- Configuration
  name VARCHAR(255) NOT NULL,
  description TEXT,
  endpoint_url TEXT NOT NULL,
  secret_key VARCHAR(255) NOT NULL,

  -- Event filtering
  event_types TEXT[],
  event_filters JSONB,

  -- Status
  is_active BOOLEAN DEFAULT TRUE,
  is_failing BOOLEAN DEFAULT FALSE,
  consecutive_failures INTEGER DEFAULT 0,
  last_success_at TIMESTAMP,
  last_failure_at TIMESTAMP,

  -- Metadata
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Webhook delivery attempts
CREATE TABLE webhook_deliveries (
  id UUID PRIMARY KEY,
  webhook_id UUID NOT NULL REFERENCES webhooks(id),
  event_id UUID NOT NULL,

  -- Request
  event_type VARCHAR(100),
  payload JSONB,
  signature VARCHAR(255),

  -- Response
  status_code INTEGER,
  response_body TEXT,
  response_headers JSONB,

  -- Timing
  duration_ms INTEGER,
  attempted_at TIMESTAMP DEFAULT NOW(),

  -- Retry tracking
  attempt_number INTEGER DEFAULT 1,
  next_retry_at TIMESTAMP,

  -- Status
  status VARCHAR(50),
  error_message TEXT
);

CREATE INDEX idx_webhook_deliveries_webhook ON webhook_deliveries(webhook_id, attempted_at);
CREATE INDEX idx_webhook_deliveries_status ON webhook_deliveries(status, next_retry_at);

-- Events log (for replay)
CREATE TABLE webhook_events (
  id UUID PRIMARY KEY,
  tenant_id UUID NOT NULL,
  event_type VARCHAR(100),
  event_data JSONB,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_webhook_events_tenant_type ON webhook_events(tenant_id, event_type, created_at);
```

## Webhook Delivery Service

```typescript
class WebhookDeliveryService {
  async deliverEvent(event: WebhookEvent): Promise<void> {
    const webhooks = await this.findMatchingWebhooks(event);

    for (const webhook of webhooks) {
      await this.queueDelivery(webhook, event);
    }
  }

  private async findMatchingWebhooks(event: WebhookEvent): Promise<Webhook[]> {
    return db.webhooks.findAll({
      where: {
        tenantId: event.tenantId,
        isActive: true,
        eventTypes: { contains: [event.type] },
      },
    });
  }

  private async queueDelivery(webhook: Webhook, event: WebhookEvent): Promise<void> {
    await webhookQueue.add('deliver-webhook', {
      webhookId: webhook.id,
      eventId: event.id,
      payload: event.data,
    }, {
      attempts: 5,
      backoff: { type: 'exponential', delay: 60000 },
    });
  }

  async deliver(webhookId: string, event: WebhookEvent): Promise<void> {
    const webhook = await db.webhooks.findById(webhookId);
    const payload = JSON.stringify(event);
    const signature = this.generateSignature(payload, webhook.secretKey);

    const startTime = Date.now();

    try {
      const response = await axios.post(webhook.endpointUrl, payload, {
        headers: {
          'Content-Type': 'application/json',
          'X-PenguinMails-Signature': signature,
          'X-PenguinMails-Event': event.type,
          'X-PenguinMails-Delivery-ID': uuidv4(),
        },
        timeout: 5000,
      });

      const duration = Date.now() - startTime;

      await db.webhookDeliveries.create({
        webhookId,
        eventId: event.id,
        eventType: event.type,
        payload: event.data,
        signature,
        statusCode: response.status,
        responseBody: response.data,
        durationMs: duration,
        status: 'success',
      });

      await db.webhooks.update(webhookId, {
        consecutiveFailures: 0,
        isFailing: false,
        lastSuccessAt: new Date(),
      });

    } catch (error) {
      await this.handleDeliveryFailure(webhook, event, error);
    }
  }

  private generateSignature(payload: string, secret: string): string {
    const hmac = crypto.createHmac('sha256', secret);
    hmac.update(payload);
    return `sha256=${hmac.digest('hex')}`;
  }

  private async handleDeliveryFailure(
    webhook: Webhook,
    event: WebhookEvent,
    error: any
  ): Promise<void> {
    const failures = webhook.consecutiveFailures + 1;

    await db.webhookDeliveries.create({
      webhookId: webhook.id,
      eventId: event.id,
      status: 'failed',
      errorMessage: error.message,
    });

    await db.webhooks.update(webhook.id, {
      consecutiveFailures: failures,
      isFailing: failures >= 3,
      lastFailureAt: new Date(),
    });

    if (failures >= 100) {
      await db.webhooks.update(webhook.id, { isActive: false });
      await notificationService.send({
        tenantId: webhook.tenantId,
        type: 'webhook_paused',
        data: { webhookId: webhook.id, failures },
      });
    }
  }
}
```

## Event Emission

```typescript
async function handleEmailOpen(emailId: string): Promise<void> {
  const email = await db.emails.findById(emailId);
  const contact = await db.contacts.findById(email.contactId);
  const campaign = await db.campaigns.findById(email.campaignId);

  const event: WebhookEvent = {
    id: uuidv4(),
    type: 'email.opened',
    tenantId: campaign.tenantId,
    createdAt: new Date(),
    data: {
      emailId: email.id,
      campaignId: campaign.id,
      contact: {
        id: contact.id,
        email: contact.email,
        firstName: contact.firstName,
        lastName: contact.lastName,
      },
      openedAt: new Date(),
      userAgent: req.headers['user-agent'],
      ipAddress: req.ip,
    },
  };

  await db.webhookEvents.create(event);
  await webhookDeliveryService.deliverEvent(event);
}
```

## API Endpoints

```typescript
// Create webhook
app.post('/api/webhooks', authenticate, async (req, res) => {
  const { name, endpointUrl, eventTypes, eventFilters } = req.body;
  const secretKey = `whsec_${randomBytes(32).toString('hex')}`;

  const webhook = await db.webhooks.create({
    tenantId: req.user.tenantId,
    name,
    endpointUrl,
    secretKey,
    eventTypes,
    eventFilters,
    isActive: true,
  });

  return res.json({
    id: webhook.id,
    name: webhook.name,
    endpointUrl: webhook.endpointUrl,
    secretKey,
    eventTypes: webhook.eventTypes,
  });
});

// Test webhook
app.post('/api/webhooks/:id/test', authenticate, async (req, res) => {
  const webhook = await db.webhooks.findById(req.params.id);

  const testEvent = {
    id: `evt_test_${Date.now()}`,
    type: 'email.opened',
    created_at: new Date().toISOString(),
    data: {
      email_id: 'msg_sample',
      contact: { email: 'test@example.com', first_name: 'Test' },
    },
  };

  try {
    await webhookDeliveryService.deliver(webhook.id, testEvent);
    return res.json({ success: true });
  } catch (error) {
    return res.status(400).json({ error: error.message });
  }
});
```

## Related Documentation

- [Webhook Overview](./)
- [Advanced Configuration](./advanced-configuration)
- [Event Reference](./event-reference)

{% endraw %}
