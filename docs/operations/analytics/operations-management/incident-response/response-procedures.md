---
title: "Response Procedures"
description: "Containment, investigation, and recovery procedures for incident response"
last_modified_date: "2025-01-15"
level: "3"
persona: "DevOps, Engineering Teams"
---

# Response Procedures

Structured procedures for containment, investigation, and recovery phases of incident response.

## Containment Phase

### Immediate Actions (Within 30 minutes)

Take these actions immediately upon incident detection:

- Isolate affected systems
- Block malicious traffic
- Disable compromised accounts
- Implement emergency patches

### Short-Term Actions (Within 2 hours)

After immediate containment, implement compensating controls:

- Deploy compensating controls
- Implement monitoring safeguards
- Establish additional logging
- Prepare rollback procedures

## Containment Playbooks

### Security Breach Containment

```typescript
const securityBreachPlaybook = [
  'Revoke all active sessions',
  'Change all privileged credentials',
  'Isolate compromised systems',
  'Enable enhanced monitoring',
  'Notify security team'
];
```

### System Outage Containment

```typescript
const systemOutagePlaybook = [
  'Check system resource utilization',
  'Restart affected services',
  'Verify database connectivity',
  'Check network connectivity',
  'Implement load balancing'
];
```

### Data Incident Containment

```typescript
const dataIncidentPlaybook = [
  'Stop data processing',
  'Assess data exposure',
  'Notify affected customers',
  'Prepare breach notification',
  'Implement data recovery'
];
```

## Investigation Phase

Systematic investigation follows containment:

1. **Evidence Collection:** Preserve logs, screenshots, and system state before any changes
2. **Root Cause Analysis:** Identify underlying cause using 5-whys technique
3. **Impact Analysis:** Assess full scope of incident effects on systems and users
4. **Timeline Reconstruction:** Document chronological sequence of events
5. **Evidence Preservation:** Maintain chain of custody for potential legal purposes

### 5-Whys Technique

For each symptom, ask "Why?" five times to reach root cause:

1. **Why** did the system fail? → Database connection pool exhausted
2. **Why** was the pool exhausted? → Connection leak in new code
3. **Why** was there a connection leak? → Missing connection cleanup
4. **Why** was cleanup missing? → Code review missed the issue
5. **Why** did code review miss it? → No automated leak detection

Root cause: Missing automated connection leak detection in CI/CD pipeline.

## Recovery Phase

### Validation Steps

Before declaring recovery complete:

- Verify fix effectiveness
- Test system functionality
- Monitor system stability
- Validate data integrity

### Recovery Approaches

| Approach | Description | Use When |
|----------|-------------|----------|
| Rollback | Revert to previous known good state | Quick fix needed, rollback is clean |
| Fix Forward | Implement and deploy fix | Rollback would cause data loss |
| Workaround | Implement temporary solution | Permanent fix requires more time |

### Recovery Criteria

All criteria must be met before declaring incident resolved:

- System stability confirmed (no recurring errors)
- All tests passing (unit, integration, smoke)
- Performance within normal ranges
- Security controls validated

### Post-Recovery Monitoring

Continue enhanced monitoring after recovery:

- **Duration:** 24-72 hours post-recovery
- **Focus Areas:**
  - System performance metrics
  - Error rates and patterns
  - User activity levels
  - Security alert patterns

## Incident Status Tracking

### Status Progression

```
detected → investigating → mitigating → resolved → post-mortem
```

### Status Definitions

| Status | Description | Actions |
|--------|-------------|---------|
| Detected | Issue identified | Assess severity, notify IRT |
| Investigating | Analyzing root cause | Document findings, update timeline |
| Mitigating | Implementing fix | Apply changes, monitor results |
| Resolved | Issue fixed | Validate fix, update stakeholders |
| Post-mortem | Learning phase | Document lessons, assign follow-ups |

---

**Related Documents:**
- [Detection and Assessment](detection-assessment.md)
- [Communication Procedures](communication.md)
- [Post-Incident Activities](post-incident.md)
