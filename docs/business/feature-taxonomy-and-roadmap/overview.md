---
title: "Feature Taxonomy Overview"
description: "Feature classification system with Core and MVP feature definitions"
last_modified_date: "2025-11-17"
level: "2"
persona: "Product, Engineering, Sales, Marketing Teams"
---

# Feature Taxonomy and Progressive Roadmap Framework

This document provides the central reference for understanding our feature classification system and progressive development roadmap. It serves as the single source of truth for feature prioritization, scope definition, and development planning.

**Document Level:** Level 1 - Strategic Framework
**Target Audience:** All teams (Product, Engineering, Sales, Marketing, Customer Success)
**Purpose:** Feature classification, scope clarification, and progressive development planning

## Feature Classification System

### Progressive Disclosure Levels

Our feature development follows a progressive disclosure approach, revealing capabilities in levels based on market validation, technical complexity, and business impact:

```
Level 1: CORE FEATURES (MVP Foundation)        → 2026 Q1
Level 2: MVP FEATURES (Basic UX Requirements)  → 2026 Q2
Level 3: GROWTH FEATURES (Market Expansion)    → 2026 Q3 - Q4
Level 4: ENTERPRISE FEATURES (Scale Features)  → 2027 - 2028
Level 5: FUTURE FEATURES (Innovation Pipeline) → 2028+
```

## Level 1: CORE FEATURES (MVP Foundation)

### Purpose

Essential infrastructure and basic functionality required for email infrastructure management.

### Scope Definition

**Timeline**: 2026 Q1 (Months 1-2)
**Team Size**: 3-4 engineers
**Investment**: $50K-75K development cost

### Features Included

#### Infrastructure Core

- [ ] **Email Infrastructure Setup**: VPS provisioning, MailU SMTP setup, DNS automation
- [ ] **Agency Client Workspaces**: Client isolation, workspace management, role-based access
- [ ] **Basic Automated Warmup**: Automated volume progression and reputation building (Shared IP)
- [ ] **Smart Subdomain Setup**: Automated separation of marketing, transactional, and cold email subdomains
- [ ] **Basic Security**: SSL certificates, SPF/DKIM authentication
- [ ] **Compliance Foundation**: GDPR/CCPA compliance, data handling

#### Basic Analytics

- [ ] **Directional KPI Tracking**: Basic metrics visibility with 75% directional accuracy
- [ ] **Manual Reporting**: Weekly/monthly reports with export capabilities
- [ ] **Basic Dashboard**: Desktop-first interface with responsive design

#### Integration Foundation

- [ ] **ESP Integration**: Postmark, Mailgun integration with basic functionality
- [ ] **Export Capabilities**: Data export to external tools for advanced analysis

### Success Criteria

- 10-20 beta customers successfully onboarded
- >85% infrastructure setup success rate
- <60 minutes average time-to-first-send
- >80% average deliverability

### Technology Constraints

- Manual processes for complex decisions
- Third-party tools for advanced analytics
- Directional insights instead of real-time monitoring
- Export-first approach for advanced capabilities

## Level 2: MVP FEATURES (Basic UX Requirements)

### Purpose

Complete the minimum viable product experience with essential user workflows.

### Scope Definition

**Timeline**: 2026 Q2 (Months 3-6)
**Team Size**: 4-5 engineers
**Investment**: $100K-150K development cost

### Features Included

#### Enhanced User Experience

- [ ] **Campaign Management**: Email sequencing, basic A/B testing, personalization tokens
- [ ] **Improved Dashboard**: Enhanced analytics with manual optimization recommendations
- [ ] **User Onboarding**: Guided setup process and feature discovery
- [ ] **Migration Tools**: CSV/JSON contact import and basic campaign state import

#### Analytics Enhancement

- [ ] **Basic Performance Tracking**: Campaign performance with directional insights
- [ ] **Manual Optimization**: Basic optimization recommendations based on performance data
- [ ] **Weekly Automation**: Automated weekly performance reports

#### Integration Expansion

- [ ] **CRM Integration**: Basic Salesforce, HubSpot integration
- [ ] **Webhook Support**: Event-driven integrations with external systems

### Success Criteria

- 75+ active customers
- >85% deliverability
- $25K MRR
- <4 hours weekly operational time

### Technology Constraints

- Manual intervention for complex optimization
- Basic automation for routine tasks
- Third-party analytics tools for advanced insights
- Desktop-first experience with basic mobile responsiveness

---

**Related Documentation:**

- [Growth and Enterprise Features](./growth-enterprise) - Level 3 and 4 feature specifications
- [Prioritization and Metrics](./prioritization-metrics) - Decision framework and success metrics

---

**Document Classification:** Level 1 - Strategic Framework
**Business Value:** Feature classification for development planning and scope definition
