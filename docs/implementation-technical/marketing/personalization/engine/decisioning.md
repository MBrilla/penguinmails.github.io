---
title: "Real-Time Decisioning System"
description: "Contextual bandit implementation and personalization API"
last_modified_date: "2025-12-15"
level: "3"
persona: "Backend Engineers, ML Engineers"
keywords: "contextual bandit, ML inference, personalization API, exploration"
---

# Real-Time Decisioning System

The decisioning system selects personalized content for each customer request using a contextual bandit approach. This balances exploitation of known preferences with exploration of new content to improve recommendations over time.

## Contextual Bandit Implementation

The bandit scores available content items based on customer features, content attributes, and contextual signals. An exploration bonus occasionally promotes less-certain options to gather feedback data.

### Core Interfaces

```typescript
interface ContentItem {
  id: string;
  type: 'article' | 'product' | 'video' | 'email' | 'banner';
  title: string;
  category: string;
  tags: string[];
  metadata: Record<string, unknown>;
}

interface PersonalizationContext {
  availableItems: ContentItem[];
  customerContext?: Record<string, unknown>;
  timeContext?: {
    hour: number;
    dayOfWeek: number;
    season: string;
  };
}

interface PersonalizationDecision {
  selectedContent: ContentItem;
  confidenceScore: number;
  reasoning: string;
  alternativeOptions: ContentItem[];
  explorationUsed: boolean;
}
```

### Decision Engine

```typescript
class ContextualBanditDecisionEngine {
  private models: Record<string, MLModel>;
  private featureStore: FeatureStore;
  private explorationRate: number = 0.1;

  async selectPersonalizedContent(
    customerId: string,
    context: PersonalizationContext
  ): Promise<PersonalizationDecision> {
    const customerFeatures = await this.featureStore.getCustomerFeatures(customerId, [
      'engagement_score',
      'recency_score',
      'frequency_score',
      'monetary_score',
      'category_preferences'
    ]);

    const scores: Array<{ item: ContentItem; score: number }> = [];

    for (const contentItem of context.availableItems) {
      const contextFeatures = this.buildContextFeatures(
        customerFeatures,
        contentItem,
        context
      );

      const predictedReward = await this.models['content_recommendation'].predict(contextFeatures);
      const explorationBonus = this.shouldExplore() ? this.getExplorationBonus() : 0;
      const finalScore = predictedReward[0] + explorationBonus;

      scores.push({ item: contentItem, score: finalScore });
    }

    const sortedScores = scores.sort((a, b) => b.score - a.score);
    const selectedContent = sortedScores[0].item;
    const confidenceScore = sortedScores[0].score;
    const alternativeOptions = sortedScores.slice(1, 4).map(s => s.item);
    const reasoning = this.generateReasoning(selectedContent, customerFeatures, confidenceScore);

    return {
      selectedContent,
      confidenceScore,
      reasoning,
      alternativeOptions,
      explorationUsed: explorationBonus > 0
    };
  }

  private buildContextFeatures(
    customerFeatures: Record<string, number>,
    contentItem: ContentItem,
    context: PersonalizationContext
  ): number[] {
    const features: number[] = [];

    features.push(customerFeatures.engagement_score || 0);
    features.push(customerFeatures.recency_score || 0);
    features.push(customerFeatures.frequency_score || 0);
    features.push(customerFeatures.monetary_score || 0);
    features.push(this.encodeCategory(contentItem.category));
    features.push(contentItem.tags.length / 10);

    if (context.timeContext) {
      features.push(context.timeContext.hour / 24);
      features.push(context.timeContext.dayOfWeek / 7);
    } else {
      features.push(0, 0);
    }

    return features;
  }

  private shouldExplore(): boolean {
    return Math.random() < this.explorationRate;
  }

  private getExplorationBonus(): number {
    return Math.random() * 0.2;
  }

  private encodeCategory(category: string): number {
    const categories = ['article', 'product', 'video', 'email', 'banner'];
    return categories.indexOf(category) / categories.length;
  }

  private generateReasoning(
    selectedContent: ContentItem,
    customerFeatures: Record<string, number>,
    confidenceScore: number
  ): string {
    const reasons = [];

    if (customerFeatures.engagement_score > 0.7) {
      reasons.push('high customer engagement');
    }
    if (selectedContent.category === 'product' && customerFeatures.monetary_score > 0.6) {
      reasons.push('strong purchase intent');
    }
    if (confidenceScore > 0.8) {
      reasons.push('high prediction confidence');
    }
    if (reasons.length === 0) {
      reasons.push('best overall match based on customer profile');
    }

    return `Selected due to ${reasons.join(', ')}`;
  }
}
```

