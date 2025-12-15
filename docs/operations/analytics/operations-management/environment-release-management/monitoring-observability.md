---
title: "Monitoring & Observability"
description: Application metrics, logging strategy, and alerting system
last_modified_date: 2025-01-15
level: 3
persona: ["devops", "operations"]
keywords: [monitoring, logging, alerting, metrics, observability]
---

# Monitoring & Observability

Application monitoring, structured logging, and alerting configuration.

## Application Metrics

```typescript
interface ApplicationMetrics {
  performance: {
    responseTime: Histogram;
    throughput: Counter;
    errorRate: Rate;
    memoryUsage: Gauge;
    cpuUsage: Gauge;
  };
  business: {
    activeUsers: Gauge;
    campaignSent: Counter;
    apiCalls: Counter;
    revenue: Counter;
  };
  infrastructure: {
    diskUsage: Gauge;
    networkIO: Counter;
    databaseConnections: Gauge;
    queueLength: Gauge;
  };
}
```

## Logging Strategy

```typescript
enum LogLevel {
  ERROR = 0,
  WARN = 1,
  INFO = 2,
  DEBUG = 3,
  TRACE = 4
}

interface LogEntry {
  timestamp: Date;
  level: LogLevel;
  service: string;
  message: string;
  context: {
    userId?: string;
    requestId?: string;
    sessionId?: string;
    environment: string;
    version: string;
  };
  metadata: Record<string, any>;
}

// Structured logging implementation
const logger = {
  error: (message: string, context?: Partial<LogEntry['context']>, metadata?: any) => {
    log(LogLevel.ERROR, message, context, metadata);
  },
  warn: (message: string, context?: Partial<LogEntry['context']>, metadata?: any) => {
    log(LogLevel.WARN, message, context, metadata);
  },
  info: (message: string, context?: Partial<LogEntry['context']>, metadata?: any) => {
    log(LogLevel.INFO, message, context, metadata);
  },
  debug: (message: string, context?: Partial<LogEntry['context']>, metadata?: any) => {
    log(LogLevel.DEBUG, message, context, metadata);
  }
};
```

## Alerting System

```typescript
interface AlertRule {
  name: string;
  description: string;
  condition: string;  // PromQL or similar query
  severity: 'critical' | 'warning' | 'info';
  channels: NotificationChannel[];
  cooldown: number;   // minutes between alerts
  enabled: boolean;
}

const alertRules: AlertRule[] = [
  {
    name: 'High Error Rate',
    description: 'API error rate above 5%',
    condition: 'rate(http_requests_total{status=~"5.."}[5m]) > 0.05',
    severity: 'critical',
    channels: ['slack', 'email', 'sms'],
    cooldown: 5,
    enabled: true
  },
  {
    name: 'Slow Response Time',
    description: '95th percentile response time above 2 seconds',
    condition: 'histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m])) > 2',
    severity: 'warning',
    channels: ['slack'],
    cooldown: 15,
    enabled: true
  }
];
```

## Alert Severity Matrix

| Severity | Response Time | Escalation | Channels |
|----------|--------------|------------|----------|
| Critical | Immediate | Auto-escalate after 5 min | Slack, Email, SMS, PagerDuty |
| Warning | 30 minutes | Manual escalation | Slack, Email |
| Info | Next business day | No escalation | Slack |

## Key Metrics Targets

| Metric | Target | Alert Threshold |
|--------|--------|-----------------|
| Response Time (p95) | < 500ms | > 2s |
| Error Rate | < 0.1% | > 5% |
| Uptime | 99.9% | < 99.5% |
| CPU Usage | < 70% | > 90% |
| Memory Usage | < 80% | > 95% |
