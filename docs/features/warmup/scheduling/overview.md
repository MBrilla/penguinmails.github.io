---
title: "Warmup Scheduling"
description: "Automated email warmup with gradual volume ramping, domain/IP strategies, engagement simulation, and progress tracking"
last_modified_date: "2025-12-04"
level: "2"
persona: "Deliverability Teams"
keywords: "warmup scheduling, IP warmup, email deliverability"
---

# Warmup Scheduling

**Quick Access**: Automatically warm up new email accounts and domains with intelligent scheduling, gradual volume increases, and engagement simulation to build sender reputation.

## Overview

Warmup Scheduling automates the process of establishing sender reputation for new email accounts and domains. The system gradually increases sending volume while simulating natural engagement patterns to avoid spam filters and build trust with email providers.

### Key Capabilities

- **Automated Warmup Schedules**: Pre-configured ramp-up plans

- **Gradual Volume Ramping**: Increase sends progressively over 4-8 weeks

- **Domain & IP Warmup**: Separate strategies for domains and dedicated IPs

- **Engagement Simulation**: Automated opens, clicks, and replies

- **Progress Tracking**: Real-time warmup status and metrics

- **Custom Schedules**: Tailor warmup plans to your needs

- **Multi-Account Warmup**: Warm up multiple accounts simultaneously

- **Safety Limits**: Automatic throttling to prevent reputation damage

---

## Level 1: Quick Start Guide

### Start Your First Warmup

#### Step 1: Select Email Account

```text

Warmup → Start New Warmup

Select Email Account:
  ○ sarah@penguinmails.com (Connected 2 days ago)
  ○ support@penguinmails.com (Connected 1 day ago)
  ● sales@penguinmails.com (Connected today)

Account Status:
  Domain: penguinmails.com
  Provider: Google Workspace
  Age: New (0 days)
  Current Reputation: Unknown

[Continue →]

```

#### Step 2: Choose Warmup Plan

```text

Select Warmup Plan:

● Standard Warmup (Recommended)
  Duration: 6 weeks
  Starting Volume: 10 emails/day
  Peak Volume: 500 emails/day
  Best for: Most use cases

○ Aggressive Warmup
  Duration: 4 weeks
  Starting Volume: 20 emails/day
  Peak Volume: 1,000 emails/day
  Best for: Established domains, urgent needs

○ Conservative Warmup
  Duration: 8 weeks
  Starting Volume: 5 emails/day
  Peak Volume: 300 emails/day
  Best for: New domains, high-value reputation

○ Custom Plan
  Configure your own schedule

[Continue →]

```

#### Step 3: Configure Settings

```text

Warmup Configuration

Engagement Simulation:
  ☑ Simulate opens (70% open rate target)
  ☑ Simulate clicks (15% click rate target)
  ☑ Simulate replies (5% reply rate target)
  ☐ Simulate forwards (2% forward rate)

Warmup Pool:
  ● Use PenguinMails warmup network (Recommended)

    - 10,000+ verified warmup accounts

    - Automatic engagement

    - Best deliverability

  ○ Use custom warmup list

    - Upload your own contacts

    - Manual engagement required

Safety Settings:
  ☑ Auto-pause on bounce rate > 5%
  ☑ Auto-pause on spam complaints > 0.1%
  ☑ Daily send limit: 500 emails

[Start Warmup]

```

#### Step 4: Warmup Running

```text

Warmup Progress: sales@penguinmails.com

Status: âœ“ Active (Day 3 of 42)

Today's Activity:
  Scheduled: 30 emails
  Sent: 28 emails
  Delivered: 28 emails (100%)
  Opened: 21 emails (75%)
  Clicked: 5 emails (18%)
  Replied: 2 emails (7%)

Progress:
  Week 1: â–ˆâ–ˆâ–ˆâ–ˆâ–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘ 21% (Day 3/7)

  Current Volume: 30/day
  Target Volume: 500/day

Next Milestone:
  Day 7: Increase to 50 emails/day

Health Score: 95/100 âœ“ Excellent

[View Details] [Pause Warmup] [Adjust Schedule]

```

### Understanding Warmup Phases

#### Phase 1: Foundation (Week 1-2)

```text

Days 1-14: Building Initial Reputation

Daily Volume: 10 → 100 emails/day
Focus: Establish sending pattern

Activities:

  - Send to warmup pool only

  - High engagement simulation (70%+ opens)

  - Consistent sending times

  - Monitor bounce rates closely

Goals:
  âœ“ Zero spam complaints
  âœ“ Bounce rate < 2%
  âœ“ Consistent delivery
  âœ“ Positive engagement signals

```

#### Phase 2: Growth (Week 3-4)

```text

Days 15-28: Scaling Volume

Daily Volume: 100 → 300 emails/day
Focus: Gradual increase

Activities:

  - Continue warmup pool (80%)

  - Introduce real contacts (20%)

  - Maintain high engagement

  - Monitor sender reputation

Goals:
  âœ“ Maintain low bounce rate
  âœ“ Build domain reputation
  âœ“ Inbox placement > 90%
  âœ“ No blacklist appearances

```

#### Phase 3: Maturity (Week 5-6)

```text

Days 29-42: Reaching Target Volume

Daily Volume: 300 → 500 emails/day
Focus: Stabilize at target

Activities:

  - Warmup pool (50%)

  - Real contacts (50%)

  - Natural engagement patterns

  - Full campaign readiness

Goals:
  âœ“ Consistent inbox placement
  âœ“ Strong sender reputation
  âœ“ Ready for production campaigns
  âœ“ Established sending patterns

```

### Quick Actions

#### Pause Warmup

```text

Pause Warmup: sales@penguinmails.com

Reason:
  ○ Temporary hold (will resume)
  ○ Reputation issue detected
  ● Manual review needed
  ○ Other

Resume Plan:
  ○ Resume from current day
  ● Restart current week
  ○ Restart from beginning

[Pause Warmup]

```

#### Adjust Schedule
