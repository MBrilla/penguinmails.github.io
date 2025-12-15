---
title: "Scheduling Best Practices"
description: "Best practices for warmup scheduling"
last_modified_date: "2025-12-04"
level: "2"
keywords: "best practices, optimization, tips"
---

# Scheduling Best Practices

This guide covers best practices for configuring and managing warmup schedules.

## Background Jobs

```typescript
// Daily warmup scheduler
cron.schedule('0 9 * * *', async () => {  // 9 AM daily
  const activeSchedules = await db.warmupSchedules.findAll({
    where: { status: 'active' },
  });

  const service = new WarmupService();

  for (const schedule of activeSchedules) {
    await service.scheduleDailyWarmup(schedule.id);

    // Increment day
    await db.warmupSchedules.update(schedule.id, {
      currentDay: schedule.currentDay + 1,
    });

    // Check if completed
    if (schedule.currentDay >= schedule.totalDays) {
      await db.warmupSchedules.update(schedule.id, {
        status: 'completed',
        completedAt: new Date(),
      });
    }
  }
});

// Health check every hour
cron.schedule('0 * * * *', async () => {
  const activeSchedules = await db.warmupSchedules.findAll({
    where: { status: 'active' },
  });

  const service = new WarmupService();

  for (const schedule of activeSchedules) {
    await service.checkWarmupHealth(schedule.id);
  }
});

```

### API Endpoints

```typescript
// Start warmup
app.post('/api/warmup/start', authenticate, async (req, res) => {
  const service = new WarmupService();

  const schedule = await service.startWarmup(
    req.body.emailAccountId,
    req.body.config
  );

  return res.json(schedule);
});

// Get warmup status
app.get('/api/warmup/:id', authenticate, async (req, res) => {
  const schedule = await db.warmupSchedules.findById(req.params.id);
  const logs = await db.warmupDailyLogs.findAll({
    where: { warmupScheduleId: req.params.id },
    order: [['logDate', 'DESC']],
    limit: 30,
  });

  return res.json({ schedule, logs });
});

// Pause warmup
app.post('/api/warmup/:id/pause', authenticate, async (req, res) => {
  const service = new WarmupService();
  await service.pauseWarmup(req.params.id, req.body.reason);

  return res.json({ success: true });
});

// Resume warmup
app.post('/api/warmup/:id/resume', authenticate, async (req, res) => {
  await db.warmupSchedules.update(req.params.id, {
    status: 'active',
    pausedAt: null,
  });

  return res.json({ success: true });
});

```

---

## Related Documentation

- **[Email Warmups](/docs/features/warmup/email-warmups/overview)** - Overall warmup strategy

- **[Reputation Monitoring](/docs/features/warmup/monitoring/overview)** - Track sender reputation

- **[Domain Management](/docs/features/domains/domain-management)** - Domain configuration

- **[Email Infrastructure](/docs/features/infrastructure/email-infrastructure-setup)** - SMTP setup

---

**Last Updated:** November 25, 2025
**Status:** Planned - High Priority (Level 2)
**Target Release:** Q3 2026
**Owner:** Deliverability Team
