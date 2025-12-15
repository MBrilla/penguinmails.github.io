---
title: "Scheduling Best Practices"
description: "Best practices and optimization"
last_modified_date: "2025-12-04"
level: "2"
keywords: "best practices, optimization, tips"
---

# Scheduling Best Practices

This guide covers best practices for campaign scheduling and send-time optimization.

## Send Time Optimization

```typescript
class SendTimeOptimizer {
  async calculateOptimalSendTime(contact: Contact): Promise<void> {
    const emails = await this.getContactEngagementHistory(contact.id);
    const openTimes = emails.filter(e => e.opened).map(e => moment(e.openedAt).hour());
    const clickTimes = emails.filter(e => e.clicked).map(e => moment(e.clickedAt).hour());

    // Find most common hour
    const optimalHour = this.getMostCommonValue([...openTimes, ...clickTimes]);

    // Calculate engagement by day
    const engagementByDay = this.calculateEngagementByDay(emails);
    const optimalDay = this.getHighestEngagementDay(engagementByDay);

    // Update preferences
    await db.contactSendPreferences.upsert({
      contactId: contact.id,
      optimalSendTime: `${optimalHour}:00`,
      optimalSendDay: optimalDay,
      typicalOpenTimes: openTimes,
      typicalClickTimes: clickTimes,
      engagementByDay,
      dataPointsCount: emails.length,
      confidenceScore: Math.min(emails.length / 20, 1.0),
      lastCalculatedAt: new Date(),
    });
  }

  logger.info('Contact send preferences updated');
});

```

### API Endpoints

```typescript
// Create campaign schedule
router.post('/api/campaigns/:id/schedule', async (req, res) => {
  const { scheduleType, scheduledDate, scheduledTime, timezone, recurrencePattern } = req.body;

  const schedule = await db.campaignSchedules.create({
    campaignId: req.params.id,
    scheduleType,
    scheduledDate,
    scheduledTime,
    timezone,
    isRecurring: !!recurrencePattern,
    recurrencePattern,
    status: 'pending',
  });

  // Calculate first send time
  const schedulingService = new CampaignSchedulingService();
  const nextSendAt = await schedulingService.calculateNextSendTime(schedule);

  await db.campaignSchedules.update(schedule.id, { nextSendAt });

  res.json(schedule);
});

// Get campaign schedule
router.get('/api/campaigns/:id/schedule', async (req, res) => {
  const schedule = await db.campaignSchedules.findByCampaign(req.params.id);
  res.json(schedule);
});

// Update campaign schedule
router.put('/api/campaigns/:id/schedule', async (req, res) => {
  const schedule = await db.campaignSchedules.findByCampaign(req.params.id);

  await db.campaignSchedules.update(schedule.id, req.body);

  // Recalculate send time
  const schedulingService = new CampaignSchedulingService();
  const nextSendAt = await schedulingService.calculateNextSendTime(schedule);

  await db.campaignSchedules.update(schedule.id, { nextSendAt });

  res.json({ success: true });
});

// Preview send times for contacts
router.post('/api/campaigns/:id/schedule/preview', async (req, res) => {
  const { sampleSize = 10 } = req.body;

  const campaign = await db.campaigns.findById(req.params.id);
  const schedule = await db.campaignSchedules.findByCampaign(req.params.id);
  const contacts = await db.segments.getContacts(campaign.segmentId, sampleSize);

  const schedulingService = new CampaignSchedulingService();

  const previews = await Promise.all(
    contacts.map(async (contact) => {
      const sendTime = await schedulingService.calculateNextSendTime(schedule, contact);

      return {
        contactId: contact.id,
        contactEmail: contact.email,
        contactTimezone: contact.timezone,
        sendTime,
        sendTimeLocal: moment.tz(sendTime, contact.timezone).format('YYYY-MM-DD HH:mm:ss z'),
      };
    })
  );

  res.json({ previews });
});

```

---

## Related Documentation

### Campaign Features

- **[Campaign Management](/docs/features/campaigns/campaign-management/hub)** - Core campaign creation and management

- **[Email Sequences](/docs/features/campaigns/email-sequences)** - Multi-step automated sequences

- **[A/B Testing](/docs/features/campaigns/ab-testing)** - Campaign optimization

### Analytics

- **[Core Analytics](/docs/features/analytics/core-analytics/overview)** - Performance tracking

- **[Enhanced Analytics](/docs/features/analytics/enhanced-analytics/overview)** - Advanced insights

### Technical

- **[Email Pipeline](/docs/features/queue/email-pipeline)** - Email sending infrastructure

- **[API Documentation](/docs/implementation-technical/api/tenant-api/campaigns)** - Campaign APIs

---

**Last Updated:** November 25, 2025
**Status:** Planned - High Priority Q1 2026 Feature
**Target Release:** Q1 2026
**Owner:** Campaigns Team
