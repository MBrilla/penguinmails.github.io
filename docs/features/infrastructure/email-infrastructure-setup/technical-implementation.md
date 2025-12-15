---
title: "Technical Implementation"
last_modified_date: "2025-12-15"
level: "3"
persona: "Developers"
---

# Technical Implementation

Database schemas, API endpoints, and background job architecture for email infrastructure.

## System Architecture

```text
┌─────────────────────────────────────────────────────────┐
│                    PenguinMails Core                     │
├─────────────────────────────────────────────────────────┤
│  Workspace Manager  │  VPS Orchestrator  │  DNS Manager  │
├─────────────────────────────────────────────────────────┤
│                 Infrastructure Layer                     │
│   ┌─────────┐    ┌─────────┐    ┌─────────┐            │
│   │  VPS 1  │    │  VPS 2  │    │  VPS N  │            │
│   │ (Hetzn) │    │ (Hostw) │    │ (Other) │            │
│   └─────────┘    └─────────┘    └─────────┘            │
└─────────────────────────────────────────────────────────┘
```

## Database Schema

### Inboxes Table

```sql
CREATE TABLE inboxes (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    workspace_id UUID NOT NULL REFERENCES workspaces(id),
    email_address VARCHAR(255) NOT NULL UNIQUE,
    display_name VARCHAR(255),
    vps_id UUID REFERENCES vps_servers(id),
    smtp_config JSONB,
    imap_config JSONB,
    warmup_enabled BOOLEAN DEFAULT false,
    warmup_status VARCHAR(50) DEFAULT 'not_started',
    daily_send_limit INTEGER DEFAULT 50,
    reputation_score DECIMAL(3,2) DEFAULT 0.00,
    status VARCHAR(50) DEFAULT 'pending_setup',
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_inboxes_tenant ON inboxes(tenant_id);
CREATE INDEX idx_inboxes_workspace ON inboxes(workspace_id);
CREATE INDEX idx_inboxes_status ON inboxes(status);
```

### VPS Servers Table

```sql
CREATE TABLE vps_servers (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    provider VARCHAR(50) NOT NULL, -- 'hetzner', 'hostwinds'
    provider_server_id VARCHAR(255),
    ip_address INET NOT NULL,
    hostname VARCHAR(255) NOT NULL,
    specs JSONB, -- {cpu, ram, disk, region}
    status VARCHAR(50) DEFAULT 'provisioning',
    mail_server_status VARCHAR(50) DEFAULT 'not_configured',
    ssl_certificate_expires TIMESTAMPTZ,
    monitoring_enabled BOOLEAN DEFAULT true,
    monthly_cost DECIMAL(10,2),
    created_at TIMESTAMPTZ DEFAULT NOW()
);
```

### DNS Records Table

```sql
CREATE TABLE dns_records (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    inbox_id UUID NOT NULL REFERENCES inboxes(id),
    domain VARCHAR(255) NOT NULL,
    record_type VARCHAR(10) NOT NULL, -- 'A', 'MX', 'TXT', 'CNAME'
    record_name VARCHAR(255) NOT NULL,
    record_value TEXT NOT NULL,
    ttl INTEGER DEFAULT 3600,
    verification_status VARCHAR(50) DEFAULT 'pending',
    last_verified_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_dns_inbox ON dns_records(inbox_id);
CREATE INDEX idx_dns_domain ON dns_records(domain);
```

## API Endpoints

### Inbox Management

```typescript
// Create new inbox
POST /api/v1/workspaces/:workspaceId/inboxes
{
  "email_address": "sales@yourdomain.com",
  "display_name": "Sales Team",
  "vps_provider": "hetzner",
  "vps_size": "professional"
}

// Get inbox status
GET /api/v1/inboxes/:inboxId

// Update inbox configuration
PATCH /api/v1/inboxes/:inboxId
{
  "daily_send_limit": 100,
  "warmup_enabled": true
}

// Delete inbox
DELETE /api/v1/inboxes/:inboxId
```

### DNS Management

