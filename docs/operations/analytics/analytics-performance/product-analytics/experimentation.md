---
title: "A/B Testing and Experimentation"
description: "Experiment design framework, categories, and statistical analysis"
last_modified_date: "2025-01-15"
level: "2"
persona: "Product Managers, Analytics Teams"
---

# A/B Testing and Experimentation

Comprehensive framework for designing, running, and analyzing product experiments.

## Experiment Design Framework

```typescript
interface ABExperiment {
  id: string;
  name: string;
  hypothesis: string;
  feature: string;
  variants: {
    control: ExperimentVariant;
    treatment: ExperimentVariant;
  };
  metrics: {
    primary: string;        // Main success metric
    secondary: string[];    // Supporting metrics
    guardrail: string[];    // Metrics that must not degrade
  };
  sampleSize: number;
  duration: number;
  status: 'draft' | 'running' | 'completed' | 'failed';
  results?: ExperimentResults;
}

interface ExperimentResults {
  winner: 'control' | 'treatment' | 'tie';
  confidence: number;
  impact: number;
  statisticalSignificance: boolean;
  practicalSignificance: boolean;
  recommendations: string[];
}
```

## Experiment Categories

| Category | Description | Examples |
|----------|-------------|----------|
| Feature Optimization | Improving existing feature performance | Button placement, form design |
| User Experience | Testing UI/UX changes and workflows | Navigation flow, visual design |
| Onboarding Flow | Optimizing user activation and setup | Tutorial steps, welcome screens |
| Pricing Optimization | Testing pricing structures and messaging | Plan display, pricing page layout |
| Content Effectiveness | Testing help content and tutorials | Help articles, tooltip text |

## Statistical Analysis

### Significance Calculation

```typescript
const calculateStatisticalSignificance = (
  control: number[],
  treatment: number[],
  alpha: number = 0.05
) => {
  const controlMean = control.reduce((a, b) => a + b) / control.length;
  const treatmentMean = treatment.reduce((a, b) => a + b) / treatment.length;

  const pooledVariance = (
    control.reduce((sum, val) => sum + Math.pow(val - controlMean, 2), 0) +
    treatment.reduce((sum, val) => sum + Math.pow(val - treatmentMean, 2), 0)
  ) / (control.length + treatment.length - 2);

  const standardError = Math.sqrt(
    pooledVariance * (1/control.length + 1/treatment.length)
  );
  const tStatistic = (treatmentMean - controlMean) / standardError;

  const pValue = 2 * (1 - cumulativeDistribution(Math.abs(tStatistic)));

  return {
    significant: pValue < alpha,
    pValue,
    effectSize: treatmentMean - controlMean,
    confidenceInterval: [
      treatmentMean - controlMean - 1.96 * standardError,
      treatmentMean - controlMean + 1.96 * standardError
    ]
  };
};
```

## Experiment Best Practices

### Before Running

1. Define clear hypothesis
2. Set primary and guardrail metrics
3. Calculate required sample size
4. Document expected duration

### During Experiment

1. Monitor for technical issues
2. Check for sample ratio mismatch
3. Track guardrail metrics
4. Avoid peeking at results

### After Experiment

1. Verify statistical significance
2. Check practical significance
3. Review guardrail metrics
4. Document learnings

## Experiment Reporting

### Key Metrics to Report

| Metric | Description |
|--------|-------------|
| Sample Size | Number of users in each variant |
| Conversion Rate | Primary metric performance |
| Confidence Level | Statistical confidence (typically 95%) |
| Effect Size | Magnitude of difference |
| P-Value | Statistical significance measure |

---

**Related Documents:**
- [Overview](overview.md)
- [Market Fit](market-fit.md)
- [Optimization & Reporting](optimization-reporting.md)
