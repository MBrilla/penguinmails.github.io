---
title: "Advanced Scheduling Options"
description: "Advanced scheduling and automation"
last_modified_date: "2025-12-04"
level: "3"
keywords: "advanced scheduling, automation, smart timing"
---

# Advanced Scheduling Options

## Daylight Saving Time Handling

**Example:**

```text

Campaign scheduled: 9:00 AM EST
DST transition: March 10, 2024 (spring forward)

Before DST: 9:00 AM EST (UTC-5)
After DST:  9:00 AM EDT (UTC-4)

System automatically adjusts to maintain 9:00 AM local time

```

### Calendar Integration

#### Google Calendar Sync

**Event-Based Campaigns:**

```yaml
calendar_integration:
  provider: "google_calendar"
  calendar_id: "primary"

  trigger_rules:

    - event_type: "webinar"
      campaign_template: "webinar_reminder"
      send_before: "1 hour"

    - event_type: "meeting"
      campaign_template: "meeting_followup"
      send_after: "2 hours"

    - event_type: "product_launch"
      campaign_template: "launch_announcement"
      send_at: "event_start"

```

**Example Workflow:**

```text

Google Calendar Event Created:
  Title: "Product Launch Webinar"
  Date: Dec 1, 2025 at 2:00 PM EST

Automatic Campaign Triggers:

  1. Reminder email: Dec 1 at 1:00 PM (1 hour before)

  2. Join link email: Dec 1 at 1:55 PM (5 min before)

  3. Follow-up email: Dec 1 at 4:00 PM (2 hours after)

  4. Recording email: Dec 2 at 9:00 AM (next day)

```

#### Outlook Integration

**Meeting-Based Automation:**

```yaml
outlook_integration:
  enabled: true

  sync_calendars:

    - "Sales Team Calendar"

    - "Marketing Events"

  automation_rules:

    - calendar: "Sales Team Calendar"
      event_contains: "Demo"
      action:
        send_campaign: "demo_followup"
        delay: "1 day"

    - calendar: "Marketing Events"
      event_contains: "Conference"
      action:
        send_campaign: "conference_recap"
        delay: "3 days"

```

### Batch Scheduling

#### Schedule Multiple Campaigns

**Bulk Schedule Interface:**

```text

Batch Scheduler:
┌─────────────────────────────────────────────────────┐
│ Selected Campaigns: 5                               │
│                                                     │
│ ☑ Welcome Series                                    │
│ ☑ Product Update                                    │
│ ☑ Feature Announcement                              │
│ ☑ Customer Survey                                   │
│ ☑ Re-engagement Campaign                            │
│                                                     │
│ Schedule Pattern:                                   │
│ ● Stagger sends (recommended)                       │
│   Interval: [2] hours between campaigns             │
│   Start: Nov 25, 9:00 AM                            │
│                                                     │
│ ○ Send all at once                                  │
│   Time: Nov 25, 9:00 AM                             │
│                                                     │
│ Conflict Resolution:                                │
│ ● Skip contacts in multiple campaigns               │
│ ○ Send to all (may receive multiple emails)         │
└─────────────────────────────────────────────────────┘

Preview:
Campaign 1: Nov 25, 9:00 AM
Campaign 2: Nov 25, 11:00 AM
Campaign 3: Nov 25, 1:00 PM
Campaign 4: Nov 25, 3:00 PM
Campaign 5: Nov 25, 5:00 PM

```

#### Smart Conflict Resolution

**Prevent Email Fatigue:**

```typescript
interface ConflictResolution {
  rules: {
    max_emails_per_day: 2;
    min_hours_between: 4;
    priority_based: true;
  };

  actions: {
    when_conflict: 'delay_lower_priority';
    delay_duration: '24 hours';
    max_delays: 3;
    after_max_delays: 'skip';
  };
}

```

**Example:**

```text

Contact receives:

- Campaign A (Priority: High) at 9:00 AM

- Campaign B (Priority: Medium) scheduled 11:00 AM

- Campaign C (Priority: Low) scheduled 2:00 PM

Conflict Resolution:
âœ“ Campaign A: Sent at 9:00 AM (high priority)
â¸ Campaign B: Delayed to 1:00 PM (4hr minimum gap)
â¸ Campaign C: Delayed to next day (max 2/day limit)

Result:
Nov 25: Campaign A (9 AM), Campaign B (1 PM)
Nov 26: Campaign C (9 AM)

```

### Holiday and Blackout Dates

#### Configure Blackout Periods

```yaml
blackout_dates:
  # US Holidays

  - date: "2025-12-25"
    name: "Christmas Day"
    action: "delay_to_next_business_day"

  - date: "2025-01-01"
    name: "New Year's Day"
    action: "delay_to_next_business_day"

  # Company blackouts

  - date_range:
      start: "2025-12-24"
      end: "2025-12-26"
    name: "Holiday Break"
    action: "skip"

  # Industry events

  - date: "2025-11-28"
    name: "Black Friday"
    action: "allow"  # Override, send anyway

```

