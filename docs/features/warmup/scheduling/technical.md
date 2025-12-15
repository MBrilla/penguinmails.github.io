---
title: "Technical Implementation"
description: "Warmup scheduling technical details"
last_modified_date: "2025-12-04"
level: "3"
keywords: "technical, implementation, architecture"
---

# Technical Implementation

This document covers the technical architecture for warmup scheduling.

## Database Schema

```sql
-- Warmup schedules table
CREATE TABLE warmup_schedules (
  id UUID PRIMARY KEY,
  tenant_id UUID NOT NULL,
  inbox_id UUID NOT NULL,
  start_volume INTEGER,
  target_volume INTEGER,
  current_volume INTEGER,

  -- Engagement targets
  target_open_rate DECIMAL(5,2),
  target_click_rate DECIMAL(5,2),
  target_reply_rate DECIMAL(5,2),

  -- Pool distribution
  warmup_pool_percentage INTEGER DEFAULT 100,
  real_contacts_percentage INTEGER DEFAULT 0,

  -- Status
  status VARCHAR(50),  -- active, paused, completed, failed
  started_at TIMESTAMP,
  paused_at TIMESTAMP,
  completed_at TIMESTAMP,

  -- Safety settings
  max_bounce_rate DECIMAL(5,2) DEFAULT 5.0,
  max_spam_rate DECIMAL(5,2) DEFAULT 0.1,
  auto_pause_enabled BOOLEAN DEFAULT TRUE,

  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_warmup_schedules_account ON warmup_schedules(email_account_id);
CREATE INDEX idx_warmup_schedules_status ON warmup_schedules(status);

-- Warmup daily logs
CREATE TABLE warmup_daily_logs (
  id UUID PRIMARY KEY,
  warmup_schedule_id UUID NOT NULL REFERENCES warmup_schedules(id),

  -- Date
  log_date DATE NOT NULL,
  day_number INTEGER,

  -- Volume
  scheduled_volume INTEGER,
  sent_count INTEGER DEFAULT 0,
  delivered_count INTEGER DEFAULT 0,

  -- Engagement
  opens INTEGER DEFAULT 0,
  clicks INTEGER DEFAULT 0,
  replies INTEGER DEFAULT 0,

  -- Deliverability
  bounces INTEGER DEFAULT 0,
  spam_complaints INTEGER DEFAULT 0,

  -- Rates
  open_rate DECIMAL(5,2),
  click_rate DECIMAL(5,2),
  reply_rate DECIMAL(5,2),
  bounce_rate DECIMAL(5,2),
  spam_rate DECIMAL(5,2),

  -- Health
  health_score INTEGER,

  created_at TIMESTAMP DEFAULT NOW(),

  UNIQUE(warmup_schedule_id, log_date)
);

CREATE INDEX idx_warmup_logs_schedule ON warmup_daily_logs(warmup_schedule_id, log_date);

-- Warmup pool accounts
CREATE TABLE warmup_pool_accounts (
  id UUID PRIMARY KEY,

  email VARCHAR(255) NOT NULL UNIQUE,
  provider VARCHAR(100),  -- gmail, outlook, yahoo, etc.

  -- Engagement settings
  will_open BOOLEAN DEFAULT TRUE,
  will_click BOOLEAN DEFAULT TRUE,
  will_reply BOOLEAN DEFAULT FALSE,

  -- Behavior
  open_delay_min INTEGER DEFAULT 5,  -- minutes
  open_delay_max INTEGER DEFAULT 120,
  click_probability DECIMAL(3,2) DEFAULT 0.15,
  reply_probability DECIMAL(3,2) DEFAULT 0.05,

  -- Status
  is_active BOOLEAN DEFAULT TRUE,
  last_engaged_at TIMESTAMP,

  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_warmup_pool_provider ON warmup_pool_accounts(provider);
CREATE INDEX idx_warmup_pool_active ON warmup_pool_accounts(is_active);

-- Warmup events
CREATE TABLE warmup_events (
  id UUID PRIMARY KEY,
  warmup_schedule_id UUID NOT NULL REFERENCES warmup_schedules(id),

  event_type VARCHAR(50),  -- sent, opened, clicked, replied, bounced, spam
  event_data JSONB,

  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_warmup_events_schedule ON warmup_events(warmup_schedule_id, created_at);

```

### Warmup Service