### Exploration Strategy

The 10% exploration rate means roughly 1 in 10 decisions adds a random bonus to content scores. This ensures the model continues learning about content performance rather than converging on local optima.

| Parameter | Value | Purpose |
|-----------|-------|---------|
| `explorationRate` | 0.1 | Fraction of requests with exploration |
| `maxExplorationBonus` | 0.2 | Maximum score boost during exploration |

## Personalization API

The Express-based API exposes personalization as a stateless HTTP service with health checks for load balancer integration.

```typescript
interface PersonalizationRequest {
  customerId: string;
  availableItems: ContentItem[];
  context?: {
    pageUrl?: string;
    referrer?: string;
    deviceType?: 'mobile' | 'desktop' | 'tablet';
    location?: string;
  };
}

interface PersonalizationResponse {
  selectedItem: ContentItem;
  confidenceScore: number;
  reasoning: string;
  latencyMs: number;
  alternativeOptions: ContentItem[];
}
```

### API Controller

```typescript
class PersonalizationAPIController {
  private decisionEngine: DecisionEngine;

  async personalizeContent(req: Request, res: Response): Promise<void> {
    const startTime = Date.now();

    try {
      const request: PersonalizationRequest = req.body;

      if (!request.customerId || !request.availableItems?.length) {
        res.status(400).json({
          error: 'Invalid request: customerId and availableItems are required'
        });
        return;
      }

      const context: PersonalizationContext = {
        availableItems: request.availableItems,
        customerContext: request.context,
        timeContext: {
          hour: new Date().getHours(),
          dayOfWeek: new Date().getDay(),
          season: this.getCurrentSeason()
        }
      };

      const decision = await this.decisionEngine.selectPersonalizedContent(
        request.customerId,
        context
      );

      const latencyMs = Date.now() - startTime;

      res.json({
        selectedItem: decision.selectedContent,
        confidenceScore: decision.confidenceScore,
        reasoning: decision.reasoning,
        latencyMs,
        alternativeOptions: decision.alternativeOptions
      });

      this.logDecision(request.customerId, decision, latencyMs);
    } catch (error) {
      res.status(500).json({
        error: 'Internal server error',
        message: error instanceof Error ? error.message : 'Unknown error'
      });
    }
  }

  private getCurrentSeason(): string {
    const month = new Date().getMonth();
    if (month >= 2 && month <= 4) return 'spring';
    if (month >= 5 && month <= 7) return 'summer';
    if (month >= 8 && month <= 10) return 'fall';
    return 'winter';
  }

  private logDecision(
    customerId: string,
    decision: PersonalizationDecision,
    latencyMs: number
  ): void {
    console.log('Personalization decision', {
      customerId,
      selectedItemId: decision.selectedContent.id,
      confidenceScore: decision.confidenceScore,
      latencyMs,
      timestamp: new Date().toISOString()
    });
  }
}
```

### Health Check Endpoint

The `/api/v1/health` endpoint validates that the decision engine and feature store are operational before accepting traffic.

```typescript
async healthCheck(req: Request, res: Response): Promise<void> {
  try {
    const testContext: PersonalizationContext = {
      availableItems: [{
        id: 'test',
        type: 'article',
        title: 'Test',
        category: 'general',
        tags: [],
        metadata: {}
      }]
    };

    await this.decisionEngine.selectPersonalizedContent('test-customer', testContext);

    res.json({
      status: 'healthy',
      timestamp: new Date().toISOString(),
      services: { decisionEngine: 'healthy', featureStore: 'healthy' }
    });
  } catch (error) {
    res.status(503).json({
      status: 'unhealthy',
      error: error instanceof Error ? error.message : 'Unknown error'
    });
  }
}
```

## Related Documentation

- [Engine Overview](README)
- [Feature Store](feature-store)
- [Content APIs](content-apis)
