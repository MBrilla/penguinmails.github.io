---
title: "Lead Scoring Technical Implementation"
description: "Database schema, scoring engine architecture, background jobs, and event listeners for lead scoring"
last_modified_date: "2025-12-15"
level: "3"
status: "PLANNED"
roadmap_timeline: "Q1 2026"
priority: "High"
parent: lead-scoring
---

# Lead Scoring Technical Implementation

Technical details for implementing the lead scoring system.

## Database Schema

### Lead Scoring Models Table

```sql
CREATE TABLE lead_scoring_models (
  id UUID PRIMARY KEY,
  tenant_id UUID NOT NULL REFERENCES tenants(id),

  name VARCHAR(255) NOT NULL,
  description TEXT,

  -- Scoring rules (JSON configuration)
  scoring_rules JSONB NOT NULL,

  -- Decay settings
  decay_enabled BOOLEAN DEFAULT true,
  decay_rate DECIMAL(5,2) DEFAULT 5.0,  -- Percentage
  decay_interval_days INTEGER DEFAULT 30,

  -- Status
  is_active BOOLEAN DEFAULT TRUE,

  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);
```

### Contact Scores Table

```sql
CREATE TABLE contact_scores (
  id UUID PRIMARY KEY,
  contact_id UUID NOT NULL REFERENCES contacts(id),
  scoring_model_id UUID REFERENCES lead_scoring_models(id),

  -- Scores
  total_score INTEGER DEFAULT 0,
  engagement_score INTEGER DEFAULT 0,
  fit_score INTEGER DEFAULT 0,
  intent_score INTEGER DEFAULT 0,

  -- Metadata
  last_calculated_at TIMESTAMP DEFAULT NOW(),
  last_activity_at TIMESTAMP,
  previous_score INTEGER DEFAULT 0,  -- For tracking changes

  UNIQUE(contact_id, scoring_model_id)
);

CREATE INDEX idx_contact_scores_score ON contact_scores(total_score DESC);
CREATE INDEX idx_contact_scores_contact ON contact_scores(contact_id);
```

### Score Events Table

```sql
CREATE TABLE score_events (
  id UUID PRIMARY KEY,
  contact_id UUID NOT NULL REFERENCES contacts(id),

  -- Event details
  event_type VARCHAR(100),  -- email_opened, clicked_link, etc.
  event_data JSONB,

  -- Score change
  points_added INTEGER,
  score_before INTEGER,
  score_after INTEGER,

  -- Timestamp
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_score_events_contact ON score_events(contact_id, created_at);
CREATE INDEX idx_score_events_type ON score_events(event_type);
```

---

## Scoring Engine

### Type Definitions

```typescript
interface ScoringRule {
  action: string;
  points: number;
  conditions?: Record<string, any>;
  multiplier?: number;
  decays?: boolean;
}

interface ScoringModel {
  id: string;
  name: string;
  rules: ScoringRule[];
  decayEnabled: boolean;
  decayRate: number;
  decayIntervalDays: number;
}
```

### Core Scoring Engine

```typescript
class LeadScoringEngine {
  async scoreAction(
    contactId: string,
    action: string,
    metadata: Record<string, any> = {}
  ): Promise<number> {
    const contact = await db.contacts.findById(contactId);
    const model = await this.getScoringModel(contact.tenantId);
    const currentScore = await this.getCurrentScore(contactId);

    // Find matching rule
    const rule = model.rules.find(r => r.action === action);

    if (!rule) {
      return currentScore.totalScore;
    }

    // Calculate points
    let points = rule.points;

    // Apply conditions
    if (rule.conditions) {
      const meetsConditions = this.evaluateConditions(
        rule.conditions,
        metadata
      );
      if (!meetsConditions) {
        return currentScore.totalScore;
      }
    }

    // Apply multiplier for repeated actions
    if (rule.multiplier) {
      const recentActions = await this.countRecentActions(
        contactId,
        action,
        30
      );
      if (recentActions > 1) {
        points *= rule.multiplier;
      }
    }

    // Apply recency boost
    points = this.applyRecencyBoost(points, new Date());

    // Calculate new score (clamped 0-100)
    const newScore = Math.max(
      0,
      Math.min(100, currentScore.totalScore + points)
    );

    // Update score
    await this.updateScore(contactId, newScore, action, points);

    // Log event
    await db.scoreEvents.create({
      contactId,
      eventType: action,
      eventData: metadata,
      pointsAdded: points,
      scoreBefore: currentScore.totalScore,
      scoreAfter: newScore,
    });

    // Check for automation triggers
    await this.checkAutomationTriggers(
      contactId,
      currentScore.totalScore,
      newScore
    );

    return newScore;
  }

  private applyRecencyBoost(basePoints: number, actionDate: Date): number {
    const hoursSinceAction = differenceInHours(new Date(), actionDate);

    if (hoursSinceAction < 24) {
      return basePoints * 2.0;
    } else if (hoursSinceAction < 168) {  // 7 days
      return basePoints * 1.5;
    } else if (hoursSinceAction < 720) {  // 30 days
      return basePoints * 1.0;
    } else {
      return basePoints * 0.5;
    }
  }
}
```

