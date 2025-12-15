---
title: "System Monitoring Routes"
description: "Admin routes for queue monitoring, logs, and infrastructure health"
last_modified_date: "2025-12-16"
level: "2"
persona: "Technical Teams"
---

# System Monitoring Routes

## `/dashboard/system/queues` - Background Job Monitoring

**User Story**: *"As an ops engineer, I want to monitor the hybrid PostgreSQL + Redis job queues for failures and delays, so I can ensure system reliability."*

### Queue Dashboard

#### Custom UI for Hybrid Queue System

- **Queue List** (PostgreSQL + Redis monitoring):
  - Columns: Queue Name, Active Jobs, Waiting, Completed (last hour), Failed, Actions.
  - **Queue Names**: `queue:email-sending:high`, `queue:email-sending`, `queue:analytics:daily-aggregate`, etc.
- **Per-Queue Actions**:
  - **View Details**: Shows job list.
  - **Pause Queue**: Emergency stop.
  - **Resume Queue**.

### Job Details (Click on failed job)

- **JSON Viewer**:
  - Shows job data (e.g., campaign ID, recipient list).
  - Error stack trace.
- **Actions**:
  - **Retry Job**: Re-queues the job.
  - **Delete Job**: Removes from queue.

**Related Documentation**:
- [Queue System Implementation](/docs/technical/architecture/detailed-technical/queue-system-implementation)
- [Incident Response](/docs/operations/incident-management/runbooks)

**Technical Integration**:
- **Hybrid Architecture**: PostgreSQL for durable state + Redis for fast processing.
- **Monitoring**: Real-time queue depth tracking and job status monitoring.
- **Alerting**: PagerDuty integration for critical failures.

---

## `/dashboard/system/infrastructure` - System Health Monitoring

**User Story**: *"As an ops engineer, I want a real-time view of server health and IP reputation, so I can detect and resolve issues before they impact users."*

### Service Health Grid

- **Status Cards** (Traffic light):
  - **API Server**: Healthy (Response time: 120ms).
  - **SMTP Service**: Degraded (Queue backlog: 5,000 emails).
  - **Database (OLTP)**: Healthy.
  - **Database (OLAP)**: Healthy.

### IP Reputation Monitor

- **Table**: IP Address, Provider, Reputation Score, Blacklists, Daily Volume, Status.
- **Alerts**: "IP 203.0.113.12 listed on Spamhaus (detected 2 hours ago)".

### Infrastructure Alerts

- **Alert Feed** (Last 24 hours):
  - Timestamp, Severity, Service, Message.
  - Example: "ERROR | SMTP | Queue size exceeded 10k threshold".

**Related Documentation**:
- [Infrastructure Overview](/docs/technical/infrastructure/server-architecture)
- [Monitoring Setup](/docs/operations/monitoring/prometheus-grafana)

**Technical Integration**:
- **Prometheus + Grafana**: Metrics collection and visualization (Future/2026 Spike).
- **ClickHouse**: Stores historical metrics (Future/2026 Spike).
- **PagerDuty**: Alerts for critical issues.

---

## `/dashboard/system/logs` - System Log Viewer

**User Story**: *"As a developer debugging a production issue, I want to search and filter application logs, so I can trace errors to their source."*

### Log Search Interface

- **Search Bar**:
  - Placeholder: "Search logs (e.g., errors, user email, request ID)".
  - Supports regex.
- **Filters** (Sidebar):
  - **Level**: Error, Warning, Info, Debug.
  - **Service**: API, SMTP, Queue, Web.
  - **Time Range**: Last hour, Last 24 hours, Custom.

### Log Results

- **Table or List View**:
  - Columns: Timestamp, Level, Service, Message, User/Request ID.
  - **Color Coding**: Errors (Red), Warnings (Yellow).
- **Expandable Rows**:
  - Click to see full log entry with metadata (stack trace, context).

### Export

- **"Export to CSV" Button**: For further analysis.

**Related Documentation**:
- [Logging Standards](/docs/technical/observability/logging)
- [Error Tracking](/docs/operations/monitoring/sentry-integration)

**Technical Integration**:
- **Elasticsearch**: Centralized log aggregation (Future/2026 Spike).
- **Sentry**: Error tracking and alerting (Future/2026 Spike).
