---
title: "Technical Implementation"
description: "Technical implementation details"
last_modified_date: "2025-12-04"
level: "3"
keywords: "technical, implementation, architecture"
---

# Technical Implementation

This document covers the technical architecture for campaign scheduling.

## Database Schema

```sql
-- Blackout dates table
CREATE TABLE blackout_dates (
  id UUID PRIMARY KEY,
  tenant_id UUID NOT NULL,
  blackout_date DATE NOT NULL,
  applies_to VARCHAR(50), -- all, specific_campaigns
  campaign_ids JSONB,

  -- Metadata
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_blackout_dates_tenant ON blackout_dates(tenant_id);
CREATE INDEX idx_blackout_dates_date ON blackout_dates(blackout_date);

-- Contact send time preferences (learned from behavior)
CREATE TABLE contact_send_preferences (
  id UUID PRIMARY KEY,
  contact_id UUID NOT NULL REFERENCES contacts(id),

  -- Optimal times (learned)
  optimal_send_time TIME,
  optimal_send_day INTEGER, -- 1-7 for Mon-Sun
  optimal_timezone VARCHAR(100),

  -- Historical patterns
  typical_open_times JSONB, -- Array of times when contact opens emails
  typical_click_times JSONB,
  engagement_by_day JSONB, -- Engagement rates by day of week
  engagement_by_hour JSONB, -- Engagement rates by hour

  -- Confidence
  data_points_count INTEGER DEFAULT 0,
  confidence_score DECIMAL(3,2), -- 0.00 - 1.00

  -- Last updated
  last_calculated_at TIMESTAMP,

  UNIQUE(contact_id)
);

CREATE INDEX idx_contact_send_prefs_contact ON contact_send_preferences(contact_id);

```

### Scheduling Service

```typescript
interface CampaignSchedule {
  id: string;
  campaignId: string;
  scheduleType: 'immediate' | 'scheduled' | 'recurring' | 'optimized';
  scheduledDate?: Date;
  scheduledTime?: string;
  timezone: string;
  timezoneStrategy: 'single' | 'recipient_local' | 'coordinated';
  isRecurring: boolean;
  recurrencePattern?: RecurrencePattern;
  useSendTimeOptimization: boolean;
  respectBusinessHours: boolean;
  skipWeekends: boolean;
  skipHolidays: boolean;
}

interface RecurrencePattern {
  frequency: 'daily' | 'weekly' | 'monthly';
  interval: number;
  daysOfWeek?: number[]; // 0-6 for Sun-Sat
  dayOfMonth?: number;
  endDate?: Date;
  occurrences?: number;
}

class CampaignSchedulingService {
  /**

   * Calculate next send time for a campaign
   */
  async calculateNextSendTime(
    schedule: CampaignSchedule,
    contact?: Contact
  ): Promise<Date> {
    let sendTime: Date;

    switch (schedule.scheduleType) {
      case 'immediate':
        sendTime = new Date();
        break;

      case 'scheduled':
        sendTime = this.parseScheduledTime(schedule);
        break;

      case 'recurring':
        sendTime = this.calculateRecurringSendTime(schedule);
        break;

      case 'optimized':
        sendTime = await this.calculateOptimalSendTime(contact);
        break;
    }

    // Apply timezone strategy
    if (schedule.timezoneStrategy === 'recipient_local' && contact) {
      sendTime = this.adjustForRecipientTimezone(sendTime, contact);
    }

    // Apply business hours constraints
    if (schedule.respectBusinessHours) {
      sendTime = this.enforceBusinessHours(sendTime, schedule);
    }

    // Check blackout dates
    sendTime = await this.checkBlackoutDates(sendTime, schedule);

    return sendTime;
  }

  /**

   * Calculate optimal send time based on contact behavior
   */
  private async calculateOptimalSendTime(contact: Contact): Promise<Date> {
    const preferences = await db.contactSendPreferences.findByContact(
      contact.id
    );

    if (!preferences || preferences.dataPointsCount < 5) {
      // Not enough data, use fallback
      return this.getDefaultSendTime();
    }

    // Use learned optimal time
    const now = new Date();
    const optimal = new Date(now);

    // Set to optimal hour
    const [hour, minute] = preferences.optimalSendTime.split(':');
    optimal.setHours(parseInt(hour), parseInt(minute), 0, 0);

    // If optimal time has passed today, schedule for tomorrow
    if (optimal < now) {
      optimal.setDate(optimal.getDate() + 1);
    }

    // Adjust for optimal day of week if needed
    if (preferences.optimalSendDay) {
      while (optimal.getDay() !== preferences.optimalSendDay) {
        optimal.setDate(optimal.getDate() + 1);
      }
    }

    return optimal;
  }

  /**

   * Adjust send time for recipient's timezone
   */
  private adjustForRecipientTimezone(
    sendTime: Date,
    contact: Contact
  ): Date {
    const contactTimezone = contact.timezone || 'UTC';
    const campaignTimezone = 'UTC';

    // Convert to contact's local time
    const localTime = moment.tz(sendTime, campaignTimezone)
      .tz(contactTimezone)
      .toDate();

    return localTime;
  }

  /**

   * Enforce business hours constraints
   */
  private enforceBusinessHours(
    sendTime: Date,
    schedule: CampaignSchedule
  ): Date {
    const hour = sendTime.getHours();
    const day = sendTime.getDay();

    // Check if weekend
    if (schedule.skipWeekends && (day === 0 || day === 6)) {
      // Move to next Monday
      const daysToAdd = day === 0 ? 1 : 2;
      sendTime.setDate(sendTime.getDate() + daysToAdd);
      sendTime.setHours(9, 0, 0, 0); // 9 AM
      return sendTime;
    }

    // Check if outside business hours
    const businessStart = parseInt(schedule.businessHoursStart.split(':')[0]);
    const businessEnd = parseInt(schedule.businessHoursEnd.split(':')[0]);

    if (hour < businessStart) {
      // Too early, move to business start
      sendTime.setHours(businessStart, 0, 0, 0);
    } else if (hour >= businessEnd) {
      // Too late, move to next business day
      sendTime.setDate(sendTime.getDate() + 1);
      sendTime.setHours(businessStart, 0, 0, 0);

      // Check if next day is weekend
      if (schedule.skipWeekends && sendTime.getDay() === 6) {
        sendTime.setDate(sendTime.getDate() + 2); // Skip to Monday
      }
    }

    return sendTime;
  }

  /**

   * Check and handle blackout dates
   */
  private async checkBlackoutDates(
    sendTime: Date,
    schedule: CampaignSchedule
  ): Promise<Date> {
    const blackouts = await db.blackoutDates.findForDate(sendTime);

    for (const blackout of blackouts) {
      if (blackout.action === 'skip') {
        throw new Error(`Cannot send on blackout date: ${blackout.name}`);
      }

      if (blackout.action === 'delay') {
        // Move to next business day
        sendTime.setDate(sendTime.getDate() + 1);
        sendTime.setHours(9, 0, 0, 0);

        // Recursively check new date
        return await this.checkBlackoutDates(sendTime, schedule);
      }
    }

    return sendTime;
  }

  /**

   * Calculate next occurrence for recurring campaign
   */
  private calculateRecurringSendTime(
    schedule: CampaignSchedule
  ): Date {
    const pattern = schedule.recurrencePattern;
    const lastSend = schedule.lastProcessedAt || new Date();
    const nextSend = new Date(lastSend);

    switch (pattern.frequency) {
      case 'daily':
        nextSend.setDate(nextSend.getDate() + pattern.interval);
        break;

      case 'weekly':
        nextSend.setDate(nextSend.getDate() + (7 * pattern.interval));

        // Adjust to correct day of week
        if (pattern.daysOfWeek && pattern.daysOfWeek.length > 0) {
          while (!pattern.daysOfWeek.includes(nextSend.getDay())) {
            nextSend.setDate(nextSend.getDate() + 1);
          }
        }
        break;

      case 'monthly':
        nextSend.setMonth(nextSend.getMonth() + pattern.interval);

        // Adjust to correct day of month
        if (pattern.dayOfMonth) {
          nextSend.setDate(pattern.dayOfMonth);
        }
        break;
    }

    return nextSend;
  }
}

```

