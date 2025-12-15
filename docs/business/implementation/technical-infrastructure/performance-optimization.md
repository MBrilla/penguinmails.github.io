---
title: "Performance Optimization"
description: "Server tuning, queue management, and capacity planning for email infrastructure"
last_modified_date: "2025-11-10"
level: "2"
persona: "Technical Teams, DevOps Engineers"
---

# Performance Optimization

Server performance tuning, email queue management, and capacity planning for high-volume email infrastructure.

## Server Performance Tuning

### Postfix Optimization

Optimize Postfix configuration for high-volume sending:

```bash
# /etc/postfix/main.cf optimizations
default_process_limit = 200
initial_destination_concurrency = 2
default_destination_concurrency_limit = 20
minimal_backoff_time = 300s
maximal_backoff_time = 3600s
default_process_limit = 50
default_destination_concurrency_limit = 5
```

### System-Level Optimizations

Kernel parameters for email servers:

```bash
# /etc/sysctl.conf
net.core.somaxconn = 1024
net.core.netdev_max_backlog = 3000
net.ipv4.tcp_max_syn_backlog = 1024
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
```

### Database Optimization

PostgreSQL performance tuning for email infrastructure:

```sql
-- PostgreSQL performance optimization
ALTER SYSTEM SET shared_buffers = '512MB';
ALTER SYSTEM SET effective_cache_size = '2GB';
ALTER SYSTEM SET max_connections = 200;
```

## Email Queue Management

### Queue Monitoring

Essential queue management commands:

```bash
# Monitor mail queue size
mailq | grep -c "^[A-F0-9]"

# Check queue status
postqueue -p

# Force queue processing
postqueue -f

# Remove specific messages
postsuper -d MESSAGE_ID
```

### Performance Metrics

Target metrics for healthy email infrastructure:

| Metric | Healthy Target | Warning Threshold | Critical Threshold |
|--------|---------------|-------------------|-------------------|
| **Queue Size** | <100 messages | 100-500 messages | >500 messages |
| **Processing Rate** | 100+ msg/min | 50-100 msg/min | <50 msg/min |
| **Delivery Time** | <5 min (95%) | 5-15 min | >15 min |
| **Bounce Rate** | <1% | 1-3% | >3% |
| **API Response** | <1 second | 1-3 seconds | >3 seconds |

## Load Testing and Capacity Planning

### Email Volume Testing

Load testing script for validating infrastructure capacity:

```bash
# Load testing script example
for i in {1..1000}; do
    (
        curl -X POST https://api.sendgrid.com/v3/mail/send \
          -H "Authorization: Bearer $API_KEY" \
          -H "Content-Type: application/json" \
          -d '{
            "personalizations": [{
              "to": [{"email": "test'$i'@example.com"}],
              "subject": "Load Test Email '$i'"
            }],
            "from": {"email": "noreply@example.com"},
            "content": [{"type": "text/plain", "value": "Test email number '$i'"}]
          }'
    ) &
done
wait
```

### Capacity Planning Guidelines

Resource allocation guidelines by component:

| Component | Capacity Rule | Example |
|-----------|--------------|---------|
| **Email Server** | 1,000 emails/minute per CPU core | 4 cores = 4,000/min |
| **Database** | 500 queries/second per CPU core | 2 cores = 1,000 qps |
| **Memory** | 1GB RAM per 10,000 active addresses | 50K = 5GB |
| **Storage** | 1GB per 1,000 emails stored | 100K = 100GB |
| **Bandwidth** | 1MB per 1,000 emails (with attachments) | 100K = 100MB |

### Scaling Strategy

Horizontal scaling approach for growing volumes:

1. **Single Server**: Up to 100K emails/month
2. **Two-Server**: Primary + backup for 100K-500K/month
3. **Load Balanced**: 3+ servers for 500K-1M/month
4. **Distributed**: Geographic distribution for 1M+/month

### Monitoring Infrastructure

Key monitoring tools and dashboards:

- **Prometheus/Grafana**: Real-time metrics visualization
- **CloudWatch**: AWS infrastructure monitoring
- **Datadog/New Relic**: Application performance monitoring
- **Nagios/Zabbix**: Infrastructure health checks
- **Custom Scripts**: Email-specific deliverability metrics

## Progressive Disclosure Navigation

### Strategic Context

- [Executive Summary](/docs/business/implementation/executive-summary) - High-level strategic findings
- [ROI Calculator](/docs/business/implementation/roi-calculator) - Cost-benefit analysis

### Operational Implementation

- [Cost Comparisons](/docs/business/implementation/cost-comparisons) - Complete TCO analysis
- [Personnel Analysis](/docs/business/implementation/personnel-analysis) - Team structure

### Complete Technical Analysis

- [Performance Benchmarks](/docs/business/implementation/performance-benchmarks) - Industry data
- [Compliance Framework](/docs/business/implementation/compliance-framework) - Technical compliance
- [Detailed Methodology](/docs/business/reference/methodology/overview) - Analysis methodology

## Related Documentation

- [Technical Infrastructure Overview](/docs/business/implementation/technical-infrastructure/overview) - Summary and navigation
- [VPS Provider Analysis](/docs/business/implementation/technical-infrastructure/vps-providers) - Provider specifications
- [Server Configuration](/docs/business/implementation/technical-infrastructure/server-configuration) - Implementation requirements
