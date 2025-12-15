---
title: "Content and Offer APIs"
description: "Content management integration and performance monitoring"
last_modified_date: "2025-12-15"
level: "3"
persona: "Backend Engineers"
keywords: "content API, performance monitoring, Prometheus, conversion tracking"
---

# Content and Offer APIs

The content API client fetches personalized content from external content management systems. Performance monitoring tracks decision latency, conversion rates, and model effectiveness.

## Content Management Integration

The content API client abstracts external CMS interactions with timeout handling and error recovery.

```typescript
interface ContentAPIResponse {
  content_items: ContentItem[];
  pagination: {
    page: number;
    per_page: number;
    total: number;
    has_more: boolean;
  };
  metadata: {
    generated_at: string;
    processing_time_ms: number;
  };
}

interface ContentAPIClient {
  getPersonalizedContent(
    customerSegment: string,
    contentType: string,
    count?: number
  ): Promise<ContentItem[]>;
  getContentByIds(ids: string[]): Promise<ContentItem[]>;
  getContentByCategory(category: string, limit?: number): Promise<ContentItem[]>;
}
```

### API Client Implementation

```typescript
class ContentAPIClientImpl implements ContentAPIClient {
  private baseUrl: string;
  private headers: Record<string, string>;
  private defaultTimeout: number = 5000;

  constructor(baseUrl: string, apiKey: string) {
    this.baseUrl = baseUrl.replace(/\/$/, '');
    this.headers = {
      'Authorization': `Bearer ${apiKey}`,
      'Content-Type': 'application/json',
      'Accept': 'application/json'
    };
  }

  async getPersonalizedContent(
    customerSegment: string,
    contentType: string,
    count: number = 10
  ): Promise<ContentItem[]> {
    const url = `${this.baseUrl}/api/v2/content/personalized`;
    const params = new URLSearchParams({
      segment: customerSegment,
      type: contentType,
      limit: count.toString(),
      include_metadata: 'true'
    });

    const response = await fetch(`${url}?${params.toString()}`, {
      method: 'GET',
      headers: this.headers,
      signal: AbortSignal.timeout(this.defaultTimeout)
    });

    if (!response.ok) {
      throw new APIException(`Content API error: ${response.status}`);
    }

    const data: ContentAPIResponse = await response.json();
    return data.content_items;
  }

  async getContentByIds(ids: string[]): Promise<ContentItem[]> {
    const url = `${this.baseUrl}/api/v2/content/batch`;

    const response = await fetch(url, {
      method: 'POST',
      headers: this.headers,
      body: JSON.stringify({ ids }),
      signal: AbortSignal.timeout(this.defaultTimeout)
    });

    if (!response.ok) {
      throw new APIException(`Content API error: ${response.status}`);
    }

    const data: ContentAPIResponse = await response.json();
    return data.content_items;
  }

  async getContentByCategory(category: string, limit: number = 20): Promise<ContentItem[]> {
    const url = `${this.baseUrl}/api/v2/content/category/${encodeURIComponent(category)}`;
    const params = new URLSearchParams({ limit: limit.toString() });

    const response = await fetch(`${url}?${params.toString()}`, {
      method: 'GET',
      headers: this.headers,
      signal: AbortSignal.timeout(this.defaultTimeout)
    });

    if (!response.ok) {
      throw new APIException(`Content API error: ${response.status}`);
    }

    const data: ContentAPIResponse = await response.json();
    return data.content_items;
  }
}

class APIException extends Error {
  constructor(message: string) {
    super(message);
    this.name = 'APIException';
  }
}
```

### Timeout and Error Handling

The 5-second default timeout prevents slow CMS responses from blocking personalization. When the content API fails, the decisioning system falls back to cached content or default recommendations.

## Performance Monitoring

Prometheus metrics track decision latency, throughput, and conversion rates for operational visibility and model improvement.

```typescript
interface DecisionMetrics {
  decisionId: string;
  customerId: string;
  latencyMs: number;
  selectedItemId: string;
  confidenceScore: number;
  timestamp: string;
}

interface ConversionMetrics {
  customerId: string;
  contentId: string;
  conversionType: string;
  value?: number;
  decisionId?: string;
  timestamp: string;
}
```

### Performance Monitor

```typescript
class PersonalizationPerformanceMonitor {
  private prometheusClient: PrometheusClient;
  private metricsBuffer: DecisionMetrics[] = [];

  constructor(prometheusClient: PrometheusClient) {
    this.prometheusClient = prometheusClient;
  }

  trackDecisionMetrics(decisionId: string, customerId: string, latencyMs: number): void {
    this.prometheusClient.histogram('personalization_decision_latency_ms', latencyMs, {
      customer_segment: this.getCustomerSegment(customerId)
    });

    this.prometheusClient.counter('personalization_decisions_total', 1, {
      customer_segment: this.getCustomerSegment(customerId)
    });

    this.metricsBuffer.push({
      decisionId,
      customerId,
      latencyMs,
      selectedItemId: '',
      confidenceScore: 0,
      timestamp: new Date().toISOString()
    });

    if (this.metricsBuffer.length > 1000) {
      this.flushMetricsBuffer();
    }
  }

  trackConversionMetrics(
    customerId: string,
    contentId: string,
    conversionType: string,
    value?: number
  ): void {
    this.prometheusClient.counter('personalization_conversions_total', 1, {
      conversion_type: conversionType,
      customer_segment: this.getCustomerSegment(customerId)
    });

    if (value !== undefined) {
      this.prometheusClient.histogram('personalization_conversion_value', value, {
        conversion_type: conversionType
      });
    }
  }

  private getCustomerSegment(customerId: string): string {
    return 'default';
  }

  private flushMetricsBuffer(): void {
    const batch = [...this.metricsBuffer];
    this.metricsBuffer = [];
    console.log(`Processed ${batch.length} decision metrics`);
  }

  async generatePerformanceReport(timeRange: { start: Date; end: Date }): Promise<{
    totalDecisions: number;
    averageLatencyMs: number;
    p95LatencyMs: number;
    conversionRate: number;
    topPerformingContent: Array<{ contentId: string; conversions: number }>;
  }> {
    return {
      totalDecisions: 10000,
      averageLatencyMs: 45,
      p95LatencyMs: 120,
      conversionRate: 0.12,
      topPerformingContent: [
        { contentId: 'article-123', conversions: 150 },
        { contentId: 'product-456', conversions: 120 },
        { contentId: 'video-789', conversions: 90 }
      ]
    };
  }
}
```

### Key Metrics

| Metric | Type | Purpose |
|--------|------|---------|
| `personalization_decision_latency_ms` | Histogram | Track response time distribution |
| `personalization_decisions_total` | Counter | Count total personalization requests |
| `personalization_conversions_total` | Counter | Track successful conversions |
| `personalization_conversion_value` | Histogram | Track revenue from conversions |

### Conversion Tracking

Conversion events link back to personalization decisions through the `decisionId` field. This enables attribution analysis to identify which content recommendations drive the highest conversion rates.

## Dependencies and Infrastructure

**Required Services**:
- PostgreSQL + Redis for event streaming
- Redis for feature serving
- TensorFlow Serving for ML inference
- Elasticsearch for customer search (planned 2026)
- Prometheus/Grafana for monitoring (planned 2026)

**Performance Targets**:
- Decision latency: <50ms
- Feature retrieval: <10ms
- Throughput: 10,000+ decisions/second

## Related Documentation

- [Engine Overview](README)
- [Decisioning System](decisioning)
- [Marketing Analytics Architecture](/docs/implementation-technical/marketing/analytics-integration/marketing-analytics-architecture)
