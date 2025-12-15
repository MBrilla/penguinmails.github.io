---
title: "Schema Migrations"
description: "Database migration strategies"
last_modified_date: "2025-12-04"
level: "3"
keywords: "migrations, schema changes, versioning"
---

# Schema Migrations

## Performance Indexes

```sql
-- User and tenant management
CREATE INDEX idx_tenant_users_tenant ON tenant_users(tenant_id);
CREATE INDEX idx_tenant_users_user ON tenant_users(user_id);

-- Infrastructure
CREATE INDEX idx_smtp_ip_addresses_status ON smtp_ip_addresses(status);
CREATE INDEX idx_domain_ip_assignments_domain ON domain_ip_assignments(domain_id);

```

---

## Security & RLS Policies

### **Row Level Security Implementation**

```sql
-- Enable RLS on all multi-tenant tables
ALTER TABLE campaigns ENABLE ROW LEVEL SECURITY;
ALTER TABLE leads ENABLE ROW LEVEL SECURITY;
ALTER TABLE domains ENABLE ROW LEVEL SECURITY;
ALTER TABLE email_accounts ENABLE ROW LEVEL SECURITY;
ALTER TABLE templates ENABLE ROW LEVEL SECURITY;
ALTER TABLE companies ENABLE ROW LEVEL SECURITY;
ALTER TABLE user_preferences ENABLE ROW LEVEL SECURITY;
ALTER TABLE tenant_config ENABLE ROW LEVEL SECURITY;
ALTER TABLE tenant_policies ENABLE ROW LEVEL SECURITY;

-- Tenant isolation policies
CREATE POLICY campaigns_tenant_isolation ON campaigns
    FOR ALL USING (tenant_id = current_setting('app.current_tenant_id')::uuid);

CREATE POLICY leads_tenant_isolation ON leads
    FOR ALL USING (tenant_id = current_setting('app.current_tenant_id')::uuid);

CREATE POLICY domains_tenant_isolation ON domains
    FOR ALL USING (tenant_id = current_setting('app.current_tenant_id')::uuid);

CREATE POLICY email_accounts_tenant_isolation ON email_accounts
    FOR ALL USING (tenant_id = current_setting('app.current_tenant_id')::uuid);

CREATE POLICY templates_tenant_isolation ON templates
    FOR ALL USING (tenant_id = current_setting('app.current_tenant_id')::uuid);

CREATE POLICY companies_tenant_isolation ON companies
    FOR ALL USING (tenant_id = current_setting('app.current_tenant_id')::uuid);

-- User-specific isolation for preferences
CREATE POLICY user_preferences_isolation ON user_preferences
    FOR ALL USING (user_id = current_setting('app.current_user_id')::uuid);

-- Tenant-wide configuration isolation
CREATE POLICY tenant_config_isolation ON tenant_config
    FOR ALL USING (tenant_id = current_setting('app.current_tenant_id')::uuid);

CREATE POLICY tenant_policies_isolation ON tenant_policies
    FOR ALL USING (tenant_id = current_setting('app.current_tenant_id')::uuid);


```

---

## External Analytics Integration

### **Monitoring & Observability**

The following infrastructure and monitoring concerns have been externalized to specialized analytics platforms for better observability and product insights:

#### **Connection Pool Monitoring**

- **External Events**: `connection_pool_metrics`, `pool_utilization`, `connection_leaks`

- **Tracking**: Connection pool performance, utilization rates, leak detection

- **Platform**: External analytics platform (PostHog, Segment, or similar)

- **Benefits**: Better visualization of database performance trends, alerting capabilities

#### **Security Event Monitoring**

- **External Events**: `security_incidents`, `authentication_failures`, `suspicious_activity`

- **Tracking**: Security events, audit trails, incident patterns

- **Platform**: External analytics platform with security focus

- **Benefits**: Centralized security analytics, threat detection, compliance reporting

#### **Infrastructure Metrics**

- **External Events**: `system_performance`, `error_rates`, `resource_utilization`

- **Tracking**: Application performance, error rates, system health

- **Platform**: Infrastructure monitoring platform

- **Benefits**: Unified monitoring dashboard, proactive alerting

> 📋 **See [External Analytics Integration Plan](/docs/implementation-technical/database-infrastructure/oltp-database/external-analytics-logging)** for detailed implementation strategies, event schemas, and platform migration guidelines.

---

## Business Impact & Technical Excellence

### Revenue & Performance Intelligence

- **Unified Billing Analytics**: `billing_analytics` table centralizes all tenant usage tracking with period-based aggregation

- **Enhanced Plan Flexibility**: Explicit limits in `plans` table support enterprise pricing models

- **Subscription Lifecycle**: `pending_plan_id` enables seamless plan upgrades/downgrades at billing cycle end

- **Separate Billing Contacts**: `billing_contact_user_id` allows different billing emails from tenant accounts

### Operational Excellence Achievements

- **4-Tier Architecture**: Clear separation between OLTP operations, content storage, analytics, and job processing

- **Multi-Tenant Security**: Row-level security with NileDB-managed authentication using ARRAY-type roles

- **Infrastructure Intelligence**: `admin_system_events` provides comprehensive system monitoring and alerting

- **Queue-Driven Processing**: Reliable job processing with retry logic and dead letter queues

### Technical Architecture Excellence

- **Data Collection Strategy**:

  - **OLTP Layer**: Fast transactional operations for real-time business logic (users, campaigns, leads)

  - **Content Layer**: Heavy email storage with retention policies and compression

  - **Analytics Layer**: Aggregated metrics with OLAP optimization for dashboards

  - **Queue Layer**: Asynchronous processing with Redis + PostgreSQL hybrid storage

---

## Success Metrics & Validation

### Performance Targets

- **OLTP Query Performance**: 60-80% improvement in campaign operations

- **Content DB Throughput**: Handle 100K+ message analytics operations/hour

- **Cross-Database Queries**: <500ms for campaign + message analytics

- **Queue Integration**: <1 second for email to email_messages creation

---

## Related Documents

### Supporting Documentation

- [Database Infrastructure](/docs/implementation-technical/database-infrastructure) - Database and infrastructure overview

- [Architecture System](/docs/implementation-technical/architecture-system/architecture-overview) - System architecture decisions

- [Development Guidelines](/docs/implementation-technical/development-guidelines) - Development standards

- [Quality Assurance](/docs/business/quality-assurance) - Testing protocols and procedures

### Business Integration

- [Business Strategy Overview](/docs/business/strategy/overview) - Strategic business alignment

- [Operations Management](/docs/operations/analytics/operations-management) - Operational procedures

- [Security Framework](/docs/compliance-security/enterprise/security-framework) - Security architecture

- [Analytics Performance](/docs/operations/analytics/analytics-performance) - Performance monitoring

---

**Keywords**: OLTP, operational database, transactional processing, multi-tenant, row level security, performance optimization, database schema, real-time operations, unified security
---
