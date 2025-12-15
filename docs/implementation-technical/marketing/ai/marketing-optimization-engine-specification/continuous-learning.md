---
title: Continuous Learning
description: Online learning and A/B testing framework for model improvement
last_modified_date: 2025-01-15
level: 4
persona: ["ml-engineer", "data-scientist"]
keywords: [online-learning, A/B-testing, drift-detection, feedback-loops]
---

{% raw %}

# Continuous Learning

Online learning implementation and A/B testing framework for continuous model improvement.

## Online Learning System

Real-time model updates using Apache Flink, online gradient descent, Thompson Sampling, and drift detection.

```typescript
interface FeedbackData {
  features: number[];
  prediction: number;
  actualOutcome: number;
  timestamp: string;
  campaignId: string;
}

interface DriftDetectionResult {
  hasDrift: boolean;
  driftScore: number;
  confidence: number;
}

class OnlineLearningSystem {
  private updateBuffer: FeedbackData[] = [];
  private driftDetector: DriftDetector;
  private model: CampaignPerformanceModel;
  private learningRate: number = 0.001;
  private maxBufferSize: number = 10000;

  constructor(model: CampaignPerformanceModel) {
    this.model = model;
    this.driftDetector = new DriftDetector();
  }

  async processFeedback(feedback: FeedbackData): Promise<void> {
    this.updateBuffer.push(feedback);

    if (this.updateBuffer.length > this.maxBufferSize) {
      this.updateBuffer.shift();
    }

    const driftResult = await this.driftDetector.update(feedback.actualOutcome);
    if (driftResult.hasDrift) {
      await this.triggerModelRetrain();
    }

    const loss = this.computeLoss(feedback.prediction, feedback.actualOutcome);
    const gradients = await this.computeGradients(feedback.features, loss);
    await this.applyGradients(gradients);
  }

  private async computeLoss(prediction: number, actualOutcome: number): Promise<number> {
    return Math.pow(prediction - actualOutcome, 2);
  }

  private async computeGradients(features: number[], loss: number): Promise<number[]> {
    return features.map(feature => feature * loss * this.learningRate);
  }

  private async applyGradients(gradients: number[]): Promise<void> {
    await this.model.applyGradients(gradients, this.learningRate);
  }

  private async triggerModelRetrain(): Promise<void> {
    console.log("Triggering model retraining due to drift detection");
  }
}
```

## Drift Detection

```typescript
class DriftDetector {
  private baselineDistribution: number[] = [];
  private currentDistribution: number[] = [];
  private driftThreshold: number = 0.05;

  async update(actualOutcome: number): Promise<DriftDetectionResult> {
    this.currentDistribution.push(actualOutcome);

    if (this.currentDistribution.length > 1000) {
      this.currentDistribution.shift();
    }

    const driftScore = await this.computeDriftScore();
    const hasDrift = driftScore > this.driftThreshold;

    return {
      hasDrift,
      driftScore,
      confidence: this.calculateConfidence()
    };
  }

  private async computeDriftScore(): Promise<number> {
    if (this.currentDistribution.length < 10) return 0;

    const baselineMean = this.calculateMean(this.baselineDistribution);
    const currentMean = this.calculateMean(this.currentDistribution);

    return Math.abs(baselineMean - currentMean) / baselineMean;
  }

  private calculateConfidence(): number {
    return Math.min(this.currentDistribution.length / 100, 1);
  }

  private calculateMean(data: number[]): number {
    return data.reduce((sum, val) => sum + val, 0) / data.length;
  }
}
```

## A/B Testing Framework

```typescript
interface ExperimentVariant {
  name: string;
  trafficPercentage: number;
  description: string;
  isActive: boolean;
}

interface ActiveTest {
  experimentId: string;
  variants: Record<string, ExperimentVariant>;
  startDate: string;
  endDate?: string;
  status: 'active' | 'paused' | 'completed';
}

class ABTestManager {
  private activeTests: Record<string, ActiveTest> = {};
  private assignmentCache: Map<string, string> = new Map();

  async getVariantAssignment(userId: string, experimentId: string): Promise<string> {
    const cacheKey = `${userId}_${experimentId}`;
    const cached = this.assignmentCache.get(cacheKey);
    if (cached) return cached;

    const hashValue = this.hashString(`${userId}_${experimentId}`) % 100;
    const assignment = await this.assignVariant(hashValue, experimentId);

    this.assignmentCache.set(cacheKey, assignment);
    return assignment;
  }

  private async assignVariant(hashValue: number, experimentId: string): Promise<string> {
    const test = this.activeTests[experimentId];
    if (!test) return 'control';

    let cumulativeSplit = 0;
    for (const [variantName, variant] of Object.entries(test.variants)) {
      if (!variant.isActive) continue;
      cumulativeSplit += variant.trafficPercentage;
      if (hashValue < cumulativeSplit) return variantName;
    }

    return 'control';
  }

  private hashString(str: string): number {
    let hash = 0;
    for (let i = 0; i < str.length; i++) {
      const char = str.charCodeAt(i);
      hash = ((hash << 5) - hash) + char;
      hash = hash & hash;
    }
    return Math.abs(hash);
  }

  async registerExperiment(experiment: ActiveTest): Promise<void> {
    this.activeTests[experiment.experimentId] = experiment;
  }

  async pauseExperiment(experimentId: string): Promise<void> {
    if (this.activeTests[experimentId]) {
      this.activeTests[experimentId].status = 'paused';
    }
  }

  async completeExperiment(experimentId: string): Promise<void> {
    if (this.activeTests[experimentId]) {
      this.activeTests[experimentId].status = 'completed';
      this.activeTests[experimentId].endDate = new Date().toISOString();
    }
  }
}
```

## Related Documentation

- [Engine Overview](./)
- [Inference Service](./inference-service)
- [Model Monitoring](./model-monitoring)

{% endraw %}
