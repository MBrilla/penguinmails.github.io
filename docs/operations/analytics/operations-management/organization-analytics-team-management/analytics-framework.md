---
title: "Analytics Framework"
last_modified_date: "2025-12-15"
level: "2"
persona: "Team Leads, Analysts"
---

# Analytics Framework

Team performance metrics, KPIs, and organization health tracking.

## Team Engagement Analytics

```typescript
interface TeamEngagement {
  companyId: string;
  teamMetrics: {
    totalMembers: number;
    activeMembers: number;
    invitedMembers: number;
    pendingInvites: number;
  };
  roleDistribution: {
    owners: number;
    admins: number;
    members: number;
  };
  activity: {
    dailyActiveUsers: number;
    weeklyActiveUsers: number;
    averageSessionTime: number;
    featureAdoptionByRole: Record<string, number>;
  };
  collaboration: {
    sharedCampaigns: number;
    teamActions: number;
    crossMemberInteractions: number;
  };
}
```

## Organization Health KPIs

| Metric | Description | Target |
|--------|-------------|--------|
| Team Utilization Rate | Active / total invited | > 80% |
| Invitation Acceptance Rate | Accepted / total sent | > 90% |
| Role Distribution Balance | Admin-to-member ratios | 1:5 |
| Team Productivity Score | Campaign creation rate | Trending up |
| Onboarding Completion Rate | Setup completed | > 95% |

## Team Analytics Dashboard

```text
Team Overview
├── Total Members: X (Active: X, Pending: X)
├── Role Distribution: X Owners, X Admins, X Members
├── Engagement Rate: X% (Last 30 days)
└── Growth Rate: +X% this month

Collaboration Metrics
├── Shared Campaigns: X this month
├── Team Actions: X (↑X% vs last month)
├── Cross-member Interactions: X
└── Average Team Size: X members

Performance Indicators
├── Team Productivity Score: X/100
├── Onboarding Completion: X%
├── Access Security Score: X/100
└── Team Satisfaction: X/5
```

## Performance Tracking

### Individual Metrics

| Metric | Measurement | Target |
|--------|-------------|--------|
| Campaigns Created | Count per month | Baseline +10% |
| Email Volume | Sends per month | Within limits |
| Engagement Rate | Opens + clicks | Above average |
| Response Time | Hours to respond | < 24 hours |

### Team Metrics

| Metric | Measurement | Target |
|--------|-------------|--------|
| Total Campaigns | Team aggregate | Growth trend |
| Collaboration Rate | Shared campaigns | > 20% |
| Feature Adoption | Features used | > 70% |
| Support Tickets | Issues raised | Decreasing |

---

[← Back to Organization Analytics](./README)
