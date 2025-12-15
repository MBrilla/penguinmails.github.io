---
title: "API Endpoints"
description: "REST API endpoints for campaign management including CRUD operations and lifecycle actions"
last_modified_date: "2025-01-20"
level: 3
persona: Developers
keywords: API endpoints, REST, campaign management, CRUD, TypeScript
---

# Campaign API Endpoints

The campaign management system exposes REST API endpoints for creating, managing, and monitoring campaigns. All endpoints require authentication and respect tenant isolation.

## Create Campaign

Creates a new campaign in draft status.

```typescript
router.post('/api/campaigns', async (req, res) => {
  const { name, campaignType, segmentId, fromEmail, fromName } = req.body;

  const campaign = await db.campaigns.create({
    tenantId: req.user.tenantId,
    workspaceId: req.user.workspaceId,
    name,
    campaignType,
    segmentId,
    fromEmail,
    fromName,
    status: 'draft',
    createdBy: req.user.id,
  });

  res.json(campaign);
});
```

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| name | string | Yes | Campaign display name |
| campaignType | string | Yes | broadcast, drip, triggered, ab_test |
| segmentId | UUID | Yes | Target audience segment |
| fromEmail | string | Yes | Sender email address |
| fromName | string | Yes | Sender display name |

## Get Campaign Details

Retrieves campaign with steps and latest analytics.

```typescript
router.get('/api/campaigns/:id', async (req, res) => {
  const campaign = await db.campaigns.findById(req.params.id);
  const steps = await db.campaignSteps.findWhere({
    campaignId: req.params.id,
  });
  const analytics = await db.campaignAnalytics.findLatest(req.params.id);

  res.json({
    campaign,
    steps,
    analytics,
  });
});
```

## Launch Campaign

Transitions campaign from draft to active and enrolls contacts.

```typescript
router.post('/api/campaigns/:id/launch', async (req, res) => {
  const engine = new CampaignExecutionEngine();
  await engine.launchCampaign(req.params.id);

  res.json({ success: true });
});
```

Launch validates campaign status before proceeding. Campaigns must be in draft or scheduled status to launch.

## Pause Campaign

Pauses an active campaign, halting all email delivery.

```typescript
router.post('/api/campaigns/:id/pause', async (req, res) => {
  await db.campaigns.update(req.params.id, {
    status: 'paused',
  });

  await db.campaignContacts.updateWhere(
    { campaignId: req.params.id, status: 'active' },
    { status: 'paused' }
  );

  res.json({ success: true });
});
```

Both the campaign and all active contact enrollments transition to paused status. Scheduled emails remain in place but do not send until resumed.

## Resume Campaign

Resumes a paused campaign.

```typescript
router.post('/api/campaigns/:id/resume', async (req, res) => {
  await db.campaigns.update(req.params.id, {
    status: 'active',
  });

  await db.campaignContacts.updateWhere(
    { campaignId: req.params.id, status: 'paused' },
    { status: 'active' }
  );

  res.json({ success: true });
});
```

Contacts continue from their current step upon resumption. The background processing job picks them up in the next cycle.

## Get Campaign Analytics

Retrieves analytics for a date range with per-step breakdown.

```typescript
router.get('/api/campaigns/:id/analytics', async (req, res) => {
  const { startDate, endDate } = req.query;

  const analytics = await db.campaignAnalytics.findWhere({
    campaignId: req.params.id,
    date: {
      gte: new Date(startDate),
      lte: new Date(endDate),
    },
  });

  const stepAnalytics = await db.campaignSteps.findWhere({
    campaignId: req.params.id,
  });

  res.json({
    overall: analytics,
    byStep: stepAnalytics,
  });
});
```

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| startDate | ISO date | Yes | Analytics period start |
| endDate | ISO date | Yes | Analytics period end |

## Clone Campaign

Creates a copy of an existing campaign with all steps.

```typescript
router.post('/api/campaigns/:id/clone', async (req, res) => {
  const original = await db.campaigns.findById(req.params.id);
  const steps = await db.campaignSteps.findWhere({
    campaignId: req.params.id,
  });

  // Create new campaign
  const cloned = await db.campaigns.create({
    ...original,
    id: undefined,
    name: `${original.name} (Copy)`,
    status: 'draft',
    createdAt: new Date(),
    launchedAt: null,
    completedAt: null,
  });

  // Clone steps
  for (const step of steps) {
    await db.campaignSteps.create({
      ...step,
      id: undefined,
      campaignId: cloned.id,
    });
  }

  res.json(cloned);
});
```

Cloned campaigns start in draft status regardless of the original campaign's status. Step analytics reset to zero on the clone.

## Endpoint Summary

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | /api/campaigns | Create new campaign |
| GET | /api/campaigns/:id | Get campaign details |
| POST | /api/campaigns/:id/launch | Launch campaign |
| POST | /api/campaigns/:id/pause | Pause active campaign |
| POST | /api/campaigns/:id/resume | Resume paused campaign |
| GET | /api/campaigns/:id/analytics | Get analytics |
| POST | /api/campaigns/:id/clone | Clone campaign |

## Related Documentation

- [Execution Engine](./execution-engine.md) - Launch and processing logic
- [Database Schema](./database-schema.md) - Data structures
- [API Documentation](/docs/implementation-technical/api/tenant-api/campaigns) - Full API reference