```typescript
// Get required DNS records
GET /api/v1/inboxes/:inboxId/dns-records

// Verify DNS configuration
POST /api/v1/inboxes/:inboxId/dns-records/verify

// Response example
{
  "records": [
    {
      "type": "TXT",
      "name": "_dmarc",
      "value": "v=DMARC1; p=none; rua=mailto:dmarc@yourdomain.com",
      "status": "verified"
    },
    {
      "type": "TXT", 
      "name": "@",
      "value": "v=spf1 ip4:123.45.67.89 ~all",
      "status": "pending"
    }
  ],
  "all_verified": false
}
```

### VPS Operations

```typescript
// Provision new VPS
POST /api/v1/workspaces/:workspaceId/vps
{
  "provider": "hetzner",
  "size": "professional",
  "region": "eu-central"
}

// Get VPS status
GET /api/v1/vps/:vpsId

// Restart mail server
POST /api/v1/vps/:vpsId/mail-server/restart

// View server logs
GET /api/v1/vps/:vpsId/logs?lines=100
```

## Background Jobs

### VPS Provisioning Pipeline

```typescript
// Job: vps.provision
class VPSProvisionJob {
  async perform(workspaceId: string, config: VPSConfig) {
    // 1. Create VPS via provider API
    const server = await this.createServer(config);
    
    // 2. Wait for server ready
    await this.waitForServerReady(server.id);
    
    // 3. Install mail server (Stalwart)
    await this.installMailServer(server);
    
    // 4. Configure SSL certificate
    await this.configureSsl(server);
    
    // 5. Update database
    await this.updateServerStatus(server.id, 'ready');
    
    // 6. Trigger DNS record generation
    await this.enqueueJob('dns.generate', { serverId: server.id });
  }
}
```

### DNS Verification Job

```typescript
// Job: dns.verify (runs every 5 minutes for pending records)
class DNSVerifyJob {
  async perform(inboxId: string) {
    const records = await this.getPendingRecords(inboxId);
    
    for (const record of records) {
      const verified = await this.verifyDNSRecord(record);
      if (verified) {
        await this.updateRecordStatus(record.id, 'verified');
      }
    }
    
    // Check if all records verified
    const allVerified = await this.checkAllRecordsVerified(inboxId);
    if (allVerified) {
      await this.updateInboxStatus(inboxId, 'ready');
      await this.notifyUser(inboxId, 'dns_verified');
    }
  }
}
```

### Health Monitoring Job

```typescript
// Job: monitoring.health (runs every 5 minutes)
class HealthMonitorJob {
  async perform(vpsId: string) {
    const checks = [
      this.checkServerReachable(vpsId),
      this.checkMailServerRunning(vpsId),
      this.checkSslValid(vpsId),
      this.checkDiskSpace(vpsId),
      this.checkQueueHealth(vpsId)
    ];
    
    const results = await Promise.all(checks);
    
    // Store metrics
    await this.storeHealthMetrics(vpsId, results);
    
    // Alert on failures
    const failures = results.filter(r => !r.healthy);
    if (failures.length > 0) {
      await this.triggerAlert(vpsId, failures);
    }
  }
}
```

## Provider Integration Examples

### Hetzner API Integration

```typescript
import { HetznerClient } from '@penguinmails/hetzner';

async function provisionHetznerVPS(config: VPSConfig) {
  const client = new HetznerClient(process.env.HETZNER_API_KEY);
  
  const server = await client.servers.create({
    name: `pm-${config.workspaceId.slice(0, 8)}`,
    server_type: mapSizeToHetznerType(config.size),
    location: config.region,
    image: 'ubuntu-22.04',
    ssh_keys: [config.sshKeyId],
    labels: {
      workspace: config.workspaceId,
      tenant: config.tenantId
    }
  });
  
  return server;
}
```

### Hostwinds API Integration

```typescript
import { HostwindsClient } from '@penguinmails/hostwinds';

async function provisionHostwindsVPS(config: VPSConfig) {
  const client = new HostwindsClient({
    apiKey: process.env.HOSTWINDS_API_KEY,
    apiSecret: process.env.HOSTWINDS_API_SECRET
  });
  
  const server = await client.createServer({
    package: mapSizeToHostwindsPackage(config.size),
    location: config.region,
    os: 'ubuntu-22.04',
    hostname: `mail-${config.workspaceId.slice(0, 8)}.penguinmails.io`
  });
  
  return server;
}
```

---

[← Back to Email Infrastructure Setup](./README)
