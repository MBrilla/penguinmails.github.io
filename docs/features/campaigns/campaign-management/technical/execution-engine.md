---
title: "Campaign Execution Engine"
description: "TypeScript implementation of the campaign execution engine for processing steps and sending emails"
last_modified_date: "2025-01-20"
level: 3
persona: Developers
keywords: execution engine, campaign processing, TypeScript, email sending, personalization
---

# Campaign Execution Engine

The CampaignExecutionEngine class manages campaign lifecycle operations including launching, contact enrollment, and step-by-step processing. The engine handles different step types and calculates optimal send times based on contact behavior.

## Core Interfaces

```typescript
interface Campaign {
  id: string;
  tenantId: string;
  name: string;
  campaignType: 'broadcast' | 'drip' | 'triggered' | 'ab_test';
  status: 'draft' | 'scheduled' | 'active' | 'paused' | 'completed';
  segmentId: string;
  fromEmail: string;
  fromName: string;
  sendStrategy: 'immediate' | 'scheduled' | 'optimized';
  timezone: string;
}

interface CampaignStep {
  id: string;
  campaignId: string;
  stepOrder: number;
  stepType: 'email' | 'wait' | 'condition' | 'action';
  templateId?: string;
  subjectLine?: string;
  emailContent?: string;
  waitDuration?: { value: number; unit: string };
  conditionRules?: any;
  trueNextStepId?: string;
  falseNextStepId?: string;
}
```

## Campaign Launch

The launchCampaign method transitions a campaign from draft to active status, enrolling all contacts from the target segment.

```typescript
class CampaignExecutionEngine {
  async launchCampaign(campaignId: string): Promise<void> {
    const campaign = await db.campaigns.findById(campaignId);

    if (campaign.status !== 'draft' && campaign.status !== 'scheduled') {
      throw new Error('Campaign must be in draft or scheduled status to launch');
    }

    // 1. Get contacts from segment
    const contacts = await db.segments.getContacts(campaign.segmentId);

    // 2. Enroll contacts in campaign
    await this.enrollContacts(campaignId, contacts);

    // 3. Update campaign status
    await db.campaigns.update(campaignId, {
      status: 'active',
      launchedAt: new Date(),
    });

    // 4. Start processing first step
    await this.processNextSteps(campaignId);

    logger.info(`Campaign ${campaignId} launched with ${contacts.length} contacts`);
  }

  private async enrollContacts(
    campaignId: string,
    contacts: Contact[]
  ): Promise<void> {
    const campaign = await db.campaigns.findById(campaignId);
    const firstStep = await db.campaignSteps.findFirst(campaignId);

    const enrollments = contacts.map(contact => ({
      campaignId,
      contactId: contact.id,
      status: 'enrolled',
      currentStepId: firstStep.id,
      enrolledAt: new Date(),
      nextEmailScheduledAt: this.calculateNextSendTime(campaign, contact),
    }));

    await db.campaignContacts.createMany(enrollments);

    await db.campaigns.update(campaignId, {
      totalContacts: contacts.length,
    });
  }
}
```

## Step Processing

The engine processes contacts who are ready for their next step, handling each step type appropriately.

```typescript
async processNextSteps(campaignId: string): Promise<void> {
  const readyContacts = await db.campaignContacts.findWhere({
    campaignId,
    status: 'active',
    nextEmailScheduledAt: { lte: new Date() },
  });

  for (const enrollment of readyContacts) {
    await this.processContactStep(enrollment);
  }
}

private async processContactStep(
  enrollment: CampaignContact
): Promise<void> {
  const step = await db.campaignSteps.findById(enrollment.currentStepId);

  switch (step.stepType) {
    case 'email':
      await this.sendCampaignEmail(enrollment, step);
      break;

    case 'wait':
      await this.processWaitStep(enrollment, step);
      break;

    case 'condition':
      await this.processConditionalStep(enrollment, step);
      break;

    case 'action':
      await this.processActionStep(enrollment, step);
      break;
  }
}
```

## Email Sending

The sendCampaignEmail method renders personalized content and queues emails for delivery.

