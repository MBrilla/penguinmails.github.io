---
title: Integration APIs
description: CRM, Analytics, E-commerce, and Webhook integration endpoints
last_modified_date: 2025-01-15
level: 3
persona: ["developer"]
keywords: [API, CRM, Salesforce, HubSpot, Shopify, webhooks]
---

# Integration APIs

CRM, Analytics, E-commerce, and Webhook integration endpoints.

## CRM Integration API

### Salesforce Integration

```http
POST /api/v1/integrations/crm/salesforce/sync
```

**Request Body:**

```json
{
  "sync_type": "bi_directional",
  "objects": ["lead", "contact", "opportunity", "campaign"],
  "field_mappings": {
    "lead": {
      "email": "Email",
      "company": "Company",
      "lead_score": "Custom_Score__c"
    }
  },
  "sync_frequency": "real_time",
  "conflict_resolution": "last_write_wins"
}
```

### HubSpot Integration

```http
POST /api/v1/integrations/crm/hubspot/contacts
```

**Request Body:**

```json
{
  "action": "create_or_update",
  "contacts": [
    {
      "properties": {
        "email": "john.doe@company.com",
        "firstname": "John",
        "lastname": "Doe",
        "company": "Tech Corp",
        "hs_lead_status": "QUALIFIED",
        "marketing_platform_lead_score": 85
      }
    }
  ]
}
```

## Analytics Integration API

### Google Analytics

```http
POST /api/v1/integrations/analytics/google/track
```

**Request Body:**

```json
{
  "event_category": "email_campaign",
  "event_action": "email_open",
  "event_label": "q1_product_launch",
  "custom_parameters": {
    "campaign_id": "camp_789xyz",
    "recipient_id": "recipient_123",
    "open_timestamp": "2025-01-15T10:30:00Z"
  }
}
```

### Customer Data Platform

```http
POST /api/v1/integrations/cdp/audience/update
```

**Request Body:**

```json
{
  "audience_id": "aud_enterprise_customers",
  "operation": "add_members",
  "members": [
    {
      "external_id": "contact_456",
      "attributes": {
        "company_size": 150,
        "industry": "technology",
        "engagement_score": 85
      }
    }
  ]
}
```

## E-commerce Integration API

### Shopify Integration

```http
POST /api/v1/integrations/ecommerce/shopify/customer-event
```

**Request Body:**

```json
{
  "customer_id": "shopify_customer_789",
  "event_type": "purchase",
  "event_data": {
    "order_id": "order_123456",
    "total_price": 299.99,
    "line_items": [
      {
        "product_id": "product_789",
        "quantity": 1,
        "price": 299.99,
        "title": "Premium Marketing Suite"
      }
    ],
    "marketing_attribution": {
      "campaign_id": "camp_789xyz",
      "utm_source": "email"
    }
  }
}
```

### WooCommerce Integration

```http
POST /api/v1/integrations/ecommerce/woocommerce/order-update
```

**Request Body:**

```json
{
  "order_id": 12345,
  "customer_email": "customer@example.com",
  "order_status": "completed",
  "total_amount": 199.99,
  "marketing_source": {
    "campaign_id": "camp_789xyz",
    "referrer": "email_campaign"
  }
}
```

## Webhook Configuration

### Register Webhook

```http
POST /api/v1/webhooks
```

**Request Body:**

```json
{
  "url": "https://your-crm.com/webhooks/marketing-platform",
  "events": [
    "campaign.sent",
    "email.opened",
    "lead.created",
    "lead.qualified",
    "conversion.completed"
  ],
  "secret": "webhook_secret_key",
  "active": true
}
```

### Webhook Payload Example

```json
{
  "event": "lead.created",
  "timestamp": "2025-01-15T10:30:00Z",
  "data": {
    "lead_id": "lead_456abc",
    "source": "website_form",
    "contact": {
      "email": "john.doe@company.com",
      "first_name": "John"
    },
    "lead_score": 85
  },
  "signature": "sha256=abc123..."
}
```

### Signature Verification

```javascript
const crypto = require('crypto');

function verifyWebhookSignature(payload, signature, secret) {
  const expectedSignature = crypto
    .createHmac('sha256', secret)
    .update(payload)
    .digest('hex');

  return signature === `sha256=${expectedSignature}`;
}
```

## Related Documentation

- [API Overview](./)
- [Core APIs](./core-apis)
- [Rate Limits & Errors](./rate-limits-errors)
