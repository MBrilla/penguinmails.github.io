---
title: "Operations & Scaling"
description: "Error handling, retry logic, worker scaling, and health monitoring"
last_modified_date: "2025-01-15"
level: "2"
persona: "Backend Engineers"
---

# Operations & Scaling

Error handling, retry logic, horizontal scaling strategies, and health monitoring for worker processes.

## Error Handling and Retry Logic

### Job Failure Handling

```pseudo
async function handleJobFailure(job: Job, error: Error) {
  const { id, max_attempts, attempt_count = 0 } = job
  const attempts = attempt_count + 1

  try {
    if (attempts < max_attempts) {
      // Schedule retry with exponential backoff
      const delay = this.calculateBackoffDelay(attempts)

      log(`Scheduling retry ${attempts}/${max_attempts} for job ${id} in ${delay}ms`)

      await this.delay(delay)

      // Re-queue the job with updated attempt count
      await this.requeueJob(job, attempts)

      // Update tracking
      await this.redis.hset(`job:${id}`, {
        status: 'retry_scheduled',
        attempt_count: attempts.toString(),
        retry_delay: delay.toString()
      })

    } else {
      // Max attempts reached - mark as failed permanently
      await this.markJobFailed(job, error)
      await this.moveToDeadLetterQueue(job, error)
    }

    // Log failure
    await this.logJobExecution(id, 'failed', error.message)

  } catch (logError) {
    logError(`Failed to log job failure for ${id}`, logError)
  }
}
```

### Exponential Backoff Calculation

```pseudo
function calculateBackoffDelay(attemptNumber: number): number {
  // Exponential backoff: 2^attempt * base_delay
  const baseDelay = 1000    // 1 second base
  const maxDelay = 300000   // 5 minutes max

  const delay = Math.pow(2, attemptNumber) * baseDelay
  return Math.min(delay, maxDelay)
}

// Example delays:
// Attempt 1: 2s
// Attempt 2: 4s
// Attempt 3: 8s
// Attempt 4: 16s
// Attempt 5: 32s
// Attempt 6: 64s
// Attempt 7: 128s
// Attempt 8: 256s
// Attempt 9: 300s (max)
```

## Worker Scaling and Load Distribution

### Horizontal Scaling Strategy

```pseudo
// Auto-scaling based on queue depth
async function monitorScaling() {
  const queueDepths = await this.getQueueDepths()
  const activeWorkers = await this.getActiveWorkerCount()

  for (const [queueName, depth] of Object.entries(queueDepths)) {
    // Scale up if queue is backing up
    if (depth > HIGH_WATER_MARK && activeWorkers < MAX_WORKERS) {
      await this.scaleUpWorkers(queueName)
    }

    // Scale down if queue is empty
    if (depth < LOW_WATER_MARK && activeWorkers > MIN_WORKERS) {
      await this.scaleDownWorkers(queueName)
    }
  }
}

function getQueueList(): string[] {
  return [
    'queue:email:processing:high',    // Highest priority
    'queue:email:processing',
    'queue:email:processing:low',
    'queue:email-sending:high',
    'queue:email-sending',
    'queue:email-sending:low',
    'queue:warmup:process',
    'queue:bounce:process',
    'queue:analytics:daily-aggregate',
    'queue:webhook:process'
  ]
}
```

### Graceful Shutdown

```pseudo
async function gracefulShutdown() {
  log('Initiating graceful shutdown')

  // Stop accepting new jobs
  this.isRunning = false

  // Wait for current jobs to complete
  const maxWaitTime = 30000 // 30 seconds
  const startTime = Date.now()

  while (this.hasActiveJobs() && (Date.now() - startTime) < maxWaitTime) {
    log('Waiting for active jobs to complete...')
    await this.delay(1000)
  }

  // Force shutdown if jobs are taking too long
  if (this.hasActiveJobs()) {
    log('Force shutdown - jobs exceeded time limit')
  }

  await this.cleanup()
  process.exit(0)
}
```

## Monitoring and Health Checks

### Worker Health Monitoring

```pseudo
async function getWorkerHealth() {
  return {
    worker_id: this.workerId,
    status: this.isRunning ? 'healthy' : 'stopped',
    uptime: process.uptime(),
    memory_usage: process.memoryUsage(),
    jobs_processed: this.getJobsProcessedCount(),
    jobs_failed: this.getJobsFailedCount(),
    average_processing_time: this.getAverageProcessingTime(),
    last_heartbeat: new Date().toISOString()
  }
}
```

## Summary

Worker processes provide:

- **Reliable Job Processing**: Comprehensive error handling and retry logic
- **Horizontal Scalability**: Stateless design supports unlimited worker instances
- **Priority Handling**: Multi-queue consumption with proper priority ordering
- **Fault Tolerance**: Graceful shutdown and failure recovery mechanisms
- **Performance Optimization**: Batch processing and memory management
- **Monitoring Integration**: Real-time health and performance metrics

This architecture ensures robust, scalable job processing that can handle varying workloads while maintaining system reliability and performance.

---

**Related Documents:**
- [Overview](overview.md)
- [Job Processing](job-processing.md)
- [Specialized Handlers](specialized-handlers.md)
- [Management](/docs/implementation-technical/database-infrastructure/queue/management)
