---
title: "Staff Operations"
last_modified_date: "2025-12-15"
level: "2"
persona: "Team Leads, HR, Operations"
---

# Staff Operations

Team scheduling, performance tracking, and onboarding procedures.

## Staff Scheduling

### Shift Management

- **24/7 Coverage**: Support for continuous operations
- **Rotation Schedules**: Fair distribution of on-call duties
- **Holiday Planning**: Coverage planning for peak periods
- **Capacity Planning**: Staff allocation based on workload forecasts

### On-Call Rotations

```yaml
On-Call Schedule:
  Primary:
    Duration: 1 week
    Rotation: Weekly (Monday 9am handoff)
    Escalation: 15 minutes response time
  
  Secondary:
    Duration: 1 week
    Rotation: Offset by 3 days from primary
    Escalation: 30 minutes if primary unavailable
  
  Escalation Chain:
    1. Primary on-call engineer
    2. Secondary on-call engineer
    3. Team lead
    4. Engineering manager
```

## Performance Tracking

### Individual Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| Sprint velocity | Consistent +/- 10% | Story points completed |
| Code review turnaround | < 24 hours | Hours from request to review |
| Bug fix rate | > 90% within sprint | Bugs fixed vs. introduced |
| Documentation | 100% for new features | Docs created per feature |

### Team Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| Sprint completion | > 85% | Story points delivered |
| Team velocity trend | Stable or improving | 6-sprint rolling average |
| Cross-training | 100% coverage | No single points of failure |
| Knowledge sharing | Weekly | Tech talks, pair programming |

## Onboarding Process

### Week 1: Foundation

- **Day 1**: HR orientation, equipment setup, account provisioning
- **Day 2**: Architecture overview, codebase walkthrough
- **Day 3**: Development environment setup, first PR
- **Day 4**: Team process introduction, pairing sessions
- **Day 5**: First week retrospective, feedback collection

### Week 2: Integration

- **Sprint participation**: First full sprint involvement
- **Mentorship**: Assigned mentor for guidance
- **Code review**: Begin participating in reviews
- **Documentation**: Update onboarding docs with learnings

### Week 3-4: Independence

- **Feature ownership**: Take ownership of small feature
- **On-call shadow**: Shadow on-call rotation
- **Process contribution**: Suggest process improvements
- **30-day review**: Formal feedback and goal setting

## Performance Reviews

### Quarterly Check-ins

- Goal progress review
- Skill development tracking
- Career path discussion
- Feedback exchange (bidirectional)

### Annual Reviews

- Comprehensive performance assessment
- Compensation review
- Career development planning
- Goal setting for next year

---

[← Back to Team Performance Framework](./README)
