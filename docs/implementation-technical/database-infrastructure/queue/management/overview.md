---
title: "Queue Management Overview"
description: "Overview of Redis-based queue management and job processing architecture"
last_modified_date: "2025-11-19"
level: 2
persona: "System Engineers"
keywords: ["queue-management", "redis", "job-processing", "priority-routing"]
---

# Queue Management Overview

Queue management handles the Redis-based high-performance job processing layer and the migrator process that bridges PostgreSQL durability with Redis speed. This component ensures efficient job routing, priority handling, and optimal worker consumption.

## Architecture Summary

The queue management system combines PostgreSQL for durable job storage with Redis for high-performance job processing. Jobs are created in PostgreSQL, migrated to Redis queues for worker consumption, and status is tracked across both systems.

## Redis Queue Structure

### Queue Naming Convention

Queues follow a consistent naming pattern for easy identification and priority routing:

```pseudo
queue naming = {
  email_processing: {
    high: "queue:email:processing:high",     // Priority ≤ 50
    normal: "queue:email:processing",        // Priority 51-150
    low: "queue:email:processing:low"        // Priority > 150
  },

  email_sending: {
    high: "queue:email-sending:high",
    normal: "queue:email-sending",
    low: "queue:email-sending:low"
  },

  analytics: {
    daily: "queue:analytics:daily-aggregate",
    campaign: "queue:analytics:campaign-aggregate",
    billing: "queue:analytics:billing-calculate"
  },

  processing: {
    warmup: "queue:warmup:process",
    bounce: "queue:bounce:process",
    webhook: "queue:webhook:process"
  }
}
```

### Priority Routing Logic

Jobs are automatically routed based on priority levels:

```pseudo
function determineQueueName(priority, baseQueueName) {
  if (priority <= HIGH_PRIORITY_THRESHOLD) {
    return `${baseQueueName}:high`    // Urgent jobs (≤ 50)
  } else if (priority <= NORMAL_PRIORITY_THRESHOLD) {
    return baseQueueName             // Standard jobs (51-150)
  } else {
    return `${baseQueueName}:low`    // Background jobs (> 150)
  }
}

HIGH_PRIORITY_THRESHOLD = 50
NORMAL_PRIORITY_THRESHOLD = 150
```

## Key Capabilities

The queue management system provides high performance through Redis-based queues with millisecond latency. Priority routing automatically routes jobs based on priority levels. Comprehensive error handling and retry mechanisms ensure reliability. Real-time queue health and performance tracking enable monitoring. Horizontal scaling through multiple workers and Redis clustering provides scalability.

## Related Documentation

- **[Data Structures](data-structures)** - Redis data structures and job payloads
- **[Migrator & Workers](migrator-workers)** - Job migration and worker processing
- **[Error Handling & Monitoring](error-handling)** - Retry logic, dead letter queue, performance
- **[Main Guide](/docs/implementation-technical/database-infrastructure/queue/main)** - Complete overview
- **[Architecture](/docs/implementation-technical/database-infrastructure/queue/architecture)** - System design principles
- **[Database Schema](/docs/implementation-technical/database-infrastructure/queue/database-schema)** - Job tables and indexes
