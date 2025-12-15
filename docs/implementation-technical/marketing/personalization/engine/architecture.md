---
title: "Personalization Architecture"
description: "Core pipeline design and segmentation engine implementation"
last_modified_date: "2025-12-15"
level: "3"
persona: "Backend Engineers"
keywords: "CDP, segmentation, Elasticsearch, Redis, pipeline"
---

# Personalization Architecture

The personalization pipeline processes customer data through three interconnected systems: the Customer Data Platform for data ingestion, the Feature Store for real-time serving, and the Decision Engine for content selection.

## Core Pipeline Components

### Customer Data Platform (CDP)

PostgreSQL and Redis handle real-time event streaming while PostgreSQL manages batch processing for historical analysis. Customer profiles are indexed in Elasticsearch (planned for 2026) with field-level encryption for data privacy controls.

### Real-Time Feature Store

Redis serves as the low-latency feature layer with sub-10ms retrieval times. PostgreSQL provides durable storage while a feature management layer handles versioning with automated rollbacks when model performance degrades.

### Decision Engine

Drools rule engine processes business logic for segment-based routing. TensorFlow Serving handles ML inference for personalized recommendations. A Redis-based contextual bandit manages exploration-exploitation with an integrated A/B testing framework for traffic allocation.

## Segmentation Engine

The segmentation engine builds dynamic customer segments from Elasticsearch queries and maintains membership in Redis sets for fast lookup during personalization requests.

### Interface Definitions

```typescript
interface SegmentDefinition {
  name: string;
  conditions: SegmentCondition[];
  description?: string;
  createdAt: string;
}

interface SegmentCondition {
  field: string;
  operator: 'equals' | 'greater_than' | 'less_than' | 'contains' | 'in';
  value: string | number | string[];
}

interface ElasticsearchQuery {
  bool: {
    must: Array<{ [key: string]: unknown }>;
  };
}
```

### Segmentation Engine Implementation

```typescript
class SegmentationEngine {
  private esClient: ElasticsearchClient;
  private redisClient: RedisClient;

  constructor(esClient: ElasticsearchClient, redisClient: RedisClient) {
    this.esClient = esClient;
    this.redisClient = redisClient;
  }

  async createDynamicSegment(segmentDefinition: SegmentDefinition): Promise<number> {
    const esQuery = this.buildSegmentQuery(segmentDefinition.conditions);

    const segmentCustomers = await this.esClient.search({
      index: 'customer_profiles',
      body: {
        query: esQuery,
        _source: ['customer_id']
      }
    });

    const customerIds = segmentCustomers.hits.hits.map(hit => hit._source.customer_id);

    for (const customerId of customerIds) {
      await this.redisClient.sadd(`segment:${segmentDefinition.name}`, customerId);
    }

    return customerIds.length;
  }

  private buildSegmentQuery(conditions: SegmentCondition[]): ElasticsearchQuery {
    const mustQueries = conditions.map(condition => {
      switch (condition.operator) {
        case 'equals':
          return { term: { [condition.field]: condition.value } };
        case 'greater_than':
          return { range: { [condition.field]: { gt: condition.value } } };
        case 'less_than':
          return { range: { [condition.field]: { lt: condition.value } } };
        case 'contains':
          return { match: { [condition.field]: condition.value } };
        case 'in':
          return { terms: { [condition.field]: condition.value as string[] } };
        default:
          return { match_all: {} };
      }
    });

    return { bool: { must: mustQueries } };
  }

  async removeCustomerFromSegment(segmentName: string, customerId: string): Promise<void> {
    await this.redisClient.srem(`segment:${segmentName}`, customerId);
  }

  async getSegmentMembers(segmentName: string): Promise<string[]> {
    return await this.redisClient.smembers(`segment:${segmentName}`);
  }
}
```

### Segment Query Operators

The query builder translates segment conditions into Elasticsearch DSL:

| Operator | Elasticsearch Query | Use Case |
|----------|---------------------|----------|
| `equals` | `term` query | Exact match on keyword fields |
| `greater_than` | `range` with `gt` | Numeric comparisons |
| `less_than` | `range` with `lt` | Numeric comparisons |
| `contains` | `match` query | Full-text search |
| `in` | `terms` query | Multiple value matching |

### Redis Segment Storage

Segment membership uses Redis sets for O(1) membership checks during personalization:

```
segment:high_value_customers -> {customer_id_1, customer_id_2, ...}
segment:active_last_30d -> {customer_id_3, customer_id_4, ...}
```

This structure supports set operations like intersection and union for complex segment queries.

## Related Documentation

- [Engine Overview](README)
- [Feature Store](feature-store)
- [Decisioning System](decisioning)
