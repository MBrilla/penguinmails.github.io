---
title: "Rule-Based Automation Framework"
description: "Trigger rules, condition rules, and action rules for customer success automation"
last_modified_date: "2025-12-16"
level: "2"
persona: "Documentation Users"
---

# Rule-Based Automation Framework

## 1. Trigger Rules

### Customer Health Score Triggers

```yaml
Health Score Automation Rules:
  Critical Health (Score < 30):
    - Immediate escalation to senior CSM
    - Automated intervention playbook deployment
    - Executive notification for enterprise accounts
    - Account freeze until resolution

  Declining Health (Score 30-50):
    - Automated check-in scheduling
    - Personalized outreach templates
    - Success plan review initiation
    - Product adoption audit

  Stable Health (Score 50-70):
    - Automated quarterly business reviews
    - Expansion opportunity identification
    - Cross-sell recommendation engine
    - Satisfaction survey deployment

  Optimal Health (Score > 70):
    - Advocate program enrollment
    - Case study development
    - Referral program activation
    - Premium feature upsell
```

### Usage-Based Triggers

```yaml
Usage Pattern Automation:
  Low Usage Detection (< 30% of licensed users):
    - Training session auto-scheduling
    - Feature adoption campaigns
    - Champion identification
    - Success milestone celebration

  High Usage Growth (> 20% MoM):
    - Capacity planning alerts
    - Expansion opportunity scoring
    - Usage-based pricing optimization
    - Success story documentation

  No Usage (0% active users):
    - Emergency intervention protocols
    - Stakeholder escalation
    - Contract renewal risk assessment
    - Onboarding process review
```

## 2. Condition Rules

### Contextual Decision Logic

```yaml
Automation Conditions:
  Account Tier Conditions:
    Enterprise:
      - 24/7 monitoring enabled
      - Dedicated success manager assigned
      - Executive business reviews scheduled
      - Custom success metrics tracked

    Mid-Market:
      - Business hours monitoring
      - Shared success manager assignment
      - Quarterly success reviews
      - Standard success metrics

    Small Business:
      - Automated monitoring
      - Self-service resources prioritized
      - Bi-annual check-ins
      - Basic success metrics

  Industry-Specific Conditions:
    Healthcare:
      - Compliance monitoring (HIPAA, FDA)
      - Security audit automation
      - Training certification tracking
      - Patient data protection protocols

    Financial Services:
      - Regulatory compliance (SOX, PCI-DSS)
      - Audit trail automation
      - Risk assessment workflows
      - Data governance protocols

    E-commerce:
      - Revenue impact monitoring
      - Seasonality-based planning
      - Customer lifetime value tracking
      - Shopping behavior analysis
```

## 3. Action Rules

### Automated Response Workflows

```yaml
Automated Actions:
  Immediate Actions (0-1 hours):
    - Critical alert notifications
    - Stakeholder escalation emails
    - Support ticket creation
    - Calendar hold placement

  Short-term Actions (1-24 hours):
    - Success plan updates
    - Training session scheduling
    - Resource provisioning
    - Stakeholder notifications

  Medium-term Actions (1-7 days):
    - Success review meetings
    - Feature adoption campaigns
    - Expansion discussions
    - Contract renewal planning

  Long-term Actions (1-4 weeks):
    - Strategic planning sessions
    - Success metric reviews
    - Team capacity planning
    - Process optimization initiatives
```
