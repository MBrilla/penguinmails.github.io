---
title: "Data Flow"
last_modified_date: "2025-12-15"
level: "2"
persona: "Technical Leads, Architects"
---

# Data Flow

Request routing, email processing, and analytics pipelines.

## Request Flow

```text
┌──────────┐     ┌───────────┐     ┌──────────────┐
│  Client  │────▶│ CloudFlare│────▶│  API Gateway │
└──────────┘     └───────────┘     └──────┬───────┘
                                          │
                    ┌─────────────────────┼─────────────────────┐
                    │                     │                     │
              ┌─────▼─────┐         ┌─────▼─────┐         ┌─────▼─────┐
              │   Auth    │         │  Campaign │         │  Infra    │
              │  Service  │         │  Service  │         │  Service  │
              └─────┬─────┘         └─────┬─────┘         └─────┬─────┘
                    │                     │                     │
                    └─────────────────────┼─────────────────────┘
                                          │
                                    ┌─────▼─────┐
                                    │ PostgreSQL│
                                    │   + Redis │
                                    └───────────┘
```

## Email Processing Pipeline

### Outbound Flow

```text
1. Campaign Created
   │
2. Contacts Segmented
   │
3. Jobs Queued (PostgreSQL)
   │
4. Queuer Moves to Redis
   │
5. Workers Process
   │
6. SMTP Server Sends
   │
7. Delivery Status Updated
   │
8. Analytics Recorded
```

### Inbound Flow

```text
1. Email Received (SMTP)
   │
2. Spam Check
   │
3. Route to Mailbox
   │
4. Reply Detection
   │
5. Thread Matching
   │
6. Notification Sent
   │
7. Analytics Updated
```

## Analytics Pipeline

### Real-time Metrics

```text
Event Occurs → Redis Pub/Sub → WebSocket → Dashboard
     │
     └──────▶ PostgreSQL (Audit Log)
```

### Aggregated Analytics

```text
Raw Events (PostgreSQL)
        │
        ▼
Aggregation Jobs (Hourly)
        │
        ▼
OLAP Tables (TimescaleDB)
        │
        ▼
Dashboard Queries
```

## Queue Architecture

### Priority Levels

| Priority | Use Case | SLA |
|----------|----------|-----|
| High | Transactional emails | < 30 seconds |
| Normal | Campaign emails | < 5 minutes |
| Low | Analytics, cleanup | < 1 hour |

### Job States

```text
┌─────────┐     ┌─────────┐     ┌─────────┐
│ pending │────▶│ queued  │────▶│ active  │
└─────────┘     └─────────┘     └────┬────┘
                                     │
                    ┌────────────────┼────────────────┐
                    │                │                │
              ┌─────▼─────┐    ┌─────▼─────┐    ┌─────▼─────┐
              │ completed │    │  failed   │    │  retry    │
              └───────────┘    └───────────┘    └───────────┘
```

---

[← Back to Architecture Overview](./README)
