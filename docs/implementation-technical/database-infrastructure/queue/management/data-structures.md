---
title: "Redis Data Structures"
description: "Redis data structures for job queues, metadata tracking, and duplicate prevention"
last_modified_date: "2025-11-19"
level: 2
persona: "System Engineers"
keywords: ["redis", "data-structures", "job-payload", "metadata-tracking"]
---

# Redis Data Structures

Redis provides the high-performance layer for job processing, using optimized data structures for queue management, status tracking, and duplicate prevention.

## Job Queue Structure (Redis Lists)

Each Redis queue uses list data structures for FIFO job processing:

```pseudo
// Job payload structure in Redis lists
interface RedisJobPayload {
  id: string              // PostgreSQL job ID for tracking
  queue_name: string      // Original queue name
  priority: number        // Job priority level
  payload: object         // Job-specific data (minimal)
  created_at: string      // ISO timestamp
  attempt_count: number   // Current attempt number
  max_attempts: number    // Maximum retry attempts
}

// Example job structure
emailJob = {
  id: "uuid-job-123",
  queue_name: "email-sending",
  priority: 75,
  payload: {
    campaign_id: "uuid-campaign",
    recipient_id: "uuid-recipient",
    template_id: "welcome-email",
    // No large content or sensitive data
  },
  created_at: "2025-10-30T09:00:00Z",
  attempt_count: 0,
  max_attempts: 3
}
```

## Job Metadata Tracking (Redis Hashes)

Real-time job status tracking using Redis hashes:

```pseudo
// Redis hash for real-time status
interface JobMetadata {
  status: 'queued' | 'migrated' | 'processing' | 'completed' | 'failed' | 'retry_scheduled'
  worker_id: string       // Worker processing this job
  started_at: string      // Processing start time
  completed_at: string    // Processing completion time
  last_error: string      // Error message if failed
  attempt_count: number   // Current attempt number
  retry_delay: number     // Delay before next retry (ms)
}

// Hash key format: job:{jobId}
redis.hgetall("job:uuid-job-123") = {
  status: "processing",
  worker_id: "worker-1",
  started_at: "2025-10-30T09:01:00Z",
  attempt_count: "1"
}
```

## Recent Jobs Tracking (Redis Sets)

Prevents duplicate job processing:

```pseudo
// Set-based duplicate prevention
redis.sadd(`recent_jobs:${queueName}`, jobId)
redis.expire(`recent_jobs:${queueName}`, 3600) // 1 hour TTL

// Check for recent processing
if redis.sismember(`recent_jobs:${queueName}`, jobId) {
  // Skip duplicate job
  return DUPLICATE
}
```

## Queue Depth Monitoring

Track queue sizes for monitoring and scaling:

```pseudo
async function monitorQueueDepths() {
  queueNames = [
    'queue:email:processing:high',
    'queue:email:processing',
    'queue:email:processing:low',
    'queue:email-sending:high',
    'queue:email-sending',
    'queue:email-sending:low',
    'queue:analytics:daily-aggregate',
    'queue:warmup:process'
  ]

  depths = {}

  for (queueName in queueNames) {
    depths[queueName] = await redis.llen(queueName)
  }

  // Alert if any queue exceeds threshold
  for (queueName, depth in depths) {
    if (depth > MAX_QUEUE_DEPTH_THRESHOLD) {
      await alertSystem.queueDepthAlert(queueName, depth)
    }
  }

  return depths
}
```

## Related Documentation

- **[Overview](overview)** - Queue management overview and naming conventions
- **[Migrator & Workers](migrator-workers)** - Job migration and worker processing
- **[Error Handling & Monitoring](error-handling)** - Retry logic and performance optimization
