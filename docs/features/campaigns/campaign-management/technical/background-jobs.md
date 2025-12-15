---
title: "Background Jobs"
description: "Scheduled background jobs for campaign processing, analytics aggregation, and auto-completion"
last_modified_date: "2025-01-20"
level: 3
persona: Developers
keywords: background jobs, cron, scheduled tasks, analytics aggregation, campaign processing
---

# Campaign Background Jobs

The campaign system relies on scheduled background jobs to process steps, aggregate analytics, and manage campaign lifecycle. Jobs run on cron schedules to ensure timely processing without blocking user operations.

## Step Processing Job

The primary processing job runs every 5 minutes to advance contacts through campaign steps. This frequency balances responsiveness with system load.

```typescript
// Process campaign steps every 5 minutes
cron.schedule('*/5 * * * *', async () => {
  const activeCampaigns = await db.campaigns.findWhere({
    status: 'active',
  });

  const engine = new CampaignExecutionEngine();

  for (const campaign of activeCampaigns) {
    try {
      await engine.processNextSteps(campaign.id);
    } catch (error) {
      logger.error(`Error processing campaign ${campaign.id}:`, error);
    }
  }
});
```

The job retrieves all active campaigns and invokes the execution engine for each. Error handling ensures that failures in one campaign do not affect others. Logging captures errors for troubleshooting.

## Analytics Aggregation Job

Daily analytics aggregation runs at 2 AM to compile campaign metrics from the previous day. Aggregated data powers dashboard displays and reporting features.

```typescript
// Aggregate campaign analytics daily
cron.schedule('0 2 * * *', async () => {  // 2 AM daily
  const yesterday = new Date();
  yesterday.setDate(yesterday.getDate() - 1);
  yesterday.setHours(0, 0, 0, 0);

  const activeCampaigns = await db.campaigns.findWhere({
    status: ['active', 'completed'],
  });

  for (const campaign of activeCampaigns) {
    // Aggregate metrics for yesterday
    const metrics = await aggregateCampaignMetrics(campaign.id, yesterday);

    await db.campaignAnalytics.upsert({
      campaignId: campaign.id,
      date: yesterday,
      ...metrics,
    });
  }
});
```

The upsert operation handles both new records and updates to existing data. Both active and completed campaigns receive analytics updates to ensure comprehensive historical data.

## Auto-Completion Job

The hourly completion check automatically transitions campaigns to completed status when all contacts finish their journey.

```typescript
// Auto-complete finished campaigns
cron.schedule('0 * * * *', async () => {  // Every hour
  const campaigns = await db.campaigns.findWhere({
    status: 'active',
  });

  for (const campaign of campaigns) {
    const activeContacts = await db.campaignContacts.count({
      campaignId: campaign.id,
      status: 'active',
    });

    if (activeContacts === 0) {
      await db.campaigns.update(campaign.id, {
        status: 'completed',
        completedAt: new Date(),
      });

      logger.info(`Campaign ${campaign.id} auto-completed`);
    }
  }
});
```

This job prevents campaigns from remaining in active status indefinitely after all contacts complete their sequences. The completed timestamp enables accurate duration reporting.

## Job Scheduling Summary

| Job | Schedule | Purpose |
|-----|----------|---------|
| Step Processing | Every 5 minutes | Advance contacts through campaign steps |
| Analytics Aggregation | 2 AM daily | Compile previous day metrics |
| Auto-Completion | Every hour | Mark finished campaigns as completed |

## Error Handling

Each job implements try-catch blocks to isolate failures. Errors log with campaign identifiers for debugging. Failed campaigns continue processing in subsequent job runs, providing automatic retry behavior.

## Performance Considerations

Step processing scales with the number of active campaigns and ready contacts. During high-volume periods, consider:

- Increasing job frequency for faster processing
- Implementing batch processing within campaigns
- Using database connection pooling
- Monitoring job execution time and queue depth

## Related Documentation

- [Execution Engine](./execution-engine.md) - Step processing logic
- [Database Schema](./database-schema.md) - Analytics tables
- [Monitoring and Observability](/docs/operations/analytics/operations-management/environment-release-management/monitoring-observability.md) - Job monitoring
