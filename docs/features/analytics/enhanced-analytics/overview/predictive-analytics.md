---
title: "Predictive Analytics"
description: ML-powered predictions for send time, subject lines, deliverability, and churn
last_modified_date: 2025-01-15
level: 3
persona: ["developer", "data-scientist"]
keywords: [predictive, ML, machine-learning, optimization, send-time]
---

# Predictive Analytics

Machine learning models for optimizing email campaigns through prediction and analysis.

## Send Time Optimization

Predicts optimal send times based on individual contact engagement patterns.

```typescript
interface SendTimeOptimization {
  contactId: string;
  recommendedSendTime: Date;
  confidence: number;  // 0-100
  expectedOpenRate: number;
  reasoning: string;
}

class SendTimePredictor {
  async predictOptimalSendTime(contactId: string): Promise<SendTimeOptimization> {
    const contact = await db.contacts.findById(contactId);
    const engagementHistory = await this.getEngagementHistory(contactId);
    const patterns = this.analyzeEngagementPatterns(engagementHistory);

    const prediction = await this.mlModel.predict({
      timezone: contact.timezone,
      dayOfWeek: patterns.preferredDays,
      hourOfDay: patterns.preferredHours,
      historicalOpenRate: patterns.avgOpenRate,
    });

    return {
      contactId,
      recommendedSendTime: prediction.optimalTime,
      confidence: prediction.confidence,
      expectedOpenRate: prediction.expectedOpenRate,
      reasoning: `Based on ${engagementHistory.length} previous interactions`,
    };
  }

  private analyzeEngagementPatterns(history: any[]): any {
    const dayOfWeekCounts = {};
    const hourOfDayCounts = {};

    for (const event of history) {
      const day = event.openedAt.getDay();
      const hour = event.openedAt.getHours();
      dayOfWeekCounts[day] = (dayOfWeekCounts[day] || 0) + 1;
      hourOfDayCounts[hour] = (hourOfDayCounts[hour] || 0) + 1;
    }

    return {
      preferredDays: this.getTopN(dayOfWeekCounts, 2),
      preferredHours: this.getTopN(hourOfDayCounts, 3),
      avgOpenRate: history.filter(e => e.opened).length / history.length,
    };
  }
}
```

## Subject Line Performance Prediction

Analyzes subject lines and predicts open rates with improvement recommendations.

{% raw %}

```typescript
class SubjectLinePredictor {
  async predictOpenRate(subjectLine: string, audience: string): Promise<{
    predictedOpenRate: number;
    confidence: number;
    recommendations: string[];
  }> {
    const features = this.extractFeatures(subjectLine);
    const similarSubjects = await this.findSimilarSubjects(subjectLine, audience);

    const prediction = await this.mlModel.predict({
      length: features.length,
      hasEmoji: features.hasEmoji,
      hasNumbers: features.hasNumbers,
      hasQuestion: features.hasQuestion,
      sentiment: features.sentiment,
      urgency: features.urgency,
      personalization: features.personalization,
      audienceSegment: audience,
    });

    return {
      predictedOpenRate: prediction.openRate,
      confidence: prediction.confidence,
      recommendations: this.generateRecommendations(features, prediction),
    };
  }

  private extractFeatures(subjectLine: string): any {
    return {
      length: subjectLine.length,
      hasEmoji: /[\u{1F600}-\u{1F64F}]/u.test(subjectLine),
      hasNumbers: /\d/.test(subjectLine),
      hasQuestion: subjectLine.includes('?'),
      sentiment: this.analyzeSentiment(subjectLine),
      urgency: this.detectUrgency(subjectLine),
      personalization: subjectLine.includes('{{') || subjectLine.includes('%'),
    };
  }

  private generateRecommendations(features: any, prediction: any): string[] {
    const recommendations: string[] = [];

    if (features.length > 60) {
      recommendations.push('Shorten subject line to < 60 characters');
    }
    if (!features.hasEmoji && prediction.emojiImpact > 0.05) {
      recommendations.push('Consider adding an emoji for +5% open rate');
    }
    if (!features.personalization) {
      recommendations.push('Add personalization ({{first_name}}) for +8% open rate');
    }

    return recommendations;
  }
}
```

{% endraw %}

## Deliverability Prediction

Predicts inbox placement rates and identifies risk factors before sending.

```typescript
class DeliverabilityPredictor {
  async predictDeliverability(
    campaignId: string,
    recipientDomain: string
  ): Promise<{
    predictedInboxRate: number;
    predictedSpamRate: number;
    riskFactors: string[];
    recommendations: string[];
  }> {
    const campaign = await db.campaigns.findById(campaignId);
    const domainReputation = await this.getDomainReputation(recipientDomain);
    const senderReputation = await this.getSenderReputation(campaign.fromEmail);
    const contentAnalysis = await this.analyzeContent(campaign.htmlContent);

    const prediction = await this.mlModel.predict({
      senderReputation: senderReputation.score,
      domainReputation: domainReputation.score,
      spamScore: contentAnalysis.spamScore,
      imageToTextRatio: contentAnalysis.imageToTextRatio,
      linkCount: contentAnalysis.linkCount,
      recipientDomain,
    });

    return {
      predictedInboxRate: prediction.inboxRate,
      predictedSpamRate: prediction.spamRate,
      riskFactors: this.identifyRiskFactors(contentAnalysis, senderReputation),
      recommendations: this.generateDeliverabilityRecommendations(prediction),
    };
  }
}
```

## Churn Prediction

Identifies contacts at risk of disengaging and generates retention actions.

```typescript
class ChurnPredictor {
  async predictChurn(contactId: string): Promise<{
    churnProbability: number;
    riskLevel: 'low' | 'medium' | 'high';
    retentionActions: string[];
  }> {
    const contact = await db.contacts.findById(contactId);
    const engagementHistory = await this.getEngagementHistory(contactId, 90);

    const features = {
      daysSinceLastOpen: this.daysSince(contact.lastEmailOpened),
      daysSinceLastClick: this.daysSince(contact.lastEmailClicked),
      openRateLast30Days: this.calculateOpenRate(engagementHistory, 30),
      openRateLast90Days: this.calculateOpenRate(engagementHistory, 90),
      engagementTrend: this.calculateTrend(engagementHistory),
      emailFrequency: engagementHistory.length / 90,
    };

    const prediction = await this.mlModel.predict(features);
    const riskLevel = prediction.churnProbability > 0.7 ? 'high'
      : prediction.churnProbability > 0.4 ? 'medium' : 'low';

    return {
      churnProbability: prediction.churnProbability,
      riskLevel,
      retentionActions: this.generateRetentionActions(riskLevel, features),
    };
  }

  private generateRetentionActions(riskLevel: string, features: any): string[] {
    const actions: string[] = [];

    if (riskLevel === 'high') {
      actions.push('Send re-engagement campaign immediately');
      actions.push('Offer exclusive discount or incentive');
      actions.push('Request feedback on email preferences');
    } else if (riskLevel === 'medium') {
      actions.push('Reduce email frequency');
      actions.push('Send preference center link');
      actions.push('Segment into "at-risk" list for special content');
    }

    if (features.daysSinceLastOpen > 60) {
      actions.push('Send "We miss you" campaign');
    }

    return actions;
  }
}
```
