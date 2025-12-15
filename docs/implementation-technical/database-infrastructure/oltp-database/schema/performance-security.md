---
title: "Performance and Security"
description: "Indexes, Row Level Security policies, and analytics integration for PenguinMails OLTP"
last_modified_date: "2025-12-15"
level: "2"
persona: "Engineering Teams"
keywords: "indexes, RLS, row level security, performance, analytics, monitoring, OLTP"
---

# Performance and Security

Performance optimization and security enforcement are foundational to OLTP operations. This document covers critical indexes, Row Level Security (RLS) policies, and external analytics integration patterns.

## Critical OLTP Indexes

Indexes are designed for the most common access patterns: multi-tenant filtering, campaign orchestration, and user management.

```sql
-- Multi-tenant filtering
CREATE INDEX idx_campaigns_tenant ON campaigns(tenant_id);
CREATE INDEX idx_leads_tenant ON leads(tenant_id);
CREATE INDEX idx_domains_tenant ON domains(tenant_id);
CREATE INDEX idx_email_accounts_tenant ON email_accounts(tenant_id);

-- Campaign orchestration
CREATE INDEX idx_campaign_sequence_steps_campaign_order 
    ON campaign_sequence_steps(campaign_id, step_order);

-- User and tenant management
CREATE INDEX idx_tenant_users_tenant ON tenant_users(tenant_id);
CREATE INDEX idx_tenant_users_user ON tenant_users(user_id);

-- Infrastructure
CREATE INDEX idx_smtp_ip_addresses_status ON smtp_ip_addresses(status);
CREATE INDEX idx_domain_ip_assignments_domain ON domain_ip_assignments(domain_id);
```

### Index Design Principles

**Tenant-first indexing** ensures that all multi-tenant queries filter efficiently. Every table with `tenant_id` has an index on that column as the leading key.

**Composite indexes** for campaign steps order by `campaign_id, step_order` to support range scans when processing sequences.

**Status indexes** on infrastructure tables enable quick lookups for available IPs and active assignments without full table scans.

## Row Level Security Implementation

RLS policies enforce tenant isolation at the database level. Even if application code has bugs, the database prevents cross-tenant data access.

### Enabling RLS

```sql
ALTER TABLE campaigns ENABLE ROW LEVEL SECURITY;
ALTER TABLE leads ENABLE ROW LEVEL SECURITY;
ALTER TABLE domains ENABLE ROW LEVEL SECURITY;
ALTER TABLE email_accounts ENABLE ROW LEVEL SECURITY;
ALTER TABLE templates ENABLE ROW LEVEL SECURITY;
ALTER TABLE companies ENABLE ROW LEVEL SECURITY;
ALTER TABLE user_preferences ENABLE ROW LEVEL SECURITY;
ALTER TABLE tenant_config ENABLE ROW LEVEL SECURITY;
ALTER TABLE tenant_policies ENABLE ROW LEVEL SECURITY;
```

### Tenant Isolation Policies

NileDB sets session variables at the start of each request. RLS policies reference these variables to filter data automatically.

```sql
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
```

### User-Specific Policies

User preferences require user-level isolation rather than tenant-level.

```sql
CREATE POLICY user_preferences_isolation ON user_preferences
    FOR ALL USING (user_id = current_setting('app.current_user_id')::uuid);
```

### Tenant Configuration Policies

```sql
CREATE POLICY tenant_config_isolation ON tenant_config
    FOR ALL USING (tenant_id = current_setting('app.current_tenant_id')::uuid);

CREATE POLICY tenant_policies_isolation ON tenant_policies
    FOR ALL USING (tenant_id = current_setting('app.current_tenant_id')::uuid);
```

## External Analytics Integration

Infrastructure monitoring and security events are externalized to specialized analytics platforms for better observability.

### Connection Pool Monitoring

External analytics track connection pool metrics including utilization rates, connection leaks, and performance trends.

- **Events**: `connection_pool_metrics`, `pool_utilization`, `connection_leaks`
- **Tracking**: Connection pool performance, utilization rates, leak detection
- **Platform**: External analytics (PostHog, Segment, or similar)
- **Benefits**: Better visualization, alerting capabilities, historical trends

### Security Event Monitoring

Security events flow to external platforms with security-focused analysis capabilities.

- **Events**: `security_incidents`, `authentication_failures`, `suspicious_activity`
- **Tracking**: Security events, audit trails, incident patterns
- **Benefits**: Centralized security analytics, threat detection, compliance reporting

### Infrastructure Metrics

System performance data integrates with monitoring platforms for proactive alerting.

- **Events**: `system_performance`, `error_rates`, `resource_utilization`
- **Tracking**: Application performance, error rates, system health
- **Benefits**: Unified monitoring dashboard, proactive alerting

See [External Analytics Integration Plan](/docs/implementation-technical/database-infrastructure/oltp-database/external-analytics-logging) for detailed implementation strategies and event schemas.

## Business Impact

### Revenue and Performance Intelligence

The `billing_analytics` table centralizes tenant usage tracking with period-based aggregation. Explicit limits in the `plans` table support enterprise pricing models with clear feature boundaries.

Subscription lifecycle management through `pending_plan_id` enables seamless plan upgrades and downgrades at billing cycle boundaries without complex proration.

### Operational Excellence

The four-tier architecture maintains clear separation between OLTP operations, content storage, analytics, and job processing. This separation optimizes each layer for its specific workload.

Multi-tenant security with NileDB-managed authentication uses ARRAY-type roles for flexible permission assignment. Queue-driven processing ensures reliable job execution with retry logic and dead letter queue handling.

## Performance Targets

- **OLTP Query Performance**: Sub-500ms for all operational queries
- **Campaign Operations**: 60-80% improvement over baseline
- **Content DB Throughput**: Handle 100K+ message analytics operations per hour
- **Cross-Database Queries**: Under 500ms for campaign plus message analytics
- **Queue Integration**: Under 1 second for email to email_messages creation

## Related Documentation

- [Schema Overview](overview)
- [Multi-Tenant Core](multi-tenant-core)
- [Database Infrastructure](/docs/implementation-technical/database-infrastructure)
