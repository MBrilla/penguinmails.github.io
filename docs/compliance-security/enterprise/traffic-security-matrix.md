---
last_modified_date: "2025-11-19"
level: "2"
persona: "Documentation Users"
---


# Traffic & Security Matrix Guide - Primary Key Strategy Framework

---

## **Strategic Alignment**

**Strategic Alignment**: This security framework supports our enterprise compliance strategy by providing regulatory compliance and risk mitigation across database security and performance optimization, establishing market leadership through strategic primary key selection and multi-tenant data isolation excellence.

**Technical Authority**: Our security infrastructure integrates with comprehensive monitoring systems featuring real-time traffic analysis, automated security classification, and enterprise-grade database optimization, positioning us as a technical authority in multi-tenant database architecture security.

**Operational Excellence**: Backed by enterprise database platforms with 99.9% security uptime, advanced traffic monitoring, and automated threat detection, ensuring reliable and secure data operations across all operational domains.

**User Journey Integration**: This security feature is part of your complete compliance and data protection experience - connects to user authentication workflows, data management processes, and privacy controls throughout your entire data interaction journey.

---

## Overview

This guide provides the framework for selecting appropriate Primary Key types based on traffic patterns and security requirements in multi-tenant database architectures.

## Framework Matrix

```markdown
                    SECURITY DANGER LEVELS
TRAFFIC    |    LOW                    |   MEDIUM                  |   HIGH
-----------|--------------------------|---------------------------|--------------------------
CRITICAL   | BIGINT                   | BIGINT                    | UUID
HIGH       | BIGINT                   | UUID                      | UUID
MEDIUM     | BIGINT                   | UUID                      | UUID
LOW        | INT                      | UUID                      | UUID

Distribution Examples:


- UUID (Security-focused): 75% of tables - User data, credentials, financial records


- BIGINT (Analytics performance): 9% of tables - High-traffic analytics, logs


- VARCHAR (External): 6% of tables - Stripe IDs, Hostwinds IDs, natural keys


- Composite (Multi-tenant): 10% of tables - Tenant associations, junction tables


```

## Security Danger Assessment

### HIGH Security (Always UUID)

- **User Credentials**: `users`, `staff_members`

- **Financial Data**: `payments`, `subscriptions`, `subscription_addons`

- **Authentication**: `tenant_users` (composite, but contains auth data)

- **Audit Trails**: `job_audit`, `content_access_audit`

- **External System IDs**: VARCHAR(255) for Stripe/Hostwinds

### MEDIUM Security (UUID Recommended)

- **Business Operations**: `campaigns`, `templates`, `email_accounts`

- **Infrastructure**: `vps_instances`, `smtp_ip_addresses`

- **Content Storage**: `content_objects` (VARCHAR natural key), `attachments`

- **System Configuration**: `tenant_config`, `tenant_policies`

### LOW Security (BIGINT Possible)

- **Analytics**: All OLAP tables (`*_analytics`)

- **Logs**: `job_logs`, `content_access_log`

- **Junction Tables**: `template_folders`, `template_tags`

- **Internal Metadata**: `folders`, `tags`

## Traffic Assessment

### CRITICAL Traffic (>100K ops)

- **Content Operations**: `content_objects`, `email_opens`

- **Message Processing**: `campaign_sequence_steps` (OLTP), `email_messages` (Content)

- **Message Processing**: `campaign_sequence_steps` (OLTP), `content_inbox_message_refs` (Content)

- **Message Processing**: `inbox_message_refs`, `email_clicks`

- **Analytics**: OLAP tables during processing

### HIGH Traffic (10K-100K ops)

- **Business Operations**: `campaigns`, `job_logs`

- **Queue Processing**: `jobs`, `job_metrics`

- **Content Access**: `content_access_log`

### MEDIUM Traffic (1K-10K ops)

- **Configuration**: `user_preferences`, `tenant_config`

- **Monitoring**: `queue_health`, `worker_performance`

### LOW Traffic (<1K ops)

- **Static Data**: `plans`, `permissions`, `staff_roles`

- **Administrative**: `system_config`, `feature_flags`

## Implementation Rules

### 1. NileDB-Managed Tables (Immutable)

