---
title: "API Documentation and Usage Analytics Routes"
description: "Route specifications for interactive API docs and usage monitoring"
last_modified_date: "2025-11-25"
level: "3"
persona: "Developers"
---

# API Documentation and Usage Analytics Routes

## `/dashboard/settings/developers/docs` - Interactive API Documentation

**User Story**: *"As a developer, I want to explore API endpoints with live examples, so I can quickly understand how to integrate."*

### API Documentation Browser

**Sidebar Navigation**:

- **Getting Started**
  - Authentication
  - Rate Limiting
  - Error Handling
- **Endpoints**
  - Emails
  - Campaigns
  - Contacts
  - Analytics
  - Webhooks
- **SDKs and Libraries**
- **Changelog**

### Endpoint Documentation

#### Example: Send Email Endpoint

**Endpoint Header**:

```http
POST /v1/emails/send
```

**Authentication**: Required (Bearer Token)
**Rate Limit**: 60 requests/minute

**Description**: Send a single transactional or marketing email. Supports templates, variables, and tracking.

**Request Parameters**:

**Headers**:

```http
Authorization: Bearer pm_live_...
Content-Type: application/json
```

**Body** (JSON):

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `to` | string | Yes | Recipient email address |
| `from` | string | No | Sender email (defaults to account default) |
| `subject` | string | Yes* | Email subject (*required if not using template) |
| `html_body` | string | Yes* | HTML email body (*required if not using template) |
| `text_body` | string | No | Plain text version |
| `template_id` | string | No | Template ID (alternative to subject/body) |
| `variables` | object | No | Template variables |
| `track_opens` | boolean | No | Enable open tracking (default: true) |
| `track_clicks` | boolean | No | Enable click tracking (default: true) |
| `tags` | array | No | Tags for organization |

**Example Request**:

```json
{
  "to": "user@example.com",
  "template_id": "welcome-v1",
  "variables": {
    "name": "Alice",
    "company": "Acme Corp"
  },
  "tags": ["onboarding", "welcome"]
}
```

**Response** (200 OK):

```json
{
  "id": "msg_abc123",
  "status": "queued",
  "to": "user@example.com",
  "created_at": "2025-11-25T10:30:00Z"
}
```

**Error Responses**:

**401 Unauthorized**:

```json
{
  "error": "invalid_token",
  "message": "API key is invalid or expired"
}
```

**429 Too Many Requests**:

```json
{
  "error": "rate_limit_exceeded",
  "message": "Rate limit exceeded. Try again in 42 seconds.",
  "retry_after": 42
}
```

### Try It Out

**Interactive API Tester**:

- **API Key**: Dropdown (select from your keys) or input field
- **Request Body**: JSON editor with syntax highlighting
- **"Send Request" Button**: Makes live API call
- **Response Display**: Shows status code, headers, and body

### Code Examples

**Language Tabs**: cURL, Node.js, Python, PHP, Ruby.

**cURL**:

```bash
curl -X POST https://api.penguinmails.com/v1/emails/send \
  -H "Authorization: Bearer pm_live_..." \
  -H "Content-Type: application/json" \
  -d '{
    "to": "user@example.com",
    "subject": "Hello",
    "html_body": "<h1>Hello World</h1>"
  }'
```

**Node.js** (Coming Soon):

```javascript
const penguinmails = require('@penguinmails/sdk');

const client = new penguinmails.Client('pm_live_...');

await client.emails.send({
  to: 'user@example.com',
  subject: 'Hello',
  html_body: '<h1>Hello World</h1>'
});
```

**Copy Button**: Copy code snippet to clipboard.

---

## `/dashboard/settings/developers/usage` - API Usage Analytics

**User Story**: *"As a developer, I want to monitor API usage and errors, so I can optimize my integration and troubleshoot issues."*

### Usage Overview

**Time Range Selector**:
- Buttons: Last Hour, Last 24 Hours, Last 7 Days, Last 30 Days, Custom Range

**Metrics Cards** (Top Row):
- **Total Requests**: "12,345"
- **Success Rate**: "99.5%"
- **Average Response Time**: "145ms"
- **Rate Limit Usage**: "15% of limit"

### Request Volume Chart

**Line Chart**: Requests over time.
- **Y-Axis**: Number of requests
- **X-Axis**: Time
- **Lines**: Total, Success (200), Errors (4xx/5xx)
- **Hover**: Shows exact values at each point

### Endpoint Usage

**Top Endpoints Table**:

| Endpoint | Requests | Success Rate | Avg Response Time | Errors |
|----------|----------|--------------|-------------------|--------|
| `POST /v1/emails/send` | 8,234 | 99.8% | 120ms | 16 |
| `GET /v1/campaigns` | 2,456 | 100% | 85ms | 0 |
| `POST /v1/contacts` | 1,234 | 98.5% | 95ms | 18 |
| `GET /v1/analytics/campaigns/{id}` | 421 | 100% | 210ms | 0 |

**"View All Endpoints" Link**: Expands full list.

### Error Analysis

**Error Breakdown** (Pie Chart):
- 400 Bad Request: 45%
- 401 Unauthorized: 30%
- 429 Rate Limit: 15%
- 500 Server Error: 10%

**Recent Errors Table**:

| Timestamp | Endpoint | Status | Error | Details |
|-----------|----------|--------|-------|---------|
| 2:34 PM | `POST /v1/emails/send` | 400 | `invalid_email` | "Recipient email is invalid" |
| 2:30 PM | `GET /v1/campaigns` | 401 | `invalid_token` | "API key expired" |
| 2:25 PM | `POST /v1/contacts` | 429 | `rate_limit` | "Rate limit exceeded" |

**"Export Error Log" Button**: Download CSV of errors.

### Rate Limit Monitoring

**Rate Limit Usage Chart** (Line Chart):
- Shows rate limit usage over time
- **Warning Line**: 80% of limit (yellow)
- **Critical Line**: 95% of limit (red)

**Rate Limit Events**:
- "Rate limit exceeded 3 times in the last 24 hours"
- "Peak usage: 285 / 300 requests/minute at 2:15 PM"

**Recommendation**:
- "Consider implementing request batching or upgrading your plan."

### API Key Breakdown

**Usage by API Key** (Table):

| Key Name | Requests | Success Rate | Last Used |
|----------|----------|--------------|-----------|
| Production App | 10,234 | 99.8% | 2 hours ago |
| Staging Server | 2,111 | 98.2% | 5 hours ago |

**Filter**: Select specific API key to view its usage.

---

## `/dashboard/settings/developers/webhooks` - Webhook Configuration

**User Story**: *"As a developer, I want to configure webhooks to receive real-time event notifications."*

**Redirect Notice**: "Webhook configuration has moved to [ESP Integration Settings →](/dashboard/settings/integrations/esp/webhooks)"

**Quick Links**:
- [Configure Postmark Webhooks →](/dashboard/settings/integrations/esp/webhooks#postmark)
- [Configure Mailgun Webhooks →](/dashboard/settings/integrations/esp/webhooks#mailgun)
- [View Webhook Documentation →](/docs/features/integrations/webhook-system)

## Related Documentation

- [Full API Reference](/docs/implementation-technical/api/README)
- [Authentication Guide](/docs/features/integrations/api-access#authentication)
- [Rate Limiting](/docs/features/integrations/api-access#rate-limiting)
- [Error Handling](/docs/implementation-technical/api/README#error-handling)
