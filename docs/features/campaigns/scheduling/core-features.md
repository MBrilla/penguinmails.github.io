---
title: "Core Scheduling Features"
description: "Core scheduling capabilities"
last_modified_date: "2025-12-04"
level: "2"
keywords: "scheduling features, calendar, timing"
---

# Core Scheduling Features

## Key Capabilities

- **Send-Time Optimization**: AI-powered optimal send time based on historical engagement

- **Business Hours Enforcement**: Automatically respect business hours and skip weekends

- **Calendar Integration**: Sync with Google Calendar and Outlook for event-based campaigns

- **Recurring Campaigns**: Set up daily, weekly, or monthly recurring schedules

- **Batch Scheduling**: Schedule multiple campaigns at once with smart conflict resolution

---

## Level 1: Quick Start Guide

### Schedule Your First Campaign

#### Step 1: Basic Scheduling

```text

Campaign Builder → Schedule Tab

┌─────────────────────────────────────────────────────┐
│ When should this campaign be sent?                  │
│                                                     │
│ ○ Send immediately                                  │
│ ● Schedule for later                                │
│                                                     │
│ Date: [Nov 26, 2025 â–¼]                             │
│ Time: [09:00 AM â–¼]                                  │
│ Timezone: [America/New_York â–¼]                      │
│                                                     │
│ Preview: Sends on Monday, Nov 26 at 9:00 AM EST     │
└─────────────────────────────────────────────────────┘

```

#### Step 2: Timezone Options

**Single Timezone (Simple):**

```text

All contacts receive email at:
9:00 AM Eastern Time (EST)

Example:

- Contact in New York: 9:00 AM EST

- Contact in Los Angeles: 6:00 AM PST

- Contact in London: 2:00 PM GMT

```

**Recipient Timezone (Smart):**

```text

Each contact receives email at their local time:
9:00 AM in their timezone

Example:

- Contact in New York: 9:00 AM EST

- Contact in Los Angeles: 9:00 AM PST

- Contact in London: 9:00 AM GMT

```

#### Step 3: Business Hours

**Enable Business Hours Protection:**

```text

┌─────────────────────────────────────────────────────┐
│ Business Hours Settings                             │
│                                                     │
│ ☑ Only send during business hours                   │
│                                                     │
│ Business Hours:                                     │
│ Monday-Friday: 9:00 AM - 5:00 PM                    │
│                                                     │
│ ☑ Skip weekends                                     │
│ ☑ Skip holidays (US calendar)                       │
│                                                     │
│ If outside business hours:                          │
│ ● Delay until next business day                     │
│ ○ Send anyway                                       │
└─────────────────────────────────────────────────────┘

```

**Result:**

```text

Scheduled: Saturday, Nov 23 at 3:00 PM
Actual Send: Monday, Nov 25 at 9:00 AM

(Delayed to next business day)

```

### Recurring Campaigns

#### Daily Newsletter

```text

Recurrence Pattern:
┌─────────────────────────────────────────────────────┐
│ Frequency: ● Daily                                  │
│                                                     │
│ Send at: 8:00 AM                                    │
│ Timezone: Recipient's local time                    │
│                                                     │
│ Days: ☑ Mon ☑ Tue ☑ Wed ☑ Thu ☑ Fri                │
│       ☐ Sat ☐ Sun                                   │
│                                                     │
│ Start: Nov 25, 2025                                 │
│ End: ○ Never  ● After 30 occurrences                │
└─────────────────────────────────────────────────────┘

Preview:

- First send: Monday, Nov 25 at 8:00 AM

- Next 5 sends: Nov 26, 27, 28, 29, Dec 2

- Total sends: 30 (weekdays only)

```

#### Weekly Report

```text

Recurrence Pattern:
┌─────────────────────────────────────────────────────┐
│ Frequency: ● Weekly                                 │
│                                                     │
│ Send every: [1] week(s) on:                         │
│ ☐ Mon ☐ Tue ☐ Wed ☐ Thu ☑ Fri ☐ Sat ☐ Sun         │
│                                                     │
│ Send at: 10:00 AM EST                               │
│                                                     │
│ Start: Nov 29, 2025                                 │
│ End: ○ Never  ● On date: Dec 31, 2025              │
└─────────────────────────────────────────────────────┘

Preview:

- Sends every Friday at 10:00 AM

- Next 3 sends: Nov 29, Dec 6, Dec 13

```

