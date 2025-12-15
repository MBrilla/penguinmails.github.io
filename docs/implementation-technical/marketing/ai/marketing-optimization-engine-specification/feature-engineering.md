---
title: Feature Engineering
description: Input signals and data processing for marketing optimization
last_modified_date: 2025-01-15
level: 4
persona: ["ml-engineer", "data-scientist"]
keywords: [feature-engineering, data-processing, signals, normalization]
---

{% raw %}

# Feature Engineering

Input signals, feature extraction, and data processing pipeline for marketing optimization models.

## Input Signal Categories

### Real-Time Signals

**Campaign Metrics:** CTR, conversion rate, CPA, impression share, quality score, bid levels, budget utilization, competitor patterns

**User Behavior Signals:** Page views, session duration, bounce rate, form submissions, email engagement, cross-device behavior, purchase propensity

**External Market Signals:** Economic indicators, seasonal trends, competitor launches, platform algorithm changes

## Feature Engineering Pipeline

```typescript
interface RawCampaignData {
  timestamp: string;
  spend: number;
  clicks: number;
  impressions: number;
  conversions: number;
  campaignId: string;
  adGroupId: string;
  keyword: string;
  device: string;
  location: string;
}

interface EngineeredFeatures {
  hourOfDay: number;
  dayOfWeek: number;
  cpc: number;
  ctr: number;
  conversionRate: number;
  costPerConversion: number;
  impressionShare: number;
  qualityScore: number;
  dayOfMonth: number;
  isWeekend: boolean;
  timeCategory: 'morning' | 'afternoon' | 'evening' | 'night';
}

class FeatureEngineeringPipeline {
  async createFeatures(rawData: RawCampaignData[]): Promise<EngineeredFeatures[]> {
    return rawData.map(data => this.extractFeatures(data));
  }

  private extractFeatures(rawData: RawCampaignData): EngineeredFeatures {
    const timestamp = new Date(rawData.timestamp);
    const hour = timestamp.getHours();
    const dayOfWeek = timestamp.getDay();
    const dayOfMonth = timestamp.getDate();

    // Calculate derived metrics
    const cpc = rawData.clicks > 0 ? rawData.spend / rawData.clicks : 0;
    const ctr = rawData.impressions > 0 ? rawData.clicks / rawData.impressions : 0;
    const conversionRate = rawData.clicks > 0 ? rawData.conversions / rawData.clicks : 0;
    const costPerConversion = rawData.conversions > 0 
      ? rawData.spend / rawData.conversions 
      : 0;

    // Time-based features
    const timeCategory = this.categorizeTime(hour);
    const isWeekend = dayOfWeek === 0 || dayOfWeek === 6;

    // Quality metrics (would come from ad platform in production)
    const impressionShare = Math.random() * 100;
    const qualityScore = 7 + Math.random() * 3;

    return {
      hourOfDay: hour,
      dayOfWeek: dayOfWeek,
      cpc,
      ctr,
      conversionRate,
      costPerConversion,
      impressionShare,
      qualityScore,
      dayOfMonth,
      isWeekend,
      timeCategory
    };
  }

  private categorizeTime(hour: number): 'morning' | 'afternoon' | 'evening' | 'night' {
    if (hour >= 6 && hour < 12) return 'morning';
    if (hour >= 12 && hour < 18) return 'afternoon';
    if (hour >= 18 && hour < 22) return 'evening';
    return 'night';
  }
}
```

## Feature Normalization

```typescript
async normalizeFeatures(features: EngineeredFeatures[]): Promise<EngineeredFeatures[]> {
  const normalized = [...features];

  // Normalize hour of day (0-23 -> 0-1)
  const maxHour = 23;
  normalized.forEach(f => f.hourOfDay = f.hourOfDay / maxHour);

  // Normalize day of week (0-6 -> 0-1)
  const maxDayOfWeek = 6;
  normalized.forEach(f => f.dayOfWeek = f.dayOfWeek / maxDayOfWeek);

  return normalized;
}
```

## Related Documentation

- [Engine Overview](./)
- [Model Architecture](./model-architecture)
- [Inference Service](./inference-service)

{% endraw %}
