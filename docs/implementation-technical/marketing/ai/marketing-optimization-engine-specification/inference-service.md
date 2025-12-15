---
title: Inference Service
description: Real-time model serving and prediction API endpoints
last_modified_date: 2025-01-15
level: 4
persona: ["ml-engineer", "developer"]
keywords: [inference, serving, API, prediction, real-time]
---

{% raw %}

# Inference Service

Real-time model serving infrastructure and prediction API endpoints.

## Serving Infrastructure

**Model Serving:** TensorFlow Serving for production deployment, ONNX Runtime for optimization

**Auto-scaling:** Kubernetes HPA for load-based scaling

## API Interfaces

```typescript
interface CampaignData {
  campaignId: string;
  adGroupId: string;
  keyword: string;
  device: string;
  location: string;
  timeOfDay: number;
  budget: number;
  currentBid: number;
  historicalMetrics: {
    ctr: number;
    conversionRate: number;
    spend: number;
    impressions: number;
    clicks: number;
  };
}

interface PredictionResponse {
  predictedCTR: number;
  predictedConversionRate: number;
  confidenceInterval: {
    lower: number;
    upper: number;
  };
  recommendedBid: number;
  expectedROAS: number;
  modelVersion: string;
  inferenceTimeMs: number;
  reasoning: string[];
}
```

## Prediction Endpoint

```typescript
async function predictPerformance(campaignData: CampaignData): Promise<PredictionResponse> {
  const startTime = Date.now();

  try {
    // Feature transformation
    const features = await featurePipeline.transform(campaignData);

    // Model prediction
    const prediction = await model.predict(features);

    // Confidence calculation
    const confidence = calculateConfidenceInterval(features);

    // Inference time
    const inferenceTime = Date.now() - startTime;

    // Generate reasoning
    const reasoning = generatePredictionReasoning(campaignData, prediction);

    return {
      predictedCTR: prediction[0],
      predictedConversionRate: prediction[1],
      confidenceInterval: {
        lower: prediction[0] - confidence,
        upper: prediction[0] + confidence
      },
      recommendedBid: calculateOptimalBid(campaignData, prediction),
      expectedROAS: calculateExpectedROAS(prediction, campaignData.budget),
      modelVersion: "v2.1.0",
      inferenceTimeMs: inferenceTime,
      reasoning
    };
  } catch (error) {
    throw new Error(`Prediction failed: ${error instanceof Error ? error.message : 'Unknown error'}`);
  }
}
```

## Helper Functions

```typescript
function calculateConfidenceInterval(features: number[]): number {
  const featureVariance = calculateFeatureVariance(features);
  const baseConfidence = 0.95;
  const variancePenalty = Math.min(featureVariance * 0.1, 0.2);
  return baseConfidence - variancePenalty;
}

function calculateOptimalBid(campaignData: CampaignData, prediction: number[]): number {
  const baseBid = campaignData.currentBid;
  const ctrMultiplier = prediction[0] / 0.02; // Normalize against 2% baseline
  const conversionMultiplier = prediction[1] / 0.05; // Normalize against 5% baseline

  return baseBid * Math.min(ctrMultiplier, 1.5) * Math.min(conversionMultiplier, 1.3);
}

function calculateExpectedROAS(prediction: number[], budget: number): number {
  const predictedRevenue = prediction[1] * budget * 50; // $50 average order value
  return predictedRevenue / budget;
}

function generatePredictionReasoning(
  campaignData: CampaignData, 
  prediction: number[]
): string[] {
  const reasoning = [];

  if (prediction[0] > 0.05) {
    reasoning.push("High predicted CTR indicates strong ad relevance");
  }

  if (prediction[1] > 0.08) {
    reasoning.push("Strong conversion rate suggests effective landing page");
  }

  if (campaignData.historicalMetrics.spend > 1000) {
    reasoning.push("Sufficient historical data enables confident prediction");
  }

  return reasoning;
}

function calculateFeatureVariance(features: number[]): number {
  const mean = features.reduce((sum, val) => sum + val, 0) / features.length;
  return features.reduce((sum, val) => sum + Math.pow(val - mean, 2), 0) / features.length;
}
```

## Related Documentation

- [Engine Overview](./)
- [Feature Engineering](./feature-engineering)
- [Continuous Learning](./continuous-learning)

{% endraw %}
