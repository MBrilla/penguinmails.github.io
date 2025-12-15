---
title: "Hostwinds Cloud API Integration"
description: "Overview of Hostwinds VPS and infrastructure API integration for PenguinMails server management"
last_modified_date: "2025-11-19"
level: "2"
persona: "Documentation Users"
---

# Hostwinds API Overview

**For Platform Engineers and DevOps Teams**: Strategic overview of Hostwinds API integration for VPS provisioning, IP management, and infrastructure automation

**Technical Authority**: Official Hostwinds Cloud API documentation is available at [https://developers.hostwinds.com/cloud/](https://developers.hostwinds.com/cloud/)

---

> [!WARNING]
> **API Limitations**: This documentation currently lacks information for required Hostwinds APIs regarding **VPS management**, **secondary IP**, **monitoring**, and **billing**. Refer to the official Hostwinds documentation for complete coverage.
> [!NOTE]
> **Architecture Context**: Hostwinds is an **external service** integrated into PenguinMails' infrastructure. For a complete overview of how it fits with our internal servers, see **[API Architecture Overview](/docs/implementation-technical/api/)**.

---

## Purpose and Scope

The Hostwinds Cloud API is PenguinMails' primary infrastructure provider for:

- **VPS Instance Provisioning**: Automated server creation and management

- **IP Address Management**: Dedicated IP allocation and configuration

- **Network Configuration**: DNS, firewall, and networking setup

- **Cost Management**: Billing cycle management and pricing queries

This API integration enables PenguinMails to:

1. **Automate Infrastructure**: Provision VPS instances on-demand for tenant workloads

2. **Manage Costs**: Track and optimize infrastructure spending through pricing APIs

3. **Ensure Reliability**: Monitor and maintain server health and performance

4. **Scale Efficiently**: Add capacity as tenant demand grows

---

## Integration Architecture

### Hostwinds as External Infrastructure Provider

**Status**: **External Service** (3rd-party API, not part of PenguinMails codebase)

Hostwinds provides the physical VPS infrastructure that powers our SMTP sending capabilities. Our internal servers consume the Hostwinds API to provision and manage this infrastructure.

### Which PenguinMails Servers Use Hostwinds API

| Internal Server | Uses Hostwinds For | Example Operations |
|-----------------|-------------------|-------------------|
| **`smtp-server`** | IP availability checks | Before assigning IP to tenant, verify VPS is active |
| **`queue-server`** | VPS health validation | Before processing send job, ensure tenant's VPS is up |
| **`platform-api`** | Admin dashboard data | Show VPS status in admin panel (up/down/provisioning) |
| **`api` (tenant)** | Tenant provisioning | Create new VPS when tenant signs up |
| **Operations scripts** | Maintenance & billing | Automated VPS upgrades, cost reconciliation |

### Architecture Diagram

```mermaid
graph TB
    subgraph "PenguinMails Internal Services"
        WEB[web: Next.js Frontend]
        API[api: Tenant API]
        SMTP[smtp-server]
        QUEUE[queue-server]
        PLATFORM[platform-api]
    end

    subgraph "External Services"
        HOSTWINDS[Hostwinds Cloud API]
    end

    subgraph "Hostwinds Infrastructure"
        VPS[VPS Instances]
        IPS[Dedicated IPs]
        NET[Network Config]
    end

    API -->|Provision VPS| HOSTWINDS
    SMTP -->|Check IP availability| HOSTWINDS
    QUEUE -->|Validate VPS health| HOSTWINDS
    PLATFORM -->|Get VPS status| HOSTWINDS

    HOSTWINDS -->|Manages| VPS
    HOSTWINDS -->|Allocates| IPS
    HOSTWINDS -->|Configures| NET

    VPS -->|Runs| MAILU[Mailu SMTP Stack]
    IPS -->|Used by| MAILU
    NET -->|Supports| MAILU

    style HOSTWINDS fill:#fff3e0
    style WEB fill:#e1f5fe
    style API fill:#e1f5fe
    style SMTP fill:#e1f5fe
    style QUEUE fill:#e1f5fe
    style PLATFORM fill:#e1f5fe
    style MAILU fill:#c8e6c9


```

### Key Integration Points

1. **Database Schema Integration**

   - `vps_instances.hostwinds_instance_id` - Links to Hostwinds server UUID

   - `vps_instances.approximate_cost` - Populated from Hostwinds pricing APIs

   - `smtp_ip_addresses.approximate_cost` - IP cost modeling

2. **Cost Modeling**

   - See [Infrastructure Overview](/docs/business/implementation/infrastructure-overview) for canonical cost model

   - Baseline: Hostwinds Unmanaged Linux VPS 1 CPU / 2 GB / 50 GB / 2 TB at **$9.99/month**

   - IP pricing: **$4.99/month per dedicated IP** (configurable constant)

3. **Automation Workflows**

   - Tenant onboarding → VPS provisioning → Mailu deployment

   - IP allocation → DNS configuration → warmup process

   - Billing reconciliation → cost tracking → margin analysis

### Example: Queue Server Usage

```typescript
// apps/queue-server/src/workers/email-sender.ts
import { HostwindsClient } from '@penguinmails/hostwinds-client';
import { db } from '@penguinmails/database';

async function processEmailSendJob(job) {
  const { tenantId, campaignId } = job.data;

  // 1. Get tenant's VPS info from DB
  const vps = await db.vps_instances.findUnique({
    where: { tenant_id: tenantId }
  });

  // 2. Check VPS health via Hostwinds API (external call)
  const hostwinds = new HostwindsClient();
  const vpsStatus = await hostwinds.getServerStatus(vps.hostwinds_instance_id);

  // 3. Abort if VPS is down
  if (vpsStatus !== 'active') {
    throw new Error(`Tenant VPS is ${vpsStatus}, cannot send emails`);
  }

  // 4. Proceed with sending...
}


```

---

## API Overview

### Base Configuration

- **Base URL**: `https://clients.hostwinds.com/cloud/api.php`

- **Protocol**: HTTP POST (all requests)

- **Data Format**: `application/x-www-form-urlencoded`

- **Authentication**: API Key (`API` field) in POST body

### Error Response Format

| Code | Status | Content | Notes |
| :---- | :---- | :---- | :---- |
| **200 OK** | "error" | `[ { "result": "error", "action": "...", "message": "<error_detail>", "ERROR": "<code-optional>" } ]` | **Critical:** Check the result field for "error". |
| **401/403** | N/A | `[ { "result": "error", "message": "Not Authorized!" } ]` | Authentication or permission failure. |

> [!IMPORTANT]
> Always check the `result` field in the response. A 200 status code does not guarantee success - the response body may contain `"result": "error"`.

---

## API Functional Areas

### 1. Server Instance Management

Endpoints for creating, managing, and monitoring VPS instances.

**Key Operations**:

- Create and provision new servers

- Retrieve server details and status

- Start, stop, reboot instances

- Rebuild and repair servers

- Manage rescue mode

**Documentation**: [Hostwinds Server Management API](/docs/implementation-technical/api/hostwinds/server-management )

**Common Use Cases**:

- Tenant onboarding: Create VPS for new tenant

- Maintenance: Reboot servers for updates

- Troubleshooting: Enter rescue mode for diagnostics

---

### 2. Networking and IP Management

Endpoints for managing IP addresses, networking, and firewall configuration.

**Key Operations**:

- Allocate and manage dedicated IPs

- Configure primary and secondary IPs

- Set reverse DNS (rDNS)

- Manage firewall profiles

- Regenerate network configuration

**Documentation**: [Hostwinds Networking API](/docs/implementation-technical/api/hostwinds/networking )

**Common Use Cases**:

- IP allocation: Add dedicated IP for tenant

- DNS configuration: Set rDNS for deliverability

- Network troubleshooting: Regenerate networking

---

### 3. Upgrade, Billing, and Pricing

Endpoints for managing server upgrades, billing cycles, and pricing queries.

**Key Operations**:

- Upgrade server resources (CPU, RAM, disk)

- Change billing cycles

- Query pricing and plans

- Manage service levels (managed/unmanaged)

- Create upgrade orders and invoices

**Documentation**: [Hostwinds Upgrade & Billing API](/docs/implementation-technical/api/hostwinds/upgrade-billing )

**Common Use Cases**:

- Capacity planning: Query available plans and pricing

- Resource scaling: Upgrade server for growing tenant

- Cost optimization: Change billing cycle for savings

---

### 4. Monitoring and Diagnostics

Endpoints for monitoring server health, logs, and performance.

**Key Operations**:

- Retrieve server logs and serial console

- Get server performance charts

- Check maintenance events

- Validate service IDs

- Monitor SMTP filters

**Documentation**: [Hostwinds Monitoring API](/docs/implementation-technical/api/hostwinds/monitoring )

**Common Use Cases**:

- Troubleshooting: Access serial console logs

- Performance monitoring: Retrieve server metrics

- Maintenance planning: Check scheduled events

---

### 5. Automation Best Practices

Recommendations for reliable, secure, and efficient automation.

**Key Topics**:

- Error handling and retry strategies

- Asynchronous operation polling

- Security and credential management

- Rate limiting and efficiency

- Integration patterns

**Documentation**: [Hostwinds Automation Best Practices](/docs/implementation-technical/api/hostwinds/automation-best-practices )

**Critical for**:

- Production automation scripts

- CI/CD pipeline integration

- Monitoring and alerting systems

---

## Related Documentation

### Infrastructure Planning

- [Infrastructure Overview](/docs/business/implementation/infrastructure-overview) - Cost modeling and provider roles

- [Database Infrastructure](/docs/implementation-technical/database-infrastructure/operations/database-migration-guide) - Schema integration and cost fields

### API Documentation

- [Central SMTP API](/docs/implementation-technical/api/central-smtp) - Internal SMTP infrastructure management

- [Executive API](/docs/implementation-technical/api/platform-api) - High-level business metrics and reporting

### Implementation Guides

- [Architecture Overview](/docs/implementation-technical/architecture-system/architecture-overview) - System architecture and integration points

- [Development Standards](/docs/implementation-technical/development-guidelines/README) - API integration best practices

---

## Getting Started

### For Platform Engineers

1. **Review API Authentication**: Understand API key management and security

2. **Study Server Management**: Learn instance lifecycle and operations

3. **Understand Cost Model**: Review pricing APIs and cost tracking

4. **Implement Error Handling**: Follow automation best practices

### For DevOps Teams

1. **Automation Setup**: Integrate Hostwinds API into deployment pipelines

2. **Monitoring Integration**: Set up health checks and alerting

3. **Cost Tracking**: Implement billing reconciliation workflows

4. **Disaster Recovery**: Plan for instance backup and restoration

### For Finance Teams

1. **Cost Modeling**: Understand `approximate_cost` field population

2. **Pricing Queries**: Use pricing APIs for budget planning

3. **Reconciliation**: Compare modeled costs with Hostwinds invoices

4. **Margin Analysis**: Track infrastructure costs per tenant

---

**Keywords**: Hostwinds API, VPS provisioning, infrastructure automation, IP management, cloud infrastructure, cost modeling, server management

---

*This API overview is part of the comprehensive Implementation Technical documentation. It provides strategic guidance for Hostwinds API integration and infrastructure automation.*