### Quick Schedule Templates

**Pre-Built Schedule Patterns:**

```text

Common Patterns:
┌─────────────────────────────────────────────────────┐
│ ⏳ Morning Blast (9 AM, recipient timezone)         │
│ 🌆 Afternoon Update (2 PM, recipient timezone)      │
│ 📰 Daily Newsletter (8 AM, weekdays only)           │
│  Weekly Report (Friday 10 AM)                     │
│ ⏳ Monthly Update (1st of month, 9 AM)              │
│ 🎯 Optimal Time (AI-powered, per contact)           │
└─────────────────────────────────────────────────────┘

```

---

## Level 2: Advanced Scheduling

### Send-Time Optimization

#### AI-Powered Optimal Timing

**How It Works:**

```text

For each contact, analyze:

1. Historical open times (when they opened past emails)

2. Historical click times (when they clicked links)

3. Industry benchmarks (typical engagement times)

4. Day-of-week patterns (best days for this contact)

Result: Personalized send time for maximum engagement

```

**Configuration:**

```yaml
send_time_optimization:
  enabled: true
  strategy: "engagement_based"

  data_sources:

    - historical_opens: weight 40%

    - historical_clicks: weight 30%

    - industry_benchmarks: weight 20%

    - day_patterns: weight 10%

  constraints:
    min_data_points: 5  # Require 5+ past interactions
    fallback_time: "09:00"
    optimization_window:
      start: "06:00"
      end: "20:00"
    respect_business_hours: true

```

**Example Results:**

```text

Contact A (Software Developer):
  Historical pattern: Opens emails at 7:30 AM
  Optimal send time: 7:15 AM (15 min before typical open)

Contact B (Executive):
  Historical pattern: Opens emails at 2:00 PM
  Optimal send time: 1:45 PM

Contact C (No history):
  Fallback: Industry benchmark (9:00 AM)

```

### Advanced Timezone Handling

#### Multi-Region Campaigns

**Coordinate Across Timezones:**

```yaml
timezone_strategy:
  mode: "coordinated_rollout"

  # Start in earliest timezone, roll west
  rollout_pattern:
    start_timezone: "Pacific/Auckland"  # New Zealand (UTC+13)
    start_time: "09:00"

    # Automatically calculate send times for other zones
    auto_calculate: true

  regions:

    - name: "APAC"
      timezones: ["Pacific/Auckland", "Australia/Sydney", "Asia/Tokyo"]
      send_time: "09:00 local"

    - name: "EMEA"
      timezones: ["Europe/London", "Europe/Paris", "Europe/Moscow"]
      send_time: "09:00 local"

    - name: "Americas"
      timezones: ["America/New_York", "America/Chicago", "America/Los_Angeles"]
      send_time: "09:00 local"

```

**Result:**

```text

Global Campaign Rollout:
09:00 NZDT (Auckland)    → Nov 25, 9:00 AM
09:00 AEDT (Sydney)      → Nov 25, 9:00 AM (1 hr later)
09:00 JST (Tokyo)        → Nov 25, 9:00 AM (2 hrs later)
09:00 GMT (London)       → Nov 25, 9:00 AM (9 hrs later)
09:00 EST (New York)     → Nov 25, 9:00 AM (14 hrs later)
09:00 PST (Los Angeles)  → Nov 25, 9:00 AM (17 hrs later)

Total rollout window: 17 hours
All contacts receive at their local 9 AM

```

#### Daylight Saving Time Handling

**Automatic DST Adjustment:**

```yaml
dst_handling:
  auto_adjust: true

  # When DST changes occur
  transition_behavior:
    spring_forward:  # Clocks move ahead 1 hour
      action: "delay_1_hour"
      reason: "Maintain same wall-clock time"

    fall_back:  # Clocks move back 1 hour
      action: "advance_1_hour"
      reason: "Avoid duplicate sends"
