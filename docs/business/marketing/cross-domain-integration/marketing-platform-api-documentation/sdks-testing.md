---
title: SDKs and Testing
description: Client libraries, sandbox environment, and support resources
last_modified_date: 2025-01-15
level: 3
persona: ["developer"]
keywords: [SDK, testing, sandbox, support, JavaScript]
---

# SDKs and Testing

Client libraries, sandbox environment, and developer support resources.

## JavaScript/TypeScript SDK

### Installation

```bash
npm install @marketing-platform/sdk
```

### Usage

```javascript
import { MarketingPlatform } from '@marketing-platform/sdk';

const client = new MarketingPlatform({
  apiKey: 'pk_live_your_api_key',
  environment: 'production'
});

// Create campaign
const campaign = await client.campaigns.create({
  name: 'Q1 Launch Campaign',
  type: 'email_marketing',
  targetAudience: {
    segments: ['enterprise_customers']
  }
});

// Track conversion
await client.analytics.trackConversion({
  campaignId: campaign.id,
  recipientId: 'recipient_123',
  revenue: 299.99
});
```

## Email Template Styles

```css
/* Marketing Platform Email Template Styles */
.email-template {
  max-width: 600px;
  margin: 0 auto;
  font-family: 'Helvetica Neue', Arial, sans-serif;
}

.header {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  padding: 30px 20px;
  text-align: center;
}

.content {
  padding: 30px 20px;
  background: #ffffff;
}

.cta-button {
  display: inline-block;
  background: #4CAF50;
  color: white;
  padding: 12px 24px;
  text-decoration: none;
  border-radius: 5px;
  font-weight: bold;
}
```

## Sandbox Environment

| Property | Value |
|----------|-------|
| Base URL | `https://api-sandbox.marketingplatform.com` |
| Purpose | Testing and development |
| Email Delivery | Simulated (no actual delivery) |
| Rate Limits | Limited for testing |
| Data | Resettable test data |

## Testing Tools

### Postman Collection

- Complete API endpoint collection
- Pre-configured authentication
- Example requests and responses
- Environment variables for easy testing

### OpenAPI Specification

```yaml
openapi: 3.0.3
info:
  title: Marketing Platform API
  version: 1.0.0
  description: Comprehensive API for marketing platform integrations
servers:
  - url: https://api.marketingplatform.com/v1
    description: Production server
  - url: https://api-sandbox.marketingplatform.com/v1
    description: Sandbox server
```

## Monitoring and Alerts

### Performance Metrics

- Response time monitoring
- Error rate tracking
- Success rate analytics
- Usage pattern analysis

### Alert Configuration

```json
{
  "alert_conditions": {
    "response_time_p95": ">500ms",
    "error_rate": ">5%",
    "rate_limit_usage": ">90%",
    "integration_failures": ">3_in_5_minutes"
  },
  "notification_channels": [
    "email: api-team@company.com",
    "slack: #api-alerts",
    "webhook: https://alerts.company.com/api"
  ]
}
```

## Support Resources

| Resource | Location |
|----------|----------|
| Developer Portal | Interactive docs and tutorials |
| Email Support | api-support@marketingplatform.com |
| Slack | #api-developer-support |
| Status Page | [status.marketingplatform.com](https://status.marketingplatform.com) |

## Related Documentation

- [API Overview](./)
- [Authentication](./authentication)
- [Rate Limits & Errors](./rate-limits-errors)
