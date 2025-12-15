---
title: "A/B Testing Technical Implementation"
last_modified_date: "2025-12-15"
level: "3"
persona: "Developers"
---

# A/B Testing Technical Implementation

API integration and automation for A/B testing.

## API Endpoints

### Create A/B Test

```typescript
POST /api/v1/campaigns/:campaignId/ab-test

{
  "test_name": "Subject Line Test Q1",
  "variants": [
    {
      "name": "control",
      "subject": "Quarterly Product Update",
      "weight": 33
    },
    {
      "name": "personalized",
      "subject": "{{firstName}}, See What's New",
      "weight": 33
    },
    {
      "name": "urgent",
      "subject": "Don't Miss: Q1 Updates Inside",
      "weight": 34
    }
  ],
  "test_config": {
    "sample_percentage": 30,
    "duration_hours": 4,
    "win_metric": "open_rate",
    "confidence_threshold": 0.95,
    "auto_deploy_winner": true
  }
}
```

### Get Test Results

```typescript
GET /api/v1/campaigns/:campaignId/ab-test/results

Response:
{
  "test_id": "test_abc123",
  "status": "completed",
  "winner": {
    "variant": "personalized",
    "confidence": 0.97,
    "lift_percentage": 26.5
  },
  "variants": [
    {
      "name": "control",
      "sends": 1000,
      "opens": 220,
      "open_rate": 0.22,
      "clicks": 45,
      "click_rate": 0.045
    },
    {
      "name": "personalized",
      "sends": 1000,
      "opens": 278,
      "open_rate": 0.278,
      "clicks": 62,
      "click_rate": 0.062
    }
  ],
  "statistical_analysis": {
    "p_value": 0.028,
    "confidence_interval": [0.04, 0.11],
    "sample_size_adequate": true
  }
}
```

## SDK Integration

```typescript
import { PenguinMails } from '@penguinmails/sdk';

const client = new PenguinMails({ apiKey: process.env.PENGUINMAILS_API_KEY });

// Create campaign with A/B test
async function createABTestCampaign() {
  const campaign = await client.campaigns.create({
    name: "Q1 Welcome Campaign",
    audience: { segmentId: "new_subscribers" },
    schedule: { type: "immediate" }
  });

  const abTest = await client.abTests.create(campaign.id, {
    testName: "Subject Line Test",
    variants: [
      { name: "control", subject: "Welcome to Our Platform" },
      { name: "personalized", subject: "{{firstName}}, Welcome Aboard!" },
      { name: "emoji", subject: "🎉 Welcome to the Family!" }
    ],
    config: {
      samplePercentage: 30,
      durationHours: 4,
      winMetric: "open_rate",
      confidenceThreshold: 0.95,
      autoDeployWinner: true
    }
  });

  return { campaign, abTest };
}

// Monitor test progress
async function monitorABTest(testId: string) {
  const results = await client.abTests.getResults(testId);
  
  if (results.status === "completed") {
    console.log(`Winner: ${results.winner.variant}`);
    console.log(`Lift: ${results.winner.lift_percentage}%`);
  } else {
    console.log(`Test in progress: ${results.status}`);
    console.log(`Current leader: ${results.current_leader.variant}`);
  }
  
  return results;
}
```

## Webhook Events

```typescript
// A/B test events
{
  "type": "ab_test.completed",
  "data": {
    "test_id": "test_abc123",
    "campaign_id": "camp_xyz789",
    "winner": {
      "variant": "personalized",
      "confidence": 0.97
    },
    "results_url": "https://app.penguinmails.com/tests/test_abc123"
  }
}

{
  "type": "ab_test.winner_deployed",
  "data": {
    "test_id": "test_abc123",
    "deployed_variant": "personalized",
    "recipients_count": 7000
  }
}
```

## Database Schema

```sql
CREATE TABLE ab_tests (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  campaign_id UUID NOT NULL REFERENCES campaigns(id),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  test_name VARCHAR(255) NOT NULL,
  status VARCHAR(50) DEFAULT 'pending',
  sample_percentage INTEGER DEFAULT 30,
  duration_hours INTEGER DEFAULT 4,
  win_metric VARCHAR(50) DEFAULT 'open_rate',
  confidence_threshold DECIMAL(3,2) DEFAULT 0.95,
  auto_deploy_winner BOOLEAN DEFAULT true,
  winner_variant_id UUID,
  started_at TIMESTAMPTZ,
  completed_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE ab_test_variants (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  ab_test_id UUID NOT NULL REFERENCES ab_tests(id),
  name VARCHAR(100) NOT NULL,
  subject VARCHAR(500),
  content_id UUID,
  weight INTEGER DEFAULT 33,
  sends INTEGER DEFAULT 0,
  opens INTEGER DEFAULT 0,
  clicks INTEGER DEFAULT 0,
  conversions INTEGER DEFAULT 0
);
```

---

[← Back to A/B Testing](./README)
