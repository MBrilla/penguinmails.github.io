---
title: "Support and QA Roles"
description: "Guide for support and QA roles within PenguinMails internal staff structure"
last_modified_date: "2025-12-16"
level: "2"
persona: "Internal Staff"
---

# Support and QA Roles

## Overview

This guide documents the support and QA roles within PenguinMails' internal staff structure. These roles have specialized access permissions designed for customer support and testing respectively.

---

## Support Role

**Purpose**: Customer-facing support staff helping tenants troubleshoot issues and resolve tickets.

### Access Level

| Permission | Access |
|------------|---------|
| View tenant information | ✅ Yes - Organization name, plan, usage, settings |
| View audit trails | ✅ Yes - User actions, campaign history, system events |
| View email contents | ❌ **No** - PII protected |
| View password hashes | ❌ **No** - Security protected |
| View personal data | ❌ **No** - PII protected |
| Modify tenant data | ❌ **No** - Read-only access |
| Execute tenant actions | ❌ **No** - Cannot send campaigns, modify settings |

### Key Features

### Tenant Account Lookup

- Search tenants by email, organization name, or tenant ID
- View subscription details and billing status
- Check usage against plan limits

### Audit Trail Viewer

- Review tenant user activity timeline
- Filter by action type, date range, user
- Export audit logs for analysis
- Link actions to support tickets

### Ticket Management

- View and respond to support tickets
- Access tenant context within ticket interface
- Create internal notes (not visible to tenant)

### Knowledge Base Access

- Search internal documentation
- Access troubleshooting guides
- View common issue resolutions

### Routes

- `/dashboard/support/*` - Support panel interface
- `/dashboard/support/tickets` - Ticket queue
- `/dashboard/support/lookup` - Tenant search
- `/dashboard/support/audit` - Audit trail viewer

### Related Documentation

- [Platform Admin Routes](/docs/design/routes/platform-admin) - Shared UI patterns
- [Support Processes](/docs/operations/support/) - Support team workflows
- [PII Protection](/docs/compliance-security/data-privacy/pii-protection) - Data privacy guidelines

---

## QA Role

**Purpose**: Testing role for validating features, bug reproduction, and quality assurance without affecting production data.

### Access Level

| Permission | Access |
|------------|---------|
| Sandbox environment | ✅ Yes - Full access to test environment |
| Limited PII access | ✅ Yes - Synthetic test data only |
| Production database | ❌ **No** - Isolated from real data |
| Affect real tenant data | ❌ **No** - Actions limited to sandbox |
| Delete test data | ✅ Yes - Can reset sandbox state |
| Feature flag testing | ✅ Yes - Toggle features in sandbox |

### Key Features

### Test Tenant Creation

- Create synthetic tenant organizations
- Generate test users with fake email addresses
- Populate test data (campaigns, leads, emails)

### Feature Flag Testing

- Enable/disable beta features in sandbox
- Test feature variations
- Validate feature behavior before production

### Sandbox Data Reset

- Reset sandbox to clean state
- Clear all test data
- Restore default configuration

### Bug Reproduction

- Replicate production issues in sandbox
- Test bug fixes before deployment
- Document reproduction steps

### Routes

- `/dashboard/qa/*` - QA panel interface
- `/dashboard/qa/sandbox` - Sandbox management
- `/dashboard/qa/features` - Feature flag control
- `/dashboard/qa/test-data` - Test data generation

### Environment Separation

**Sandbox Environment**:

- Isolated PostgreSQL database (separate schema)
- Dedicated Redis instance
- Test SMTP servers (no real emails sent)
- Synthetic data only

**Production Environment**:

- QA role has **no access**
- Read-only view for monitoring (via shared dashboards)
- Cannot execute actions

### Related Documentation

- [Testing Strategy](/docs/implementation-technical/development-guidelines/testing-strategy) - QA processes
- [Sandbox Environment](/docs/implementation-technical/infrastructure/sandbox-environment) - Sandbox architecture
- [Feature Flags](/docs/operations/feature-management/flags) - Feature flag system

---

## Role Comparison Matrix

| Feature | super_admin | admin | support | qa | Tenant Owner |
|---------|-------------|-------|---------|----|--------------|\n| Platform-wide access | ✅ | ✅ | ✅ | ❌ | ❌ |
| Modify tenants | ✅ | ✅ | ❌ | ❌ | ❌ |
| View PII | ✅ | ✅ | ❌ | ⚠️ Synthetic only | ✅ Own tenant |
| System configuration | ✅ | ⚠️ Limited | ❌ | ❌ | ❌ |
| Production access | ✅ | ✅ | ✅ Read-only | ❌ | ✅ Own tenant |
| Sandbox access | ✅ | ✅ | ❌ | ✅ | ❌ |
| Financial data | ✅ | ⚠️ Limited | ❌ | ❌ | ✅ Own billing |

---

## Access Control Implementation

**Database Level**:

- Row-level security (RLS) policies enforce access restrictions
- Support role cannot query PII columns
- QA role limited to sandbox schema

**API Level**:

- Middleware checks role before executing actions
- Support endpoints return redacted PII
- QA endpoints only work in sandbox mode

**UI Level**:

- Role-based component rendering
- Sensitive fields hidden from support view
- QA panel only visible in sandbox environment

**Audit Logging**:

- All support access logged with timestamp, user, reason
- QA actions logged separately from production
- Support ticket ID linked to audit events

---

## Best Practices

### For Support Staff

1. **Always link actions to tickets**: Document why you're accessing tenant data
2. **Use internal notes**: Don't expose internal details to tenants
3. **Escalate when needed**: Transfer to super_admin for data modifications
4. **Respect PII boundaries**: Never screenshot or share protected data

### For QA Staff

1. **Never test in production**: Always use sandbox environment
2. **Reset after testing**: Clean up test data to keep sandbox usable
3. **Document reproductions**: Create detailed bug reports with steps
4. **Validate fixes**: Confirm bug fixes work before production deployment

---

[← Back to Operations](/docs/operations/)
