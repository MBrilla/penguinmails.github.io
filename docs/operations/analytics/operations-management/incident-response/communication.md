---
title: "Communication Procedures"
description: "Internal and external communication procedures during incidents"
last_modified_date: "2025-01-15"
level: "2"
persona: "Operations Teams, Communications"
---

# Communication Procedures

Structured communication procedures for internal teams, customers, regulators, and media during incidents.

## Internal Communication

### Immediate Communication (Active Response)

During active incident response:

- **Channels:** Slack incident channel, Emergency phone bridge
- **Audience:** IRT members, Engineering team, Management
- **Frequency:** Every 30 minutes during active response
- **Content:** Status updates, action items, blockers

### Regular Updates

For ongoing awareness across the organization:

- **Channels:** Slack updates, Email summaries
- **Audience:** All employees, Stakeholders
- **Frequency:** Hourly updates for SEV-1, Daily for others
- **Content:** Incident status, expected resolution time, business impact

## External Communication

### Customer Communication

Trigger customer communication when:
- Service degradation exceeds 5 minutes
- Data security incident is confirmed

Communication channels:
- Status page updates
- Email notifications
- Support portal announcements

Content to include:
- Incident acknowledgment
- Impact assessment
- Resolution timeline
- Workaround instructions (if available)

### Regulatory Communication

Regulatory notification is required for:
- Data breach affecting personal information
- Security incident with regulatory implications
- Service outage exceeding 4 hours

Timeframes:
- **GDPR:** 72 hours to supervisory authority, without undue delay to individuals
- **Critical security:** Immediate notification required

Content requirements:
- Incident description
- Affected data scope
- Mitigation measures
- Prevention plans

### Media Communication

Media engagement triggers:
- Significant customer impact
- Public interest in incident

Coordination:
- All media contact through designated PR contact
- Prepared statements approved by legal
- Background information for context

## Status Page Management

### Real-Time Updates

The status page provides customer-facing incident information:

- **Automated Updates:** Status changes based on monitoring
- **Incident Details:** Clear description of issues and impact
- **Timeline:** Chronological incident timeline
- **Progress Updates:** Regular updates on resolution progress
- **Post-Resolution:** Public summary after incident closure

### Status Levels

| Status | Display | Customer Impact |
|--------|---------|-----------------|
| Operational | Green | No issues |
| Degraded | Yellow | Partial service impact |
| Partial Outage | Orange | Some features unavailable |
| Major Outage | Red | Service unavailable |

## Communication Templates

### Initial Incident Notification

```
INCIDENT: [Brief Description]
SEVERITY: [SEV-1/SEV-2/SEV-3/SEV-4]
IMPACT: [Affected systems/users]
STATUS: Investigating
NEXT UPDATE: [Time - e.g., in 30 minutes]
```

### Status Update

```
INCIDENT UPDATE: [Brief Description]
STATUS: [Investigating/Mitigating/Resolved]
CURRENT ACTIONS: [What we're doing now]
ETA: [Estimated resolution time]
NEXT UPDATE: [Time for next update]
```

### Resolution Notification

```
INCIDENT RESOLVED: [Brief Description]
RESOLUTION: [How it was fixed]
DURATION: [Total incident duration]
IMPACT SUMMARY: [Final impact assessment]
FOLLOW-UP: [Post-incident review date]
```

## Data Breach Notifications

### GDPR Requirements

For data breaches affecting EU residents:

- **Timeframe:** 72 hours to supervisory authority
- **Individual Notification:** Without undue delay when high risk

Required content:
- Nature of breach
- Categories and number of individuals affected
- Contact details of DPO
- Likely consequences
- Measures taken to mitigate

### CCPA Requirements

For breaches affecting California residents:

- **Timeframe:** 45 days for written notification
- **Credit Reporting:** 45 days notification to agencies

Required content:
- Breach description
- Affected data types
- Remediation steps
- Contact information

---

**Related Documents:**
- [Incident Response Overview](overview.md)
- [Response Procedures](response-procedures.md)
- [Post-Incident Activities](post-incident.md)
