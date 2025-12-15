---
title: "IP Management and Reputation"
description: "IP allocation strategy, reputation management, and monitoring for email infrastructure"
last_modified_date: "2025-11-10"
level: "2"
persona: "Technical Teams, Deliverability Engineers"
---

# IP Management and Reputation

IP allocation strategy, reputation monitoring, and management techniques for maintaining email deliverability.

## IP Allocation Strategy

Cold email operations require dedicated IPs with proper warmup protocols. Volume-based allocation prevents reputation damage from concentrated sending patterns.

### Small Volume (1K-10K)

- **Primary IP**: 1 dedicated IP
- **Backup IP**: 1 additional IP for rotation
- **Warmup Period**: 2-3 weeks gradual ramp-up
- **Daily Send Limit**: 10 emails/day initial, 10-20% daily increase

### Medium Volume (10K-100K)

- **Primary IPs**: 3-5 dedicated IPs
- **Rotation Strategy**: Round-robin with reputation-based routing
- **Warmup Period**: 3-4 weeks per domain
- **Daily Send Limit**: 50-100 emails/day per IP

### Large Volume (100K+)

- **Primary IPs**: 10-20+ dedicated IPs
- **Load Balancing**: Geographic and reputation-based distribution
- **Warmup Period**: 4-6 weeks for new domains
- **Daily Send Limit**: 1,000+ emails/day per IP

## Reputation Management Technical Implementation

### Monitoring Setup

Blacklist monitoring ensures rapid detection and response to listing events:

```bash
#!/bin/bash
# Check RBL status for all IPs

for ip in $(cat /etc/email); do
    for rbl in zen.spamhaus.org bl.spamcop.net; do
        if dig +short $ip.$rbl; then
            echo "IP $ip listed in $rbl"
        fi
    done
done
```

### Deliverability Monitoring

Key metrics for ongoing reputation management:

- **Open Rate Tracking**: ESP APIs for engagement data
- **Bounce Rate Monitoring**: Real-time bounce processing
- **Complaint Rate Tracking**: Feedback loop integration
- **Domain Health**: SPF/DKIM/DMARC validation

### ISP Feedback Loops

Register for feedback loops with major ISPs:

| ISP | Feedback Loop URL | Response Time |
|-----|-------------------|---------------|
| **Gmail** | Postmaster Tools | Real-time |
| **Microsoft** | SNDS / JMRP | 24-48 hours |
| **Yahoo** | Complaint Feedback Loop | 24 hours |
| **AOL** | Feedback Loop Program | 24 hours |
| **Comcast** | Feedback Loop | 24-48 hours |

### Blacklist Prevention

Proactive measures to prevent IP blacklisting:

1. **Maintain list hygiene**: Remove bounces and complaints immediately
2. **Monitor engagement**: Suppress inactive addresses after 90 days
3. **Warmup properly**: Never skip warmup for new IPs or domains
4. **Authenticate properly**: Ensure SPF/DKIM/DMARC pass consistently
5. **Respect rate limits**: Honor ISP throttling signals

### Blacklist Recovery

When blacklisting occurs:

1. **Identify the cause**: Review recent sending patterns and complaints
2. **Stop sending**: Pause campaigns from affected IP
3. **Request delisting**: Follow RBL-specific procedures
4. **Remediate issues**: Fix underlying list or content problems
5. **Resume gradually**: Restart with low volume and monitor closely

## IP Warmup Schedule

### Week 1-2: Foundation

| Day | Daily Volume | Cumulative | Notes |
|-----|--------------|------------|-------|
| 1 | 10 | 10 | Test deliverability |
| 2 | 15 | 25 | Monitor bounces |
| 3 | 25 | 50 | Check inbox placement |
| 4 | 40 | 90 | Review engagement |
| 5 | 60 | 150 | Adjust if needed |
| 6-7 | 80 | 310 | Weekend pause optional |
| 8-14 | 100-200 | 1,500+ | Gradual increase |

### Week 3-4: Scaling

- Increase by 15-25% daily
- Target 500-1,000 emails/day by end of week 3
- Monitor complaint rates (target <0.1%)
- Check spam folder placement regularly

### Week 5-6: Maturation

- Reach target sending volume
- Establish consistent sending patterns
- Monitor long-term reputation metrics
- Adjust based on ISP-specific feedback

## Related Documentation

- [Technical Infrastructure Overview](/docs/business/implementation/technical-infrastructure/overview) - Summary and navigation
- [Server Configuration](/docs/business/implementation/technical-infrastructure/server-configuration) - Implementation requirements
- [Performance Optimization](/docs/business/implementation/technical-infrastructure/performance-optimization) - Tuning and capacity
