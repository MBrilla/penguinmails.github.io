---
title: "Executive Dashboard Backend API"
description: "API endpoints, response formats, and integration specifications"
last_modified_date: "2025-11-19"
level: "3"
persona: "Backend Engineers, API Developers"
---

# Executive Dashboard Backend API

API endpoints, response formats, and integration specifications for the Executive Dashboard.

## Executive Dashboard API Endpoints

### 1. Dashboard Data Retrieval

```http
GET /api/executive-dashboard/{tenantId}/summary
```

**Response:**

```json
{
  "tenantId": "string",
  "overallBusinessHealth": 85,
  "revenueProtection": {
    "deliverabilityRate": 98.5,
    "bounceRate": 1.2,
    "spamRate": 0.3,
    "revenueAtRisk": 2500,
    "riskLevel": "low"
  },
  "costOptimization": {
    "monthlySavings": 1500,
    "optimizationOpportunities": 3,
    "costEfficiencyRatio": 2.1,
    "infrastructureCosts": 8500,
    "emailServiceCosts": 1200
  },
  "operationalEfficiency": {
    "overallScore": 87,
    "resourceUtilization": 78,
    "performanceScore": 92,
    "efficiencyTrend": "improving"
  },
  "strategicDecisions": {
    "decisionsTracked": 12,
    "avgDecisionTime": "36 hours",
    "successRate": 85,
    "roiAchievement": 120
  },
  "alerts": [
    {
      "id": "alert-123",
      "type": "revenue_risk",
      "severity": "warning",
      "message": "Bounce rate approaching threshold",
      "timestamp": "2025-12-19T10:30:00Z",
      "actionsRequired": ["Review email content", "Check IP reputation"]
    }
  ],
  "lastUpdated": "2025-12-19T10:35:00Z"
}
```

### 2. Real-time Business Events

```http
GET /api/executive-dashboard/{tenantId}/events/realtime
```

**WebSocket Event Format:**

```json
{
  "eventType": "revenue_impact",
  "tenantId": "string",
  "data": {
    "impactAmount": 2500,
    "impactType": "negative",
    "urgencyLevel": "high",
    "actionRequired": "IP warmup and content review"
  },
  "timestamp": "2025-12-19T10:35:00Z"
}
```

### 3. Cost Analysis Deep Dive

```http
GET /api/executive-dashboard/{tenantId}/cost-analysis
```

**Response:**

```json
{
  "tenantId": "string",
  "costBreakdown": {
    "infrastructure": {
      "vpsInstances": {
        "totalCost": 8500,
        "costPerTenant": 212.50,
        "utilizationRate": 78,
        "optimizationPotential": 1200
      },
      "smtpIps": {
        "totalCost": 1200,
        "costPerIp": 25,
        "utilizationRate": 85,
        "optimizationPotential": 300
      }
    },
    "emailService": {
      "totalCost": 1500,
      "costPerDeliveredEmail": 0.015,
      "deliverabilityRate": 98.5,
      "optimizationPotential": 450
    }
  },
  "trendAnalysis": {
    "costTrend": "decreasing",
    "efficiencyTrend": "improving",
    "optimizationImpact": 1950
  },
  "recommendations": [
    {
      "category": "infrastructure",
      "description": "Right-size VPS instances for low-utilization tenants",
      "estimatedSavings": 1200,
      "implementationEffort": "low"
    }
  ]
}
```

## Integration Points

### External Systems Integration

- **PostHog Analytics:** Business event tracking and real-time insights
- **Deliverability Providers:** SendGrid, Mailgun, Amazon SES API integration
- **Infrastructure Monitoring:** VPS provider APIs for real-time usage data
- **Financial Systems:** Billing and subscription management integration

### Internal System Integration

- **OLTP Database:** Executive summary views and cost allocation data
- **Authentication System:** Role-based access and user management
- **Notification System:** Alert distribution and escalation management
- **Report Generation:** Automated executive reporting and export capabilities

## Related Documentation

- [Executive Dashboard Overview](/docs/implementation-technical/business-intelligence/executive-dashboard/overview) - Main page
- [Business Requirements](/docs/implementation-technical/business-intelligence/executive-dashboard/requirements) - Success criteria
- [Technical Architecture](/docs/implementation-technical/business-intelligence/executive-dashboard/architecture) - Component structure
- [Frontend Implementation](/docs/implementation-technical/business-intelligence/executive-dashboard/frontend) - React components