```typescript
interface WarmupConfig {
  scheduleType: 'standard' | 'aggressive' | 'conservative' | 'custom';
  totalDays: number;
  startVolume: number;
  targetVolume: number;
  targetOpenRate: number;
  targetClickRate: number;
  targetReplyRate: number;
  warmupPoolPercentage: number;
}

class WarmupService {
  async startWarmup(
    emailAccountId: string,
    config: WarmupConfig
  ): Promise<WarmupSchedule> {
    const schedule = await db.warmupSchedules.create({
      emailAccountId,
      scheduleType: config.scheduleType,
      totalDays: config.totalDays,
      currentDay: 1,
      startVolume: config.startVolume,
      targetVolume: config.targetVolume,
      currentVolume: config.startVolume,
      targetOpenRate: config.targetOpenRate,
      targetClickRate: config.targetClickRate,
      targetReplyRate: config.targetReplyRate,
      warmupPoolPercentage: config.warmupPoolPercentage,
      realContactsPercentage: 100 - config.warmupPoolPercentage,
      status: 'active',
      startedAt: new Date(),
    });

    // Schedule first day's warmup emails
    await this.scheduleDailyWarmup(schedule.id);

    return schedule;
  }

  async scheduleDailyWarmup(scheduleId: string): Promise<void> {
    const schedule = await db.warmupSchedules.findById(scheduleId);

    if (schedule.status !== 'active') {
      return;
    }

    // Calculate today's volume
    const dailyVolume = this.calculateDailyVolume(schedule);

    // Get warmup pool accounts
    const warmupCount = Math.floor(dailyVolume * (schedule.warmupPoolPercentage / 100));
    const realCount = dailyVolume - warmupCount;

    const warmupAccounts = await this.getWarmupPoolAccounts(warmupCount);
    const realContacts = await this.getRealContacts(schedule.tenantId, realCount);

    // Create warmup emails
    for (const account of warmupAccounts) {
      await this.createWarmupEmail(schedule, account, 'warmup_pool');
    }

    for (const contact of realContacts) {
      await this.createWarmupEmail(schedule, contact, 'real_contact');
    }

    // Log daily activity
    await db.warmupDailyLogs.create({
      warmupScheduleId: scheduleId,
      logDate: new Date(),
      dayNumber: schedule.currentDay,
      scheduledVolume: dailyVolume,
    });
  }

  private calculateDailyVolume(schedule: WarmupSchedule): number {
    const { currentDay, totalDays, startVolume, targetVolume } = schedule;

    // Linear ramp-up
    const volumeIncrease = (targetVolume - startVolume) / totalDays;
    const dailyVolume = Math.floor(startVolume + (volumeIncrease * currentDay));

    return Math.min(dailyVolume, targetVolume);
  }

  private async getWarmupPoolAccounts(count: number): Promise<any[]> {
    return await db.warmupPoolAccounts.findAll({
      where: { isActive: true },
      order: db.sequelize.random(),
      limit: count,
    });
  }

  private async getRealContacts(tenantId: string, count: number): Promise<any[]> {
    // Get engaged contacts with good history
    return await db.contacts.findAll({
      where: {
        tenantId,
        isActive: true,
        bounceType: null,
        unsubscribedAt: null,
      },
      order: [['lastActivityAt', 'DESC']],
      limit: count,
    });
  }

  private async createWarmupEmail(
    schedule: WarmupSchedule,
    recipient: any,
    type: 'warmup_pool' | 'real_contact'
  ): Promise<void> {
    // Create email with warmup template
    const email = await emailQueue.add('send-warmup-email', {
      scheduleId: schedule.id,
      emailAccountId: schedule.emailAccountId,
      recipientEmail: recipient.email,
      recipientType: type,
      subject: this.generateWarmupSubject(),
      body: this.generateWarmupBody(recipient),
    });
  }

  private generateWarmupSubject(): string {
    const subjects = [
      'Quick question',
      'Following up',
      'Checking in',
      'Hope you\'re well',
      'Quick update',
    ];

    return subjects[Math.floor(Math.random() * subjects.length)];
  }

  private generateWarmupBody(recipient: any): string {
    return `
      Hi ${recipient.firstName || 'there'},

      Hope this email finds you well. Just wanted to check in and see how things are going.

      Let me know if you have any questions!

      Best regards
    `;
  }

  async simulateEngagement(
    emailId: string,
    recipientType: string
  ): Promise<void> {
    if (recipientType !== 'warmup_pool') {
      return;
    }

    const email = await db.emails.findById(emailId);
    const poolAccount = await db.warmupPoolAccounts.findOne({
      where: { email: email.recipientEmail },
    });

    // Simulate open
    if (poolAccount.willOpen) {
      const openDelay = this.randomDelay(
        poolAccount.openDelayMin,
        poolAccount.openDelayMax
      );

      setTimeout(async () => {
        await this.trackOpen(emailId);

        // Simulate click
        if (Math.random() < poolAccount.clickProbability) {
          const clickDelay = this.randomDelay(1, 30);
          setTimeout(() => this.trackClick(emailId), clickDelay * 60 * 1000);
        }

        // Simulate reply
        if (Math.random() < poolAccount.replyProbability) {
          const replyDelay = this.randomDelay(60, 2880);  // 1-48 hours
          setTimeout(() => this.sendReply(emailId), replyDelay * 60 * 1000);
        }
      }, openDelay * 60 * 1000);
    }
  }

  private randomDelay(min: number, max: number): number {
    return Math.floor(Math.random() * (max - min + 1)) + min;
  }

  async checkWarmupHealth(scheduleId: string): Promise<void> {
    const schedule = await db.warmupSchedules.findById(scheduleId);
    const todayLog = await db.warmupDailyLogs.findOne({
      where: {
        warmupScheduleId: scheduleId,
        logDate: new Date().toISOString().split('T')[0],
      },
    });

    if (!todayLog) return;

    // Check bounce rate
    if (todayLog.bounceRate > schedule.maxBounceRate) {
      await this.pauseWarmup(scheduleId, 'High bounce rate detected');
      return;
    }

    // Check spam rate
    if (todayLog.spamRate > schedule.maxSpamRate) {
      await this.pauseWarmup(scheduleId, 'Spam complaints detected');
      return;
    }

    // Calculate health score
    const healthScore = this.calculateHealthScore(todayLog);

    await db.warmupDailyLogs.update(todayLog.id, { healthScore });
  }

  private calculateHealthScore(log: any): number {
    let score = 100;

    // Penalize for bounces
    score -= log.bounceRate * 10;

    // Penalize for spam
    score -= log.spamRate * 50;

    // Reward for engagement
    score += (log.openRate / 100) * 20;
    score += (log.clickRate / 100) * 10;

    return Math.max(0, Math.min(100, Math.floor(score)));
  }