```typescript
private async sendCampaignEmail(
  enrollment: CampaignContact,
  step: CampaignStep
): Promise<void> {
  const contact = await db.contacts.findById(enrollment.contactId);
  const campaign = await db.campaigns.findById(enrollment.campaignId);

  // 1. Render email with personalization
  const renderedContent = await this.renderEmailContent(
    step.emailContent,
    contact
  );
  const renderedSubject = await this.renderEmailContent(
    step.subjectLine,
    contact
  );

  // 2. Queue email for sending
  await emailQueue.add('send-campaign-email', {
    campaignId: campaign.id,
    stepId: step.id,
    contactId: contact.id,
    from: `${campaign.fromName} <${campaign.fromEmail}>`,
    to: contact.email,
    subject: renderedSubject,
    html: renderedContent,
    trackOpens: campaign.trackOpens,
    trackClicks: campaign.trackClicks,
  });

  // 3. Update enrollment progress
  const nextStep = await this.getNextStep(step);

  await db.campaignContacts.update(enrollment.id, {
    currentStepId: nextStep?.id,
    stepsCompleted: enrollment.stepsCompleted + 1,
    lastEmailSentAt: new Date(),
    nextEmailScheduledAt: nextStep
      ? this.calculateNextStepTime(nextStep, contact)
      : null,
    status: nextStep ? 'active' : 'completed',
    completedAt: nextStep ? null : new Date(),
  });

  // 4. Update step analytics
  await db.campaignSteps.increment(step.id, 'emailsSent', 1);
}
```

## Conditional Logic

Conditions evaluate contact engagement and attributes to determine branching paths.

```typescript
private async processConditionalStep(
  enrollment: CampaignContact,
  step: CampaignStep
): Promise<void> {
  const contact = await db.contacts.findById(enrollment.contactId);

  const conditionMet = await this.evaluateCondition(
    step.conditionRules,
    contact,
    enrollment
  );

  const nextStepId = conditionMet
    ? step.trueNextStepId
    : step.falseNextStepId;

  if (!nextStepId) {
    await db.campaignContacts.update(enrollment.id, {
      status: 'completed',
      completedAt: new Date(),
    });
    return;
  }

  const nextStep = await db.campaignSteps.findById(nextStepId);

  await db.campaignContacts.update(enrollment.id, {
    currentStepId: nextStepId,
    nextEmailScheduledAt: this.calculateNextStepTime(nextStep, contact),
  });
}

private async evaluateCondition(
  rules: any,
  contact: Contact,
  enrollment: CampaignContact
): Promise<boolean> {
  const emailHistory = await db.emails.findWhere({
    contactId: contact.id,
    campaignId: enrollment.campaignId,
  });

  switch (rules.type) {
    case 'email_opened':
      return emailHistory.some(e => e.opened);

    case 'email_clicked':
      return emailHistory.some(e => e.clicked);

    case 'email_replied':
      return emailHistory.some(e => e.replied);

    case 'contact_attribute':
      return this.evaluateAttributeCondition(rules, contact);

    case 'lead_score':
      return contact.leadScore >= rules.threshold;

    default:
      return false;
  }
}
```

## Send Time Optimization

The engine calculates optimal send times based on campaign strategy and contact behavior patterns.

```typescript
private calculateNextSendTime(
  campaign: Campaign,
  contact: Contact
): Date {
  const now = new Date();

  switch (campaign.sendStrategy) {
    case 'immediate':
      return now;

    case 'scheduled':
      return campaign.startDate || now;

    case 'optimized':
      return this.calculateOptimalSendTime(contact);

    default:
      return now;
  }
}

private calculateOptimalSendTime(contact: Contact): Date {
  const historicalOpens = contact.emailEngagement?.typicalOpenTimes || [];

  if (historicalOpens.length > 0) {
    const optimalHour = this.getMostCommonHour(historicalOpens);

    const tomorrow = new Date();
    tomorrow.setDate(tomorrow.getDate() + 1);
    tomorrow.setHours(optimalHour, 0, 0, 0);

    return tomorrow;
  }

  const defaultTime = new Date();
  defaultTime.setHours(9, 0, 0, 0);
  return defaultTime;
}
```

## Related Documentation

- [Database Schema](./database-schema.md) - Table structures
- [Background Jobs](./background-jobs.md) - Scheduled processing
- [Personalization System](/docs/features/campaigns/personalization-system) - Email personalization