### Score Decay Logic

```typescript
async applyDecay(contactId: string): Promise<void> {
  const contact = await db.contacts.findById(contactId);
  const model = await this.getScoringModel(contact.tenantId);

  if (!model.decayEnabled) {
    return;
  }

  const currentScore = await this.getCurrentScore(contactId);
  const daysSinceLastActivity = differenceInDays(
    new Date(),
    currentScore.lastActivityAt
  );

  if (daysSinceLastActivity < model.decayIntervalDays) {
    return;
  }

  // Calculate number of decay periods
  const decayPeriods = Math.floor(
    daysSinceLastActivity / model.decayIntervalDays
  );

  // Apply exponential decay
  let newScore = currentScore.totalScore;
  for (let i = 0; i < decayPeriods; i++) {
    newScore = newScore * (1 - model.decayRate / 100);
  }

  newScore = Math.max(0, Math.floor(newScore));

  if (newScore !== currentScore.totalScore) {
    await this.updateScore(
      contactId,
      newScore,
      'score_decay',
      newScore - currentScore.totalScore
    );
  }
}
```

### Automation Triggers

```typescript
private async checkAutomationTriggers(
  contactId: string,
  oldScore: number,
  newScore: number
): Promise<void> {
  const thresholds = [25, 50, 75];

  for (const threshold of thresholds) {
    if (oldScore < threshold && newScore >= threshold) {
      await this.triggerScoreAutomation(contactId, threshold);
    }
  }
}

private async triggerScoreAutomation(
  contactId: string,
  threshold: number
): Promise<void> {
  switch (threshold) {
    case 75:
      // Sales Qualified Lead
      await this.markAsSQLead(contactId);
      await this.notifySalesTeam(contactId);
      await this.addToSegment(contactId, 'hot-leads');
      break;

    case 50:
      // Marketing Qualified Lead
      await this.markAsMQLead(contactId);
      await this.addToSegment(contactId, 'warm-leads');
      break;

    case 25:
      // Warming up
      await this.addToSegment(contactId, 'nurture-leads');
      break;
  }
}
```

---

## Background Jobs

### Daily Decay Processing

```typescript
// Apply decay to all contacts daily
cron.schedule('0 3 * * *', async () => {  // 3 AM daily
  const contacts = await db.contacts.findAll({
    where: {
      isActive: true,
    },
  });

  const scoringEngine = new LeadScoringEngine();

  for (const contact of contacts) {
    await scoringQueue.add('apply-decay', {
      contactId: contact.id,
    });
  }
});

// Queue worker
scoringQueue.process('apply-decay', async (job) => {
  const { contactId } = job.data;
  const engine = new LeadScoringEngine();
  await engine.applyDecay(contactId);
});
```

### Score Recalculation

```typescript
async function recalculateScoresForContact(
  contactId: string
): Promise<void> {
  const engine = new LeadScoringEngine();

  // Recalculate fit score based on demographics
  const contact = await db.contacts.findById(contactId);
  const fitScore = await engine.calculateFitScore(contact);

  await db.contactScores.update(
    { where: { contactId } },
    { fitScore }
  );
}
```

---

## Event Listeners

### Email Event Handlers

```typescript
eventEmitter.on('email.opened', async (event) => {
  const engine = new LeadScoringEngine();
  await engine.scoreAction(event.contactId, 'email_opened', {
    emailId: event.emailId,
    campaignId: event.campaignId,
  });
});

eventEmitter.on('email.clicked', async (event) => {
  const engine = new LeadScoringEngine();

  // Check if clicked URL is high-intent
  const isHighIntent = event.url.includes('/pricing') ||
                       event.url.includes('/demo');

  const action = isHighIntent
    ? 'clicked_high_intent_link'
    : 'clicked_link';

  await engine.scoreAction(event.contactId, action, {
    url: event.url,
    emailId: event.emailId,
  });
});
```

### Contact Update Handler

```typescript
eventEmitter.on('contact.updated', async (event) => {
  // Recalculate fit score when demographics change
  await recalculateScoresForContact(event.contactId);
});
```

---

## Related Topics

- **[Quick Start Guide](./quick-start-guide)** - Basic scoring concepts
- **[Advanced Configuration](./advanced-scoring-configuration)** - Customize rules
- **[Score Analytics](./score-analytics)** - Track effectiveness

---

**[Back to Lead Scoring Overview](./README)**
