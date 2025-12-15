---
title: "Feature Store Implementation"
description: "Customer feature storage and behavioral feature computation"
last_modified_date: "2025-12-15"
level: "3"
persona: "Backend Engineers, ML Engineers"
keywords: "feature store, Redis, behavioral features, engagement score"
---

# Feature Store Implementation

The feature store provides low-latency access to customer features for real-time personalization. Redis serves as the primary storage layer with sub-10ms retrieval times, while PostgreSQL maintains durable copies for recovery and batch processing.

## Customer Feature Schema

Features are organized by type and version to support model evolution without breaking existing consumers.

```typescript
interface CustomerFeature {
  customerId: string;
  features: Record<string, number>;
  timestamp: number;
  featureType: 'customer' | 'behavioral' | 'contextual' | 'derived';
  version: string;
  metadata?: Record<string, unknown>;
}

interface FeatureStore {
  getCustomerFeatures(customerId: string, featureNames: string[]): Promise<Record<string, number>>;
  setCustomerFeatures(customerId: string, features: Record<string, number>): Promise<void>;
  getFeatureHistory(customerId: string, featureName: string, limit?: number): Promise<Array<CustomerFeature>>;
}
```

## Redis Feature Storage

The implementation uses Redis hashes for efficient multi-field retrieval. Each customer's features are stored under a single key with individual fields for each feature value.

```typescript
class FeatureStoreImpl implements FeatureStore {
  private redisClient: RedisHashClient;
  private defaultTTL: number = 3600; // 1 hour

  constructor(redisClient: RedisHashClient) {
    this.redisClient = redisClient;
  }

  async getCustomerFeatures(
    customerId: string,
    featureNames: string[]
  ): Promise<Record<string, number>> {
    const redisKey = `customer_features:${customerId}`;
    const cachedFeatures = await this.redisClient.hmget(redisKey, ...featureNames);

    const features: Record<string, number> = {};
    for (let i = 0; i < featureNames.length; i++) {
      const value = cachedFeatures[i];
      features[featureNames[i]] = value !== null ? parseFloat(value) : 0.0;
    }

    return features;
  }

  async setCustomerFeatures(
    customerId: string,
    features: Record<string, number>
  ): Promise<void> {
    const redisKey = `customer_features:${customerId}`;

    for (const [featureName, value] of Object.entries(features)) {
      await this.redisClient.hset(redisKey, featureName, value.toString());
    }

    await this.redisClient.expire(redisKey, this.defaultTTL);
  }

  async getFeatureVector(customerId: string): Promise<CustomerFeature> {
    const redisKey = `customer_features:${customerId}`;
    const features = await this.redisClient.hgetall(redisKey);

    const numericFeatures: Record<string, number> = {};
    for (const [key, value] of Object.entries(features)) {
      numericFeatures[key] = parseFloat(value) || 0.0;
    }

    return {
      customerId,
      features: numericFeatures,
      timestamp: Date.now(),
      featureType: 'customer',
      version: '1.0'
    };
  }
}
```

### Redis Key Structure

```
customer_features:{customer_id} -> {
  engagement_score: "0.75",
  recency_score: "0.82",
  frequency_score: "0.45",
  monetary_score: "0.91",
  category_preferences: "0.67"
}
```

The TTL ensures stale features expire automatically. Batch jobs refresh features before expiration for active customers.

## Behavioral Feature Computation

Behavioral features aggregate customer events from the last 30 days into normalized scores. The computation runs on Spark for batch processing or in-process for real-time updates.

```typescript
interface CustomerEvent {
  customerId: string;
  eventType: 'page_view' | 'email_open' | 'email_click' | 'form_submit' | 'purchase' | 'login';
  timestamp: string;
  metadata?: Record<string, unknown>;
  sessionId?: string;
}

interface EngagementFeatures {
  engagementScore: number;
  pageViews30d: number;
  emailOpens30d: number;
  formSubmits30d: number;
  conversionRate: number;
  loginFrequency: number;
  sessionDuration: number;
}
```

### Feature Computation Engine

```typescript
class BehavioralFeatureComputer {
  private featureWeights = {
    pageView: 0.3,
    emailOpen: 0.2,
    formSubmit: 0.2,
    purchase: 0.3,
    login: 0.1
  };

  async computeEngagementFeatures(
    customerEvents: CustomerEvent[]
  ): Promise<EngagementFeatures> {
    const thirtyDaysAgo = new Date();
    thirtyDaysAgo.setDate(thirtyDaysAgo.getDate() - 30);

    const recentEvents = customerEvents.filter(
      event => new Date(event.timestamp) >= thirtyDaysAgo
    );

    const aggregated = this.aggregateEvents(recentEvents);
    const engagementScore = this.calculateEngagementScore(aggregated);
    const conversionRate = this.calculateConversionRate(aggregated);

    return {
      engagementScore: Math.min(engagementScore, 1.0),
      pageViews30d: aggregated.pageViews,
      emailOpens30d: aggregated.emailOpens,
      formSubmits30d: aggregated.formSubmits,
      conversionRate,
      loginFrequency: aggregated.logins,
      sessionDuration: aggregated.sessionDuration
    };
  }

  private aggregateEvents(events: CustomerEvent[]): {
    pageViews: number;
    emailOpens: number;
    formSubmits: number;
    purchases: number;
    logins: number;
    sessionDuration: number;
  } {
    const counts = { pageViews: 0, emailOpens: 0, formSubmits: 0, purchases: 0, logins: 0 };

    for (const event of events) {
      switch (event.eventType) {
        case 'page_view': counts.pageViews++; break;
        case 'email_open': counts.emailOpens++; break;
        case 'form_submit': counts.formSubmits++; break;
        case 'purchase': counts.purchases++; break;
        case 'login': counts.logins++; break;
      }
    }

    return { ...counts, sessionDuration: 0 };
  }

  private calculateEngagementScore(features: {
    pageViews: number;
    emailOpens: number;
    formSubmits: number;
    purchases: number;
    logins: number;
  }): number {
    const normalized = {
      pageViews: Math.min(features.pageViews / 100, 1),
      emailOpens: Math.min(features.emailOpens / 50, 1),
      formSubmits: Math.min(features.formSubmits / 20, 1),
      purchases: Math.min(features.purchases / 10, 1),
      logins: Math.min(features.logins / 30, 1)
    };

    return (
      normalized.pageViews * this.featureWeights.pageView +
      normalized.emailOpens * this.featureWeights.emailOpen +
      normalized.formSubmits * this.featureWeights.formSubmit +
      normalized.purchases * this.featureWeights.purchase +
      normalized.logins * this.featureWeights.login
    );
  }

  private calculateConversionRate(features: {
    formSubmits: number;
    pageViews: number;
  }): number {
    return features.pageViews === 0 ? 0 : features.formSubmits / features.pageViews;
  }
}
```

### Feature Weights

The engagement score uses weighted components to balance different behavioral signals:

| Event Type | Weight | Normalization Factor |
|------------|--------|---------------------|
| Page views | 0.3 | 100 views = 1.0 |
| Email opens | 0.2 | 50 opens = 1.0 |
| Form submits | 0.2 | 20 submits = 1.0 |
| Purchases | 0.3 | 10 purchases = 1.0 |
| Logins | 0.1 | 30 logins = 1.0 |

Weights can be adjusted based on business priorities. Higher purchase weight emphasizes revenue-generating behavior.

## Related Documentation

- [Engine Overview](README)
- [Architecture](architecture)
- [Decisioning System](decisioning)
