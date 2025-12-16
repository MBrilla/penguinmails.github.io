---
title: "Job Processing Workflow"
description: "Core job processing logic, execution workflow, and job type routing"
last_modified_date: "2025-01-15"
level: "2"
persona: "Backend Engineers"
---

# Job Processing Workflow

Core processing logic for consuming and executing jobs from Redis queues, including status updates and job type routing.

## Core Processing Logic

```pseudo
async function processJob(job: Job, queueName: string) {
  const startTime = Date.now()

  try {
    log(`Processing job ${job.id} from ${queueName}`)

    // Update job status in PostgreSQL
    await this.updateJobStatus(job.id, 'running', {
      started_at: new Date(),
      worker_id: this.workerId
    })

    // Update Redis tracking for real-time monitoring
    await this.redis.hset(`job:${job.id}`, {
      status: 'processing',
      worker_id: this.workerId,
      started_at: new Date().toISOString()
    })

    // Execute job based on queue type
    const result = await this.executeJobLogic(job.payload, queueName)

    // Mark as completed
    await this.markJobCompleted(job.id, result)

    // Log successful execution
    await this.logJobExecution(job.id, 'success', null, startTime)

    log(`Job ${job.id} completed successfully in ${Date.now() - startTime}ms`)

  } catch (error) {
    await this.handleJobFailure(job, error)
  }
}
```

## Job Execution by Queue Type

```pseudo
async function executeJobLogic(payload: JobPayload, queueName: string) {
  switch (true) {
    case queueName.includes('email:processing'):
      return await this.processIncomingEmail(payload)

    case queueName.includes('email-sending'):
      return await this.processEmailSending(payload)

    case queueName.includes('warmup'):
      return await this.processWarmupJob(payload)

    case queueName.includes('bounce'):
      return await this.processBounceJob(payload)

    case queueName.includes('analytics'):
      return await this.processAnalyticsJob(payload)

    case queueName.includes('webhook'):
      return await this.processWebhookJob(payload)

    default:
      throw new Error(`Unknown queue type: ${queueName}`)
  }
}
```

## Job Status Management

### Status Updates

```pseudo
async function updateJobStatus(jobId: string, status: JobStatus, additionalData: any = {}) {
  const updateData = {
    status,
    updated_at: new Date(),
    ...additionalData
  }

  // Update PostgreSQL
  await this.database.jobs.update({
    where: { id: jobId },
    data: updateData
  })

  // Update Redis for real-time monitoring
  await this.redis.hset(`job:${jobId}`, {
    status: status,
    updated_at: new Date().toISOString(),
    ...additionalData
  })
}

async function markJobCompleted(jobId: string, result: any) {
  await this.updateJobStatus(jobId, 'completed', {
    completed_at: new Date(),
    result: JSON.stringify(result)
  })

  await this.redis.hset(`job:${jobId}`, {
    status: 'completed',
    completed_at: new Date().toISOString(),
    result: JSON.stringify(result)
  })
}
```

## Batch Processing

```pseudo
// Process multiple jobs in batch for efficiency
async function processBatch(jobs: Job[]) {
  const results = []

  // Process jobs concurrently with limits
  const batchSize = 5  // Max concurrent jobs
  const batches = chunk(jobs, batchSize)

  for (const batch of batches) {
    const batchPromises = batch.map(async (job) => {
      try {
        return await this.processJob(job, job.queue_name)
      } catch (error) {
        logError(`Batch job ${job.id} failed`, error)
        return { success: false, error: error.message }
      }
    })

    const batchResults = await Promise.all(batchPromises)
    results.push(...batchResults)

    // Small delay between batches to prevent overwhelming
    await this.delay(100)
  }

  return results
}
```

---

**Related Documents:**
- [Overview](overview.md)
- [Specialized Handlers](specialized-handlers.md)
- [Operations & Scaling](operations.md)
