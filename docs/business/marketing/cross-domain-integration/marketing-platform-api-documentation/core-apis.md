---
title: Core Marketing APIs
description: Campaign, Lead, and Analytics API endpoints
last_modified_date: 2025-01-15
level: 3
persona: ["developer"]
keywords: [API, campaign, lead, analytics, endpoints]
---

# Core Marketing APIs

Campaign management, lead capture, and analytics API endpoints.

## Campaign Management API

### Create Campaign

```http
POST /api/v1/campaigns
```

**Request Body:**

```json
{
  "name": "Q1 Product Launch Campaign",
  "type": "email_marketing",
  "target_audience": {
    "segments": ["enterprise_customers", "trial_users"],
    "criteria": {
      "company_size": "50-500",
      "industry": ["technology", "finance"],
      "engagement_score": ">75"
    }
  },
  "content": {
    "subject": "Introducing Our Latest Innovation",
    "template_id": "template_123",
    "personalization": {
      "dynamic_fields": ["first_name", "company", "last_purchase"]
    }
  },
  "schedule": {
    "start_date": "2025-01-15T09:00:00Z",
    "timezone": "America/New_York",
    "frequency": "once"
  },
  "integration_settings": {
    "crm_sync": true,
    "attribution_tracking": true
  }
}
```

**Response:**

```json
{
  "campaign_id": "camp_789xyz",
  "status": "created",
  "scheduled_for": "2025-01-15T09:00:00Z",
  "api_endpoints": {
    "performance": "/api/v1/campaigns/camp_789xyz/performance",
    "analytics": "/api/v1/campaigns/camp_789xyz/analytics",
    "manage": "/api/v1/campaigns/camp_789xyz"
  }
}
```

### Get Campaign Performance

```http
GET /api/v1/campaigns/{campaign_id}/performance
```

**Response:**

```json
{
  "campaign_id": "camp_789xyz",
  "metrics": {
    "sent": 12500,
    "delivered": 12400,
    "opened": 3720,
    "clicked": 892,
    "converted": 156,
    "bounced": 85,
    "unsubscribed": 12
  },
  "rates": {
    "delivery_rate": 99.3,
    "open_rate": 30.0,
    "click_rate": 7.2,
    "conversion_rate": 1.3
  },
  "revenue": {
    "attributed_revenue": 45200.00,
    "cost_per_acquisition": 89.50,
    "roi": 405.2
  }
}
```

## Lead Management API

### Capture Lead

```http
POST /api/v1/leads
```

**Request Body:**

```json
{
  "source": "website_form",
  "campaign_id": "camp_789xyz",
  "contact": {
    "email": "john.doe@company.com",
    "first_name": "John",
    "last_name": "Doe",
    "company": "Tech Corp",
    "phone": "+1-555-0123",
    "title": "Marketing Director"
  },
  "qualification": {
    "lead_score": 85,
    "interest_level": "high",
    "budget_range": "25000-50000",
    "timeline": "1-3_months"
  },
  "tracking": {
    "utm_source": "google_ads",
    "utm_medium": "cpc",
    "utm_campaign": "q1_product_launch"
  }
}
```

**Response:**

```json
{
  "lead_id": "lead_456abc",
  "status": "captured",
  "crm_sync": {
    "status": "pending",
    "estimated_sync": "2025-12-19T15:30:00Z"
  },
  "next_actions": [
    "assign_to_sales_rep",
    "trigger_nurture_sequence",
    "schedule_follow_up"
  ]
}
```

### Update Lead Status

```http
PATCH /api/v1/leads/{lead_id}
```

**Request Body:**

```json
{
  "status": "qualified",
  "sales_stage": "discovery_call_scheduled",
  "sales_rep": "rep_789",
  "notes": "High-value prospect, interested in enterprise features",
  "follow_up_date": "2025-01-25T14:00:00Z"
}
```

## Analytics API

### Get Campaign Analytics

```http
GET /api/v1/analytics/campaigns?start_date=2025-01-01&end_date=2025-01-31
```

**Response:**

```json
{
  "summary": {
    "total_campaigns": 8,
    "total_recipients": 125000,
    "average_open_rate": 28.5,
    "average_click_rate": 6.8,
    "total_revenue": 385000,
    "overall_roi": 325.8
  },
  "campaigns": [
    {
      "campaign_id": "camp_789xyz",
      "name": "Q1 Product Launch",
      "metrics": {
        "sent": 12500,
        "opened": 3750,
        "clicked": 900,
        "revenue": 45200
      }
    }
  ],
  "insights": [
    {
      "type": "performance_optimization",
      "message": "Email campaigns sent on Tuesday show 15% higher open rates",
      "recommendation": "Consider scheduling campaigns for Tuesday launches"
    }
  ]
}
```

## Related Documentation

- [API Overview](./)
- [Authentication](./authentication)
- [Integration APIs](./integration-apis)
