---
title: "Domain Integration APIs"
description: "Marketing, Sales, Product, Finance, and Operations API integration endpoints and patterns"
last_modified_date: "2025-12-05"
level: "3"
persona: "Integration Teams, API Developers"
---

# Domain-Specific Integration APIs

Comprehensive API specifications for integrating Customer Success with Marketing, Sales, Product, Finance, and Operations systems.

## Marketing Integration APIs

### Health-to-Marketing API

```json
GET /api/v1/cs-marketing/health/{customer_id}

- Customer health score and trending
- Campaign trigger recommendations
- Lifecycle stage determination

POST /api/v1/cs-marketing/campaigns

- Triggers lifecycle campaigns from CS signals
- Tracks campaign attribution to success metrics
- Measures retention/expansion impact

Response:
{
  "campaign_id": "camp_67890",
  "trigger_type": "health_decline",
  "campaign_type": "retention",
  "target_audience": {
    "customer_segments": ["at_risk", "low_engagement"],
    "size": 1250
  },
  "estimated_impact": {
    "retention_improvement": "15-20%",
    "engagement_lift": "25%"
  }
}
```

### Customer Journey Coordination

```json
POST /api/v1/cs-marketing/journey/{customer_id}

- Updates customer journey stage
- Triggers appropriate marketing touchpoints
- Coordinates messaging with success activities

GET /api/v1/cs-marketing/journey/{customer_id}/status

- Returns current journey stage
- Next recommended actions
- Historical journey progression
```

## Sales Integration APIs

### Handoff API

```json
POST /api/v1/cs-sales/handoff/{customer_id}

- Completes sales-to-CS transition
- Initiates success plan execution
- Establishes baseline metrics

Request:
{
  "customer_id": "cust_12345",
  "handoff_data": {
    "deal_value": 150000,
    "contract_terms": "3_year",
    "key_stakeholders": ["john.doe@company.com"],
    "success_criteria": ["adoption_target", "roi_target"],
    "sales_notes": "Strong technical champion, budget confirmed"
  },
  "success_initiation": {
    "onboarding_start": "2025-12-05",
    "success_manager": "sarah.smith@company.com",
    "first_check_in": "2025-12-12"
  }
}
```

### Expansion Pipeline API

```json
GET /api/v1/cs-sales/expansion/{customer_id}

- Returns expansion readiness score
- Provides success evidence portfolio
- Identifies cross-sell opportunities

Response:
{
  "expansion_readiness": {
    "score": 78,
    "readiness_factors": [
      "high_product_adoption",
      "strong_health_score",
      "recent_success_milestones"
    ]
  },
  "opportunities": [
    {
      "type": "upsell",
      "product": "premium_features",
      "value_estimate": 50000,
      "probability": 0.75,
      "timeline": "q1_2026"
    }
  ],
  "success_evidence": [
    "achieved_roi_target",
    "executive_sponsorship",
    "expansion_requested_by_customer"
  ]
}
```

## Product Integration APIs

### Adoption Insights API

```json
GET /api/v1/cs-product/adoption/{feature_id}

- Feature adoption by customer segment
- Success correlation analysis
- Barrier and opportunity identification

Response:
{
  "feature_id": "feat_analytics",
  "adoption_metrics": {
    "overall_adoption": 0.65,
    "segment_adoption": {
      "enterprise": 0.85,
      "mid_market": 0.60,
      "small_business": 0.45
    }
  },
  "success_correlation": {
    "adoption_success_link": 0.82,
    "health_impact": "high",
    "retention_correlation": 0.73
  },
  "improvement_opportunities": [
    "enhanced_onboarding",
    "training_content",
    "feature_discovery"
  ]
}
```

### Customer Feedback API

```json
POST /api/v1/cs-product/feedback

- Submits prioritized customer feedback
- Includes success impact assessment
- Tracks delivery commitment

Request:
{
  "customer_id": "cust_12345",
  "feedback": {
    "type": "feature_request",
    "category": "workflow_automation",
    "priority": "high",
    "business_impact": "critical_for_success",
    "description": "Need automated health score triggers"
  },
  "success_context": {
    "health_impact": "churn_risk_reduction",
    "efficiency_gain": "40% reduction in manual work",
    "customer_value": "proactive_success_intervention"
  }
}
```

## Finance Integration APIs

### Value Realization API

```json
GET /api/v1/cs-finance/value/{customer_id}

- Contractual value realization tracking
- ROI attribution breakdown
- Expansion revenue contribution

Response:
{
  "customer_id": "cust_12345",
  "value_realization": {
    "contracted_value": 150000,
    "realized_value": 180000,
    "roi_percentage": 120,
    "realization_timeline": "18_months"
  },
  "revenue_attribution": {
    "success_contribution": 0.65,
    "churn_prevention_value": 25000,
    "expansion_revenue": 30000
  },
  "financial_health": {
    "payment_status": "current",
    "renewal_probability": 0.92,
    "expansion_pipeline": 75000
  }
}
```

### Revenue Forecasting API

```json
POST /api/v1/cs-finance/forecast

- Submits CS-informed revenue forecast
- Includes risk-adjusted probabilities
- Supports financial planning models

Request:
{
  "forecast_period": "q2_2026",
  "customer_forecasts": [
    {
      "customer_id": "cust_12345",
      "renewal_probability": 0.95,
      "expansion_probability": 0.70,
      "expansion_value": 50000,
      "risk_factors": ["contract_renewal_timing"]
    }
  ],
  "aggregate_metrics": {
    "total_forecasted_revenue": 2500000,
    "confidence_level": 0.88,
    "risk_adjusted_value": 2200000
  }
}
```

## Operations Integration APIs

### Workflow Automation API

```json
POST /api/v1/cs-ops/workflows/{customer_id}

- Executes automated CS workflows
- Triggers operational escalations
- Captures process performance data

Request:
{
  "workflow_type": "health_decline_intervention",
  "customer_id": "cust_12345",
  "trigger_conditions": {
    "health_score_drop": 20,
    "usage_decline": 0.30,
    "support_increase": 0.50
  },
  "automation_steps": [
    "send_immediate_alert",
    "schedule_priority_check_in",
    "deploy_success_playbook",
    "notify_account_team"
  ]
}
```

### Performance Monitoring API

```json
GET /api/v1/cs-ops/performance

- Real-time CS operational metrics
- Resource utilization dashboards
- Optimization opportunity alerts

Response:
{
  "performance_metrics": {
    "workflow_execution_rate": 0.95,
    "average_resolution_time": "2.3_hours",
    "customer_satisfaction": 4.6,
    "team_productivity": 1.15
  },
  "resource_utilization": {
    "capacity_usage": 0.78,
    "skill_matching_accuracy": 0.92,
    "workload_balancing": "optimal"
  },
  "optimization_opportunities": [
    "increase_automation_coverage",
    "improve_skill_matching_algorithms",
    "expand_predictive_capabilities"
  ]
}
```

---

**Related Documentation:**

- [API Integration Overview](./overview) - Architecture and RESTful design patterns
- [Webhooks and Security](./webhooks-security) - Real-time notifications and authentication

---

**Document Classification:** Level 3 - Implementation Procedures
**Business Value:** Comprehensive domain APIs enabling automated cross-functional coordination