### Background Jobs

```typescript
// Process scheduled campaigns every minute
cron.schedule('* * * * *', async () => {
  const now = new Date();

  // Find campaigns ready to send
  const readySchedules = await db.campaignSchedules.findWhere({
    status: 'pending',
    nextSendAt: { lte: now },
  });

  const schedulingService = new CampaignSchedulingService();

  for (const schedule of readySchedules) {
    try {
      // Launch campaign
      const campaignEngine = new CampaignExecutionEngine();
      await campaignEngine.launchCampaign(schedule.campaignId);

      // Update schedule status
      if (schedule.isRecurring) {
        // Calculate next occurrence
        const nextSend = schedulingService.calculateRecurringSendTime(schedule);

        await db.campaignSchedules.update(schedule.id, {
          lastProcessedAt: now,
          nextSendAt: nextSend,
        });
      } else {
        // One-time schedule, mark as completed
        await db.campaignSchedules.update(schedule.id, {
          status: 'completed',
          lastProcessedAt: now,
        });
      }

      logger.info(`Campaign ${schedule.campaignId} sent on schedule`);
    } catch (error) {
      logger.error(`Error sending scheduled campaign ${schedule.campaignId}:`, error);

      await db.campaignSchedules.update(schedule.id, {
        status: 'failed',
      });
    }
  }
});

// Update contact send preferences daily
cron.schedule('0 3 * * *', async () => {  // 3 AM daily
  const contacts = await db.contacts.findAll();

  for (const contact of contacts) {
    // Get email engagement history
    const emails = await db.emails.findWhere({
      contactId: contact.id,
      sentAt: { gte: moment().subtract(90, 'days').toDate() },
    });

    if (emails.length < 5) continue; // Need minimum data

    // Calculate optimal send time
    const openTimes = emails
      .filter(e => e.openedAt)
      .map(e => moment(e.openedAt).hour());

    const clickTimes = emails
      .filter(e => e.clickedAt)
