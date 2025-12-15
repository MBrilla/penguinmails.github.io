---
title: "Monitoring and Health"
last_modified_date: "2025-12-15"
level: "2"
persona: "All Users"
---

# Monitoring and Health

Real-time monitoring dashboards, alerts, and health metrics for email infrastructure.

## Dashboard Overview

The Infrastructure Dashboard displays:

- **Server Status**: Real-time health of all VPS servers
- **Inbox Status**: Configuration state of each inbox
- **DNS Verification**: Record verification progress
- **Sending Metrics**: Daily volume and success rates
- **Reputation Scores**: Per-inbox reputation tracking

## Health Metrics

### Server Health Indicators

| Metric | Healthy | Warning | Critical |
|--------|---------|---------|----------|
| CPU Usage | < 60% | 60-80% | > 80% |
| Memory | < 70% | 70-85% | > 85% |
| Disk Space | < 70% | 70-90% | > 90% |
| Mail Queue | < 100 | 100-500 | > 500 |
| SSL Expiry | > 30 days | 7-30 days | < 7 days |

### Inbox Health Indicators

| Metric | Healthy | Warning | Critical |
|--------|---------|---------|----------|
| Bounce Rate | < 2% | 2-5% | > 5% |
| Spam Reports | < 0.1% | 0.1-0.3% | > 0.3% |
| Reply Rate | > 5% | 2-5% | < 2% |
| DNS Records | All verified | Partial | Failed |

## Alert Configuration

### Default Alert Rules

```yaml
alerts:
  - name: "Server Unreachable"
    condition: "server.ping_failed for 5m"
    severity: critical
    channels: [email, slack, sms]
    
  - name: "High Bounce Rate"
    condition: "inbox.bounce_rate > 5%"
    severity: warning
    channels: [email, slack]
    
  - name: "SSL Expiring Soon"
    condition: "server.ssl_expires_in < 7d"
    severity: warning
    channels: [email]
    
  - name: "Mail Queue Backlog"
    condition: "server.queue_size > 500 for 15m"
    severity: warning
    channels: [slack]
```

### Notification Channels

- **Email**: Default for all alerts
- **Slack**: Real-time team notifications
- **SMS**: Critical alerts only (Enterprise)
- **Webhooks**: Custom integrations (Enterprise)

## Troubleshooting

### Common Issues

**Server Shows "Unhealthy"**:
1. Check network connectivity to server
2. Verify SSH access is working
3. Check disk space and memory usage
4. Review mail server logs

**DNS Records Not Verifying**:
1. Confirm records added to DNS provider
2. Wait for TTL propagation (up to 24-48 hours)
3. Use MXToolbox to verify external visibility
4. Check for typos in record values

**High Bounce Rate**:
1. Review bounce reasons in logs
2. Clean invalid addresses from lists
3. Check if domain is on any blocklists
4. Verify SPF/DKIM/DMARC are configured

**Mail Queue Growing**:
1. Check target servers are accepting mail
2. Review deferrals for patterns
3. Verify rate limits aren't exceeded
4. Check for authentication issues

---

[← Back to Email Infrastructure Setup](./README)
