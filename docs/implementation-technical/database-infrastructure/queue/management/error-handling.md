---
title: "Error Handling & Monitoring"
description: "Retry logic, dead letter queue management, and performance optimization"
last_modified_date: "2025-11-19"
level: 2
persona: "System Engineers"
keywords: ["error-handling", "retry-logic", "dead-letter-queue", "monitoring", "performance"]
---

# Error Handling & Monitoring

Comprehensive error handling, retry mechanisms, and performance optimization ensure reliable job processing.

## Retry Logic and Error Handling

### Exponential Backoff Strategy

```pseudo
async function handleJobFailure(job, error) {
  attempts = job.attempt_count + 1

  if (attempts < job.max_attempts) {
    // Calculate exponential backoff delay
    delay = Math.pow(2, attempts) * 1000  // 2s, 4s, 8s, etc.

    await delay(delay)

    // Re-queue job with updated attempt count
    updatedJob = {
      ...job,
      attempt_count: attempts
    }

    queueName = determineQueueName(job.priority, job.queue_name)
    await redis.lpush(queueName, JSON.stringify(updatedJob))

    // Update database
    await database.jobs.update({
      where: { id: job.id },
      data: {
        attempt_count: attempts,
        updated_at: new Date()
      }
    })

    await redis.hset(`job:${job.id}`, {
      status: 'retry_scheduled',
      attempt_count: attempts.toString(),
      retry_delay: delay.toString()
    })

  } else {
    // Max attempts reached - mark as failed
    await database.jobs.update({
      where: { id: job.id },
      data: {
        status: 'failed',
        failed_at: new Date(),
        last_error_message: error.message,
        updated_at: new Date()
      }
    })

    await redis.hset(`job:${job.id}`, {
      status: 'failed',
      error: error.message,
      failed_at: new Date().toISOString()
    })

    // Move to dead letter queue
    await moveToDeadLetterQueue(job, error)
  }
}
```

## Dead Letter Queue Management

```pseudo
async function moveToDeadLetterQueue(job, error) {
  deadLetterJob = {
    id: job.id,
    queue_name: job.queue_name,
    payload: job.payload,
    error: error.message,
    failed_at: new Date().toISOString(),
    attempt_count: job.attempt_count,
    original_priority: job.priority
  }

  // Add to dead letter queue
  await redis.lpush('deadletter:queue', JSON.stringify(deadLetterJob))

  // Mark as processed in dead letter tracking
  await redis.set(`deadletter:${job.id}`, 'moved', 'EX', 604800) // 7 days
}

async function replayDeadLetterJob(jobId) {
  // Get job from dead letter queue
  jobData = await redis.lpop('deadletter:queue')
  if (!jobData) {
    throw new Error('No jobs in dead letter queue')
  }

  deadLetterJob = JSON.parse(jobData)

  // Reset job in PostgreSQL
  await database.jobs.update({
    where: { id: jobId },
    data: {
      status: 'queued',
      attempt_count: 0,
      last_error_message: null,
      failed_at: null,
      updated_at: new Date()
    }
  })

  // Re-queue for processing
  queueName = determineQueueName(deadLetterJob.original_priority, deadLetterJob.queue_name)
  await redis.lpush(queueName, JSON.stringify({
    ...deadLetterJob.payload,
    id: jobId,
    attempt_count: 0
  }))

  return { success: true, job_id: jobId }
}
```

## Performance Optimization

### Batch Operations

```pseudo
// Batch migration for efficiency
async function batchMigrateJobs(jobs) {
  pipeline = redis.pipeline()

  for (job in jobs) {
    queueName = determineQueueName(job.priority, job.queue_name)
    redisPayload = createRedisPayload(job)

    pipeline.lpush(queueName, JSON.stringify(redisPayload))
    pipeline.hset(`job:${job.id}`, {
      status: 'migrated',
      queued_at: new Date().toISOString()
    })
  }

  // Execute all operations in single round trip
  await pipeline.exec()
}
```

### Memory Management

```pseudo
// Redis memory optimization
async function optimizeRedisMemory() {
  // Get Redis memory info
  memoryInfo = await redis.info('memory')

  // Clean up expired job tracking
  await redis.eval(`
    local keys = redis.call('KEYS', 'job:*')
    local expired = 0
    for i=1,#keys do
      if redis.call('TTL', keys[i]) <= 0 then
        redis.call('DEL', keys[i])
        expired = expired + 1
      end
    end
    return expired
  `)

  // Monitor memory usage
  if (memoryInfo.used_memory_percentage > 80) {
    await triggerMemoryCleanup()
  }
}
```

## Monitoring and Health Checks

### Queue Health Metrics

```pseudo
async function getQueueHealth() {
  return {
    timestamp: new Date().toISOString(),
    redis: await checkRedisHealth(),
    queues: await checkQueueDepths(),
    recent_jobs: await checkRecentJobCounts(),
    dead_letter_queue: await checkDeadLetterDepth()
  }
}

async function checkRedisHealth() {
  try {
    ping = await redis.ping()
    info = await redis.info()

    return {
      status: 'healthy',
      ping: ping,
      memory_used: parseMemoryUsage(info),
      connected_clients: parseClientCount(info)
    }
  } catch (error) {
    return {
      status: 'unhealthy',
      error: error.message
    }
  }
}
```

## Related Documentation

- **[Overview](overview)** - Queue management overview and naming conventions
- **[Data Structures](data-structures)** - Redis data structures and job payloads
- **[Migrator & Workers](migrator-workers)** - Job migration and worker processing