```sql
-- NEVER CHANGE NileDB-managed tables
CREATE TABLE users (id UUID PRIMARY KEY, ...);
CREATE TABLE tenants (id UUID PRIMARY KEY, ...);
CREATE TABLE tenant_users (...); -- Composite key


```

### 2. External System Integration (VARCHAR)

```sql
-- Use VARCHAR(255) for external provider IDs
CREATE TABLE subscriptions (id VARCHAR(255) PRIMARY KEY, ...);
CREATE TABLE vps_instances (hostwinds_instance_id VARCHAR(255), ...);
CREATE TABLE payments (stripe_payment_intent_id VARCHAR(255), ...);


```

### 3. Natural Keys (VARCHAR)

```sql
-- Use appropriate VARCHAR length for natural keys
CREATE TABLE job_queues (name VARCHAR(100) PRIMARY KEY, ...);
CREATE TABLE content_objects (storage_key VARCHAR(500) PRIMARY KEY, ...);


```

### 4. Performance Optimization (BIGINT)

```sql
-- Use BIGINT for high-traffic, low-security analytics
CREATE TABLE campaign_analytics (id BIGINT PRIMARY KEY, ...);
CREATE TABLE admin_audit_log (id BIGINT PRIMARY KEY, ...);


```

### 5. Security Protection (UUID)

```sql
-- Use UUID for user-facing, sensitive data
CREATE TABLE companies (id UUID PRIMARY KEY, ...);
CREATE TABLE leads (id UUID PRIMARY KEY, ...);
CREATE TABLE transactional_emails (id UUID PRIMARY KEY, ...);


```

## Migration Considerations

### When to Migrate (Rare)

- **Security Escalation**: Low security table becomes user-facing

- **Traffic Surge**: Low traffic table becomes high-traffic

- **Business Requirements**: External system ID standardization

### Migration Strategy

1. **Add New Column**: Add new PK column alongside old

2. **Dual Writing**: Write to both old and new PK

3. **Backfill Data**: Migrate existing data

4. **Update References**: Update foreign key constraints

5. **Remove Old Column**: Drop old PK column

6. **Rollback Plan**: Maintain ability to revert

### Migration Example

```sql
-- Step 1: Add new PK column
ALTER TABLE low_security_table ADD COLUMN id_uuid UUID DEFAULT gen_random_uuid();

-- Step 2: Create unique constraint
ALTER TABLE low_security_table ADD CONSTRAINT uk_id_uuid UNIQUE (id_uuid);

-- Step 3: Update references (if needed)
-- Foreign key updates would go here

-- Step 4: Drop old PK, make new PK
ALTER TABLE low_security_table DROP CONSTRAINT pk_old_id CASCADE;
ALTER TABLE low_security_table ADD CONSTRAINT pk_id_uuid PRIMARY KEY (id_uuid);
ALTER TABLE low_security_table DROP COLUMN id_old;


```

## Performance Impact Analysis

### UUID Performance Characteristics

- **Storage**: 16 bytes vs 8 bytes for BIGINT

- **Indexing**: Slightly slower due to size

- **Randomness**: Reduces index fragmentation

- **Security**: Prevents enumeration attacks

- **Global Uniqueness**: No collision risks

### BIGINT Performance Characteristics

- **Storage**: 8 bytes (optimal)

- **Indexing**: Faster lookups

- **Sequential**: Better for range queries

- **Readable**: Easier for debugging

- **Collision Risk**: Minimal with proper sequences

### Real-World Benchmarks

- **UUID Inserts**: ~5-10% slower than BIGINT

- **UUID Queries**: ~2-5% slower than BIGINT

- **Storage Overhead**: ~100% for UUID vs BIGINT

- **Index Size**: ~25-30% larger for UUID

## Security Considerations

### Enumeration Protection

- **URLs**: Avoid exposing sequential IDs in user-visible URLs

- **API Endpoints**: UUID prevents guessing adjacent records

- **Data Leakage**: Sequential IDs can reveal system usage patterns

### Attack Vectors Mitigated

- **ID Enumeration**: UUID prevents guessing valid IDs

- **Resource Discovery**: Random IDs hide data structure