**Custom Holiday Calendars:**

```text

Holiday Calendar: US Federal + Company
┌─────────────────────────────────────────────────────┐
│ ☑ New Year's Day                                    │
│ ☑ Martin Luther King Jr. Day                        │
│ ☑ Presidents Day                                    │
│ ☑ Memorial Day                                      │
│ ☑ Independence Day                                  │
│ ☑ Labor Day                                         │
│ ☑ Thanksgiving                                      │
│ ☑ Christmas                                         │
│                                                     │
│ Custom Dates:                                       │
│ + Company Anniversary (June 15)                     │
│ + Annual Conference (Oct 10-12)                     │
└─────────────────────────────────────────────────────┘

```

---

## Level 3: Technical Implementation

### Database Schema

```sql
-- Campaign schedules
CREATE TABLE campaign_schedules (
  id UUID PRIMARY KEY,
  campaign_id UUID NOT NULL REFERENCES campaigns(id) ON DELETE CASCADE,

  -- Schedule type
  schedule_type VARCHAR(50), -- immediate, scheduled, recurring, optimized

  -- Basic scheduling
  scheduled_date DATE,
  scheduled_time TIME,
  timezone VARCHAR(100) DEFAULT 'UTC',

  -- Timezone strategy
  timezone_strategy VARCHAR(50), -- single, recipient_local, coordinated

  -- Recurrence
  is_recurring BOOLEAN DEFAULT FALSE,
  recurrence_pattern JSONB,
  recurrence_end_date DATE,
  recurrence_count INTEGER,

  -- Optimization
  use_send_time_optimization BOOLEAN DEFAULT FALSE,
  optimization_window_start TIME,
  optimization_window_end TIME,

  -- Business hours
  respect_business_hours BOOLEAN DEFAULT FALSE,
  business_hours_start TIME DEFAULT '09:00',
  business_hours_end TIME DEFAULT '17:00',
  skip_weekends BOOLEAN DEFAULT FALSE,
  skip_holidays BOOLEAN DEFAULT FALSE,

  -- Calendar integration
  calendar_provider VARCHAR(50), -- google, outlook, null
  calendar_event_id VARCHAR(255),
  calendar_trigger_type VARCHAR(50), -- before, after, at
  calendar_trigger_offset INTERVAL,

  -- Status
  status VARCHAR(50), -- pending, processing, completed, failed
  last_processed_at TIMESTAMP,
  next_send_at TIMESTAMP,

  -- Metadata
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_campaign_schedules_campaign ON campaign_schedules(campaign_id);
CREATE INDEX idx_campaign_schedules_next_send ON campaign_schedules(next_send_at);
CREATE INDEX idx_campaign_schedules_status ON campaign_schedules(status);

-- Send time rules (for optimization)
CREATE TABLE send_time_rules (
  id UUID PRIMARY KEY,
  tenant_id UUID NOT NULL REFERENCES tenants(id),

  -- Rule configuration
  rule_name VARCHAR(255),
  rule_type VARCHAR(50), -- global, segment, contact

  -- Target
  segment_id UUID REFERENCES segments(id),
  contact_id UUID REFERENCES contacts(id),

  -- Optimization settings
  optimization_strategy VARCHAR(50), -- engagement_based, industry_benchmark, custom
  min_data_points INTEGER DEFAULT 5,
  fallback_time TIME DEFAULT '09:00',

  -- Constraints
  allowed_days JSONB, -- [1,2,3,4,5] for Mon-Fri
  allowed_hours_start TIME,
  allowed_hours_end TIME,

  -- Weights for optimization
  open_time_weight DECIMAL(3,2) DEFAULT 0.40,
  click_time_weight DECIMAL(3,2) DEFAULT 0.30,
  industry_weight DECIMAL(3,2) DEFAULT 0.20,
  day_pattern_weight DECIMAL(3,2) DEFAULT 0.10,

  -- Metadata
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_send_time_rules_tenant ON send_time_rules(tenant_id);
CREATE INDEX idx_send_time_rules_segment ON send_time_rules(segment_id);

-- Blackout dates
CREATE TABLE blackout_dates (
  id UUID PRIMARY KEY,
  tenant_id UUID NOT NULL REFERENCES tenants(id),

  -- Date range
  blackout_date DATE,
  blackout_start_date DATE,
  blackout_end_date DATE,

  -- Details
  name VARCHAR(255),
  description TEXT,
  blackout_type VARCHAR(50), -- holiday, company_event, industry_event

  -- Action
  action VARCHAR(50), -- skip, delay, allow

  -- Scope
