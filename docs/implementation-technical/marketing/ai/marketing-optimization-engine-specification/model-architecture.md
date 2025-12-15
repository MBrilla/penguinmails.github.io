---
title: Model Architecture
description: Neural network and bid optimization model implementations
last_modified_date: 2025-01-15
level: 4
persona: ["ml-engineer", "data-scientist"]
keywords: [neural-network, bid-optimization, deep-learning, XGBoost]
---

{% raw %}

# Model Architecture

Campaign performance prediction and bid optimization model implementations.

## Campaign Performance Prediction Model

Multi-layer neural network for predicting CTR and conversion rates.

```typescript
interface ModelConfig {
  inputDim: number;
  hiddenDims: number[];
  dropoutRate: number;
}

interface PredictionResult {
  predictedCTR: number;
  predictedConversionRate: number;
  confidence: number;
}

class CampaignPerformanceModel {
  private layers: NeuralLayer[];
  private outputLayer: DenseLayer;
  private config: ModelConfig;

  constructor(config: ModelConfig = { 
    inputDim: 128, 
    hiddenDims: [256, 128, 64], 
    dropoutRate: 0.2 
  }) {
    this.config = config;
    this.layers = [];
    this.initializeLayers();
  }

  private initializeLayers(): void {
    let inputDim = this.config.inputDim;

    // Input layer to first hidden layer
    this.layers.push(new DenseLayer(inputDim, this.config.hiddenDims[0], 'relu'));
    this.layers.push(new DropoutLayer(this.config.dropoutRate));

    // Hidden layers
    for (let i = 0; i < this.config.hiddenDims.length - 1; i++) {
      this.layers.push(
        new DenseLayer(this.config.hiddenDims[i], this.config.hiddenDims[i + 1], 'relu')
      );
      this.layers.push(new DropoutLayer(this.config.dropoutRate));
    }

    // Output layer with sigmoid activation
    this.outputLayer = new DenseLayer(
      this.config.hiddenDims[this.config.hiddenDims.length - 1], 
      1, 
      'sigmoid'
    );
  }

  async predict(input: number[]): Promise<PredictionResult> {
    let output = [...input];

    for (const layer of this.layers) {
      output = await layer.forward(output);
    }

    const finalPrediction = await this.outputLayer.forward(output);

    return {
      predictedCTR: finalPrediction[0],
      predictedConversionRate: finalPrediction[0],
      confidence: this.calculateConfidence(output)
    };
  }

  private calculateConfidence(features: number[]): number {
    const variance = this.calculateFeatureVariance(features);
    return Math.max(0, 1 - variance);
  }

  private calculateFeatureVariance(features: number[]): number {
    const mean = features.reduce((sum, val) => sum + val, 0) / features.length;
    return features.reduce((sum, val) => sum + Math.pow(val - mean, 2), 0) / features.length;
  }
}
```

## Bid Optimization Model

XGBoost-based model for optimal bid prediction.

```typescript
interface BidFeatures {
  historicalCTR: number;
  timeOfDay: number;
  dayOfWeek: number;
  competitorBid: number;
  budgetRemaining: number;
  campaignAge: number;
}

interface BidPrediction {
  optimalBid: number;
  confidence: number;
  reasoning: string[];
}

class BidOptimizationModel {
  private model: XGBoostModel;

  constructor() {
    this.model = new XGBoostModel({
      nEstimators: 100,
      maxDepth: 6,
      learningRate: 0.1,
      subsample: 0.8,
      randomState: 42
    });
  }

  async predictOptimalBid(features: BidFeatures): Promise<BidPrediction> {
    const featureArray = this.extractFeatures(features);
    const prediction = await this.model.predict(featureArray);

    return {
      optimalBid: prediction[0],
      confidence: this.calculateConfidence(features),
      reasoning: this.generateReasoning(features, prediction[0])
    };
  }

  private extractFeatures(features: BidFeatures): number[] {
    return [
      features.historicalCTR,
      features.timeOfDay,
      features.dayOfWeek,
      features.competitorBid,
      features.budgetRemaining,
      features.campaignAge
    ];
  }

  private calculateConfidence(features: BidFeatures): number {
    const dataReliability = Math.min(features.campaignAge / 30, 1);
    const budgetReliability = features.budgetRemaining > 0.1 ? 1 : 0.5;
    return (dataReliability + budgetReliability) / 2;
  }

  private generateReasoning(features: BidFeatures, bid: number): string[] {
    const reasoning = [];

    if (features.historicalCTR > 0.05) {
      reasoning.push("High historical CTR supports higher bid");
    }

    if (features.campaignAge > 14) {
      reasoning.push("Mature campaign data enables precise bidding");
    }

    if (features.budgetRemaining < 0.1) {
      reasoning.push("Low budget remaining may limit bid potential");
    }

    return reasoning;
  }
}
```

## Related Documentation

- [Engine Overview](./)
- [Feature Engineering](./feature-engineering)
- [Inference Service](./inference-service)

{% endraw %}
