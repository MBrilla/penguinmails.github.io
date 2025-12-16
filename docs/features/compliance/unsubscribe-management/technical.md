---
title: "Technical Implementation"
description: "API endpoints, database schema, webhooks, and best practices"
last_modified_date: "2025-01-15"
level: "3"
persona: "Developers, Technical Teams"
status: "ACTIVE"
category: "Compliance"
---

# Technical Implementation

API endpoints, database schema, webhook events, and unsubscribe best practices.

## Unsubscribe API Endpoints

### Get Suppression List

```javascript
GET /api/v1/suppression-list
Authorization: Bearer {api_key}

Response:
{
  "total": 1523,
  "page": 1,
  "per_page": 100,
  "data": [
    {
      "email": "user@example.com",
      "added_at": "2025-11-24T10:30:00Z",
      "reason": "user_request",
      "source": "one_click_unsubscribe"
    }
  ]
}
```

### Add to Suppression List

```javascript
POST /api/v1/suppression-list
Authorization: Bearer {api_key}
Content-Type: application/json

{
  "email": "user@example.com",
  "reason": "user_request",
  "source": "api"
}
```

### Check if Email Suppressed

```javascript
GET /api/v1/suppression-list/check?email=user@example.com
Authorization: Bearer {api_key}

Response:
{
  "suppressed": true,
  "reason": "user_request",
  "added_at": "2025-11-24T10:30:00Z"
}
```

---

## Database Schema

```sql
-- Suppression List Table
CREATE TABLE suppression_list (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID REFERENCES tenants(id),
  workspace_id UUID REFERENCES workspaces(id),
  email VARCHAR(255) NOT NULL,
  reason VARCHAR(50) NOT NULL,
  source VARCHAR(50),
  added_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  added_by UUID REFERENCES users(id),
  scope VARCHAR(20) DEFAULT 'global', -- global, tenant, workspace, campaign_type
  expires_at TIMESTAMP NULL, -- NULL = never expires

  UNIQUE(tenant_id, email, scope)
);

CREATE INDEX idx_suppression_email ON suppression_list(email);
CREATE INDEX idx_suppression_tenant ON suppression_list(tenant_id);
CREATE INDEX idx_suppression_added_at ON suppression_list(added_at);
```

---

## Webhook Events

**Real-time notifications for suppression events:**

### Unsubscribe Event

```javascript
{
  "event": "contact.unsubscribed",
  "timestamp": "2025-11-24T10:30:00Z",
  "data": {
    "email": "user@example.com",
    "campaign_id": "camp_abc123",
    "reason": "user_request",
    "method": "one_click"
  }
}
```

### Spam Complaint Event

```javascript
{
  "event": "contact.spam_complaint",
  "timestamp": "2025-11-24T10:30:00Z",
  "data": {
    "email": "user@example.com",
    "campaign_id": "camp_abc123",
    "isp": "gmail",
    "action_taken": "added_to_suppression"
  }
}
```

---

## Best Practices

### Unsubscribe Best Practices

1. **Make it Easy** - One-click, no login required
2. **Make it Visible** - Clear footer placement
3. **Process Immediately** - Don't wait the full 10 days
4. **Offer Alternatives** - Preference center before full unsubscribe
5. **Respect Choices** - Never re-add without explicit consent
6. **Monitor Rates** - Track unsubscribe and complaint rates
7. **Use Feedback** - Understand why users unsubscribe
8. **Keep Forever** - Never delete suppression records

### Reducing Unsubscribes

- **Send Relevant Content** - Segment your audience
- **Control Frequency** - Don't over-email
- **Provide Value** - Quality over quantity
- **Set Expectations** - Deliver what you promised
- **Make it Personal** - Use personalization
- **Test and Optimize** - A/B test content and timing

---

## Reporting & Analytics

### Key Metrics

| Metric | Description |
|--------|-------------|
| Unsubscribe Rate | % of recipients who unsubscribe per campaign |
| Spam Report Rate | % of recipients who report spam |
| Suppression List Growth | Rate of suppression list additions |
| Bounce Rate | Hard/soft bounce percentage |
| Re-engagement Rate | Users who return after unsubscribing |

### Benchmarks

| Status | Unsubscribe Rate |
|--------|------------------|
| Good | < 0.2% |
| Concerning | 0.2% - 0.5% |
| Poor | > 0.5% |

---

**Related Documents:**
- [Overview](overview.md)
- [Core Features](core-features.md)
- [Advanced Processing](advanced-processing.md)
- [Campaign Management](/docs/features/campaigns/campaign-management/hub)
- [Analytics](/docs/features/analytics/core-analytics/overview)
