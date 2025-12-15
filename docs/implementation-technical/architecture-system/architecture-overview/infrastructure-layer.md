---
title: "Infrastructure Layer"
last_modified_date: "2025-12-15"
level: "2"
persona: "Technical Leads, DevOps"
---

# Infrastructure Layer

VPS management, SMTP servers, DNS, and database architecture.

## VPS Management

### Provisioning Flow

```text
Customer Request → VPS Provisioning → SMTP Setup → DNS Configuration → Warm-up → Ready
      │                │                 │             │              │           │
   5 minutes        10 minutes        15 minutes    10 minutes    48 hours   Active
```

## SMTP Server Stack

| Component | Purpose |
|-----------|---------|
| **MailU Postfix** | Reliable email sending with anti-spam features |
| **Dovecot** | Secure email storage and retrieval |
| **Roundcube** | Web-based email access (optional) |
| **SpamAssassin** | Advanced spam filtering and reputation |

## DNS Configuration Automation

| Record Type | Purpose |
|-------------|---------|
| **SPF Records** | Email sending authorization |
| **DKIM Signatures** | Email integrity verification |
| **DMARC Policies** | Anti-spoofing protection |
| **MX Records** | Mail server routing |

## Database Architecture

### Primary Database (PostgreSQL)

| Data Type | Description |
|-----------|-------------|
| User Data | Authentication, profiles, preferences |
| Tenant Data | Multi-tenant isolation and configuration |
| Campaign Data | Email campaigns, contacts, analytics |
| Infrastructure Data | VPS instances, SMTP configurations |
| Compliance Data | Audit logs, consent records, unsubscribe lists |

### Cache Layer (Redis)

| Use Case | Description |
|----------|-------------|
| Session Storage | User sessions and authentication tokens |
| Real-time Data | Current campaign status, deliverability metrics |
| Rate Limiting | API rate limiting and abuse prevention |
| Queue Processing | Fast job queues for email processing |

## Hybrid Queue System

### Architecture

```text
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│ PostgreSQL  │────▶│   Queuer    │────▶│    Redis    │
│ (Durable)   │     │  (Process)  │     │  (Fast)     │
└─────────────┘     └─────────────┘     └──────┬──────┘
                                               │
                                        ┌──────▼──────┐
                                        │   Workers   │
                                        │ (Horizontal)│
                                        └─────────────┘
```

### Components

| Component | Role |
|-----------|------|
| **PostgreSQL** | Durable record of truth for job state and audit |
| **Redis** | Fast ephemeral queue for high-performance execution |
| **Queuer** | Migrates ready jobs from PostgreSQL to Redis |
| **Workers** | Horizontal scaling with Redis-based consumption |
| **Priority Queues** | Separate queues for high/normal/low priority |

---

[← Back to Architecture Overview](./README)