- **Rate Limiting Bypass**: Harder to enumerate valid targets

### When Enumeration Risk is Low

- **Internal APIs**: Non-public endpoints

- **Junction Tables**: Many-to-many relationship tables

- **Audit Logs**: Historical data not user-accessible

- **Analytics**: Aggregated data, not individual records

## Decision Tree for New Tables

```markdown
Start: New Table Creation
    ↓
Does table contain user data, credentials, or financial info?
    ├─ YES → UUID (Security)
    └─ NO → Continue
        ↓
Is table accessible via public APIs or user interfaces?
    ├─ YES → UUID (Enumeration Protection)
    └─ NO → Continue
        ↓
Expected traffic > 10K ops/hour?
    ├─ YES → BIGINT (Performance)
    └─ NO → Continue
        ↓
Is this an analytics/aggregation table?
    ├─ YES → BIGINT (Analytics Pattern)
    └─ NO → Continue
        ↓
Natural key available (name, external ID)?
    ├─ YES → VARCHAR (Natural Key)
    └─ NO → UUID (Default)


```

## Current Implementation Status

### Perfectly Aligned Tables

- **OLTP Security Tables**: UUID (75% of tables)

- **OLAP Analytics Tables**: BIGINT (9% of tables)

- **External System Tables**: VARCHAR (6% of tables)

- **Multi-tenant Junctions**: Composite (10% of tables)

### 🔍 Potential Optimization Candidates

#### LOW Security + LOW Traffic (Consider BIGINT)

- `folders` - Internal organization only

- `template_folders` - Junction table, internal

- `template_tags` - Junction table, internal

- `tags` - Internal tagging system

#### MEDIUM Security + LOW Traffic (Keep UUID)

- `user_preferences` - User data, keep secure

- `tenant_config` - Configuration, maintain security

- `feature_flags` - Operational control, keep secure

## Security Traffic Matrix Integration

### Traffic Assessment Matrix

- **CRITICAL Traffic (>100K ops)**: `content_objects`, `inbox_message_refs`, `email_opens`

- **HIGH Traffic (10K-100K ops)**: `campaigns`, `job_logs`, `content_access_log`

- **MEDIUM Traffic (1K-10K ops)**: `user_preferences`, `tenant_config`, `queue_health`

### Infrastructure Protection Measures

- **Rate Limiting**: API endpoints protected with Redis-based rate limiter (100 requests)

- **Traffic Filtering**: UFW firewall rules with specific IP restrictions for SSH access

- **DDoS Protection**: Cloudflare integration for traffic analysis and mitigation

- **VPN Access**: Mandatory VPN for infrastructure management and database access

### Security Event Monitoring

```javascript
// Security traffic monitoring
const trafficSecurity = {
  monitorTrafficPatterns: (req, res, next) => {
    const clientIP = req.ip;
    const userAgent = req.get('User-Agent');
    const endpoint = req.path;

    // Check for suspicious patterns
    if (isSuspiciousTraffic(clientIP, userAgent, endpoint)) {
      securityLogger.logEvent({
        type: 'suspicious_traffic',
        severity: 'medium',
        details: { clientIP, userAgent, endpoint }
      });
    }

    // Rate limiting check
    const identifier = `${clientIP}:${req.user?.id || 'anonymous'}`;
    const result = rateLimiter.check(identifier, 100, 3600);

    if (!result.allowed) {
      return res.status(429).json({ error: 'Rate limit exceeded' });
    }

    next();
  }
};


```

---

## Conclusion

The current primary key strategy demonstrates excellent architectural judgment and aligns perfectly with security and performance requirements. The 75% UUID usage for security-sensitive tables provides strong protection against enumeration attacks, while the strategic use of BIGINT for analytics ensures optimal performance.

For future table creation, use this matrix to ensure consistent and appropriate primary key selection across the system.

### Related Documents

- [Security Framework](/docs/compliance-security/enterprise/security-framework) - Comprehensive security architecture

- [Security & Privacy Integration](/docs/compliance-security/enterprise/security-privacy-integration) - Unified security and privacy approach

- [Compliance Procedures](/docs/compliance-security/detailed-compliance) - Regulatory compliance workflows

---
