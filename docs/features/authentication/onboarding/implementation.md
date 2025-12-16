---
title: "Onboarding Technical Implementation"
description: "Database schema, state management, and frontend components for onboarding"
last_modified_date: "2025-11-19"
level: "3"
persona: "Backend Developers, Frontend Developers"
---

# Onboarding Technical Implementation

Database schema, state management, and frontend components for PenguinMails onboarding.

## Onboarding State Management

```typescript
interface OnboardingState {
  userId: string;
  tenantId: string;

  // Progress tracking
  currentStep: number;
  totalSteps: number;
  completedSteps: string[];
  startedAt: Date;
  completedAt?: Date;

  // Checklist
  checklistItems: OnboardingChecklistItem[];
  checklistProgress: number; // 0-100

  // User preferences
  skipTutorials: boolean;
  dismissedTooltips: string[];
  watchedVideos: string[];

  // Milestones
  unlockedMilestones: string[];

  // Metadata
  lastActiveStep: string;
  lastActivityAt: Date;
}

interface OnboardingChecklistItem {
  id: string;
  label: string;
  description: string;
  completed: boolean;
  required: boolean;
  order: number;
  actionUrl?: string;
  completedAt?: Date;
}
```

## Database Schema

```sql
CREATE TABLE user_onboarding (
  id UUID PRIMARY KEY,
  user_id UUID NOT NULL REFERENCES users(id),
  tenant_id UUID NOT NULL REFERENCES tenants(id),

  -- Progress
  current_step INTEGER DEFAULT 1,
  completed_steps TEXT[] DEFAULT '{}',
  started_at TIMESTAMP DEFAULT NOW(),
  completed_at TIMESTAMP,

  -- Checklist
  checklist_progress INTEGER DEFAULT 0,
  checklist_items JSONB DEFAULT '[]',

  -- Preferences
  skip_tutorials BOOLEAN DEFAULT FALSE,
  dismissed_tooltips TEXT[] DEFAULT '{}',
  watched_videos TEXT[] DEFAULT '{}',

  -- Milestones
  unlocked_milestones TEXT[] DEFAULT '{}',

  -- Metadata
  last_active_step VARCHAR(100),
  last_activity_at TIMESTAMP DEFAULT NOW(),

  UNIQUE(user_id)
);

-- Onboarding analytics
CREATE TABLE onboarding_events (
  id UUID PRIMARY KEY,
  user_id UUID NOT NULL,
  event_type VARCHAR(100),
  event_data JSONB,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_onboarding_events_user ON onboarding_events(user_id, created_at);
```

## Onboarding Service

```typescript
class OnboardingService {
  async initializeOnboarding(userId: string, tenantId: string): Promise<OnboardingState> {
    const checklist = this.generateChecklist(userId, tenantId);

    const onboarding = await db.userOnboarding.create({
      userId,
      tenantId,
      currentStep: 1,
      checklistItems: checklist,
      checklistProgress: 0,
    });

    await this.trackEvent(userId, 'onboarding_started');
    return onboarding;
  }

  async completeStep(userId: string, step: string): Promise<void> {
    const onboarding = await db.userOnboarding.findByUser(userId);

    await db.userOnboarding.update(userId, {
      completedSteps: [...onboarding.completedSteps, step],
      lastActiveStep: step,
      lastActivityAt: new Date(),
    });

    await this.trackEvent(userId, 'onboarding_step_completed', { step });
    await this.checkMilestones(userId, step);
    await this.updateChecklist(userId, step);
  }

  async updateChecklist(userId: string, action: string): Promise<void> {
    const onboarding = await db.userOnboarding.findByUser(userId);
    const checklist = onboarding.checklistItems;

    const item = checklist.find(i => i.id === action);
    if (item && !item.completed) {
      item.completed = true;
      item.completedAt = new Date();

      const progress = this.calculateProgress(checklist);

      await db.userOnboarding.update(userId, {
        checklistItems: checklist,
        checklistProgress: progress,
      });

      if (progress === 100) {
        await this.completeOnboarding(userId);
      }
    }
  }

  private calculateProgress(checklist: OnboardingChecklistItem[]): number {
    const total = checklist.filter(i => i.required).length;
    const completed = checklist.filter(i => i.required && i.completed).length;
    return Math.round((completed / total) * 100);
  }

  async checkMilestones(userId: string, event: string): Promise<void> {
    const milestones = {
      'workspace_created': 'first_workspace',
      'domain_verified': 'first_domain',
      'email_sent': 'first_email',
      'email_opened': 'first_open',
    };

    const milestone = milestones[event];
    if (milestone) {
      await this.unlockMilestone(userId, milestone);
    }
  }

  async unlockMilestone(userId: string, milestone: string): Promise<void> {
    await db.userOnboarding.update(userId, {
      unlockedMilestones: db.raw('array_append(unlocked_milestones, ?)', [milestone]),
    });

    await this.trackEvent(userId, 'milestone_unlocked', { milestone });

    await notificationService.send({
      userId,
      type: 'milestone_unlocked',
      data: { milestone },
    });
  }
}
```

## Frontend Components

### Onboarding Wizard

```tsx
function OnboardingWizard() {
  const { onboarding, completeStep } = useOnboarding();

  const steps = [
    { id: 'workspace', component: WorkspaceSetup },
    { id: 'domain', component: DomainSetup },
    { id: 'payment', component: PaymentSetup },
    { id: 'infrastructure', component: InfrastructureSetup },
    { id: 'email-account', component: EmailAccountSetup },
    { id: 'complete', component: OnboardingComplete },
  ];

  const currentStepComponent = steps[onboarding.currentStep - 1]?.component;

  return (
    <div className="onboarding-wizard">
      <ProgressBar
        current={onboarding.currentStep}
        total={steps.length}
      />

      {currentStepComponent && React.createElement(currentStepComponent, {
        onComplete: () => completeStep(steps[onboarding.currentStep - 1].id),
      })}
    </div>
  );
}
```

### Persistent Checklist

```tsx
function OnboardingChecklist() {
  const { checklist, progress } = useOnboarding();
  const [collapsed, setCollapsed] = useState(false);

  if (progress === 100) return null;

  return (
    <aside className="onboarding-checklist">
      <header onClick={() => setCollapsed(!collapsed)}>
        <h3>Getting Started</h3>
        <ProgressBar value={progress} />
        <span>{progress}%</span>
      </header>

      {!collapsed && (
        <ul>
          {checklist.map(item => (
            <li key={item.id} className={item.completed ? 'completed' : ''}>
              {item.completed ? '✓' : '○'} {item.label}
            </li>
          ))}
        </ul>
      )}
    </aside>
  );
}
```

## Related Documentation

- [Onboarding Overview](/docs/features/authentication/onboarding/overview) - Main page
- [User Flow](/docs/features/authentication/onboarding/user-flow) - Step-by-step guide
- [Advanced Features](/docs/features/authentication/onboarding/advanced-features) - Checklists and milestones
