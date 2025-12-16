---
title: "Migrator & Worker Processing"
description: "Job migrator process and worker job retrieval mechanisms"
last_modified_date: "2025-11-19"
level: 2
persona: "System Engineers"
keywords: ["migrator", "worker", "job-retrieval", "queue-processing"]
---

# Migrator & Worker Processing

The migrator service bridges PostgreSQL durability with Redis speed, while workers efficiently retrieve and process jobs using blocking operations.

## Queuer Process (Migrator)

### Core Migration Logic

The migrator service continuously moves jobs from PostgreSQL to Redis:

```pseudo
class JobMigrator {
  private isRunning = false
  private batchSize = 100
  private migrationInterval = 5000 // 5 seconds

  async start() {
    this.isRunning = true
    log("Job migrator started")

    while (this.isRunning) {
      try {
        await this.migrateReadyJobs()
        await this.delay(this.migrationInterval)
      } catch (error) {
        logError("Migration error", error)
        await this.delay(10000) // Wait 10 seconds on error
      }
    }
  }

  private async migrateReadyJobs() {
    // Query PostgreSQL for ready jobs
    readyJobs = await database.jobs.findMany({
      where: {
        status: 'queued',
        run_at: { lte: new Date() }
      },
      orderBy: [
        { priority: 'asc' },      // Higher priority first
        { created_at: 'asc' }     // FIFO within same priority
      ],
      take: this.batchSize
    })

    if (readyJobs.length === 0) return

    log(`Migrating ${readyJobs.length} jobs to Redis`)

    for (job in readyJobs) {
      try {
        await this.migrateJob(job)
      } catch (error) {
        logError(`Failed to migrate job ${job.id}`, error)
      }
    }
  }

  private async migrateJob(job) {
    // Determine target Redis queue
    queueName = this.getQueueName(job.priority, job.queue_name)

    // Prepare Redis payload
    redisPayload = {
      id: job.id,
      queue_name: job.queue_name,
      priority: job.priority,
      payload: job.payload,
      created_at: job.created_at,
      attempt_count: job.attempt_count,
      max_attempts: job.max_attempts
    }

    // Add to Redis queue (left push for FIFO)
    await redis.lpush(queueName, JSON.stringify(redisPayload))

    // Track in Redis hash for status monitoring
    await redis.hset(`job:${job.id}`, {
      status: 'migrated',
      queued_at: new Date().toISOString(),
      attempt_count: job.attempt_count.toString()
    })

    // Prevent duplicates
    await redis.sadd(`recent_jobs:${job.queue_name}`, job.id)
    await redis.expire(`recent_jobs:${job.queue_name}`, 3600)

    // Update PostgreSQL status
    await database.jobs.update({
      where: { id: job.id },
      data: {
        status: 'migrated_to_redis',
        updated_at: new Date()
      }
    })
  }
}
```

## Job Retrieval for Workers

### Blocking Pop with Priority

Workers use blocking pop (BRPOP) to efficiently retrieve jobs:

```pseudo
async function workerJobRetrieval(workerId) {
  queues = [
    'queue:email:processing:high',     // Highest priority first
    'queue:email:processing',
    'queue:email:processing:low',
    'queue:email-sending:high',
    'queue:email-sending',
    'queue:email-sending:low',
    'queue:warmup:process',
    'queue:bounce:process'
  ]

  while (isRunning) {
    try {
      // Blocking pop with timeout
      result = await redis.brpop(queues, 0)

      if (!result) continue

      [queueName, jobData] = result
      job = JSON.parse(jobData)

      await processJob(job, queueName, workerId)

    } catch (error) {
      logError(`Worker ${workerId} retrieval error`, error)
      await this.delay(1000)
    }
  }
}
```

### Job Processing Workflow

Complete job processing flow:

```pseudo
async function processJob(job, queueName, workerId) {
  try {
    // Update PostgreSQL status
    await database.jobs.update({
      where: { id: job.id },
      data: {
        status: 'running',
        started_at: new Date(),
        worker_id: workerId,
        updated_at: new Date()
      }
    })

    // Update Redis tracking
    await redis.hset(`job:${job.id}`, {
      status: 'processing',
      worker_id: workerId,
      started_at: new Date().toISOString()
    })

    // Execute job based on queue type
    if (queueName.includes('email:processing')) {
      await executeEmailProcessing(job.payload)
    } else if (queueName.includes('email-sending')) {
      await executeEmailSending(job.payload)
    } else if (queueName.includes('warmup')) {
      await executeWarmup(job.payload)
    } else if (queueName.includes('bounce')) {
      await executeBounceProcessing(job.payload)
    }

    // Mark as completed
    await database.jobs.update({
      where: { id: job.id },
      data: {
        status: 'completed',
        completed_at: new Date(),
        updated_at: new Date()
      }
    })

    await redis.hset(`job:${job.id}`, {
      status: 'completed',
      completed_at: new Date().toISOString()
    })

    log(`Job ${job.id} completed successfully`)

  } catch (error) {
    await handleJobFailure(job, error)
  }
}
```

## Related Documentation

- **[Overview](overview)** - Queue management overview and naming conventions
- **[Data Structures](data-structures)** - Redis data structures and job payloads
- **[Error Handling & Monitoring](error-handling)** - Retry logic and performance optimization
