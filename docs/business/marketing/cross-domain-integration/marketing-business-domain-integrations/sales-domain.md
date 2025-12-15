---
title: Sales Domain Integration
description: CRM synchronization and campaign attribution patterns
last_modified_date: 2025-01-15
level: 3
persona: ["developer", "analyst"]
keywords: [sales, CRM, Salesforce, HubSpot, attribution]
---

{% raw %}

# Sales Domain Integration

Real-time bidirectional integration patterns for CRM synchronization and multi-touch campaign attribution.

## CRM Lead Synchronization

**Pattern:** Real-time bidirectional sync with conflict resolution

### Lead Processing Pipeline

```typescript
interface LeadProcessingWorkflow {
  step1: "Lead Capture & Validation"
  step2: "Duplicate Detection & Merging"
  step3: "Lead Scoring & Qualification"
  step4: "CRM Record Creation/Update"
  step5: "Sales Rep Assignment"
  step6: "Automated Follow-up Triggers"
}

const leadProcessingPipeline = {
  async processLead(rawLead: RawLeadData): Promise<ProcessedLead> {
    const validatedLead = await validateLeadData(rawLead);

    const duplicateCheck = await checkCRMForDuplicates(validatedLead.email);
    if (duplicateCheck.found) {
      return await mergeWithExistingLead(validatedLead, duplicateCheck.record);
    }

    const leadScore = await calculateLeadScore(validatedLead);

    const crmRecord = await syncToCRM({
      ...validatedLead,
      lead_score: leadScore,
      source: 'marketing_platform',
      sync_timestamp: new Date().toISOString()
    });

    const assignment = await assignSalesRep(crmRecord, leadScore);
    await triggerFollowUpSequence(crmRecord.id, assignment.rep_id);

    return crmRecord;
  }
};
```

### Salesforce Field Mapping

```json
{
  "salesforce_mapping": {
    "lead_fields": {
      "marketing_platform.lead_id": "Marketing_Platform_ID__c",
      "marketing_platform.email": "Email",
      "marketing_platform.first_name": "FirstName",
      "marketing_platform.last_name": "LastName",
      "marketing_platform.company": "Company",
      "marketing_platform.lead_score": "Lead_Score__c",
      "marketing_platform.source": "LeadSource",
      "marketing_platform.campaign_id": "Campaign_ID__c"
    },
    "custom_objects": {
      "lead_interactions": {
        "object_name": "Lead_Interaction__c",
        "fields": {
          "interaction_type": "Text",
          "interaction_date": "DateTime",
          "campaign_id": "Lookup",
          "engagement_score": "Number"
        }
      }
    }
  }
}
```

### HubSpot Property Mapping

```json
{
  "hubspot_mapping": {
    "contact_properties": {
      "marketing_platform_id": "marketing_platform_id",
      "lead_score": "hs_lead_score",
      "lead_source": "hs_analytics_source",
      "first_conversion_date": "first_conversion_date",
      "recent_conversion_date": "recent_conversion_date"
    },
    "custom_properties": {
      "marketing_platform_lead_score": {
        "name": "marketing_platform_lead_score",
        "type": "number",
        "fieldType": "number"
      },
      "last_campaign_interaction": {
        "name": "last_campaign_interaction",
        "type": "string",
        "fieldType": "text"
      }
    }
  }
}
```

## Campaign Attribution Tracking

**Pattern:** Multi-touch attribution with revenue tracking

### Attribution Engine

```typescript
interface AttributionTracking {
  trackTouchpoint(touchpoint: TouchpointEvent): Promise<void>;
  calculateRevenueAttribution(campaignId: string): Promise<RevenueAttribution>;
  syncAttributionToCRM(opportunityId: string, attribution: RevenueAttribution): Promise<void>;
}

class CampaignAttributionEngine implements AttributionTracking {
  async trackTouchpoint(touchpoint: TouchpointEvent): Promise<void> {
    await this.database.touchpoints.insert({
      contact_id: touchpoint.contact_id,
      campaign_id: touchpoint.campaign_id,
      touchpoint_type: touchpoint.type,
      timestamp: touchpoint.timestamp,
      metadata: touchpoint.metadata,
      weight: this.calculateTouchpointWeight(touchpoint)
    });

    await this.updateRealTimeAttribution(touchpoint.contact_id);
  }

  async calculateRevenueAttribution(campaignId: string): Promise<RevenueAttribution> {
    const attributionModel = await this.getAttributionModel(campaignId);

    return {
      campaign_id: campaignId,
      total_revenue: await this.calculateTotalRevenue(campaignId),
      attribution_breakdown: await this.calculateMultiTouchAttribution(attributionModel),
      model_used: attributionModel.type,
      confidence_score: attributionModel.confidence
    };
  }
}
```

### First-Touch Attribution Query

```sql
WITH first_touch AS (
  SELECT
    contact_id,
    MIN(timestamp) as first_interaction,
    campaign_id
  FROM touchpoints
  WHERE campaign_id = ?
    AND contact_id IN (SELECT contact_id FROM opportunities WHERE is_closed_won = true)
  GROUP BY contact_id, campaign_id
)
SELECT
  ft.campaign_id,
  COUNT(*) as attributed_customers,
  SUM(o.amount) as attributed_revenue,
  AVG(o.amount) as avg_deal_size
FROM first_touch ft
JOIN opportunities o ON o.contact_id = ft.contact_id
WHERE o.is_closed_won = true
GROUP BY ft.campaign_id;
```

### Multi-Touch Linear Attribution Query

```sql
WITH touchpoint_sequence AS (
  SELECT
    t.contact_id,
    t.campaign_id,
    t.touchpoint_type,
    t.timestamp,
    ROW_NUMBER() OVER (PARTITION BY t.contact_id ORDER BY t.timestamp) as sequence_order,
    COUNT(*) OVER (PARTITION BY t.contact_id) as total_touchpoints
  FROM touchpoints t
  WHERE t.campaign_id = ?
    AND t.contact_id IN (SELECT contact_id FROM opportunities WHERE is_closed_won = true)
),
weighted_attribution AS (
  SELECT
    ts.campaign_id,
    ts.touchpoint_type,
    (1.0 / ts.total_touchpoints) as attribution_weight,
    o.amount,
    o.amount * (1.0 / ts.total_touchpoints) as weighted_revenue
  FROM touchpoint_sequence ts
  JOIN opportunities o ON o.contact_id = ts.contact_id
  WHERE o.is_closed_won = true
)
SELECT
  campaign_id,
  touchpoint_type,
  SUM(weighted_revenue) as attributed_revenue,
  SUM(attribution_weight) as total_weight
FROM weighted_attribution
GROUP BY campaign_id, touchpoint_type;
```

## Related Documentation

- [Integration Overview](./)
- [Finance Domain Integration](./finance-domain)
- [Data Synchronization](./data-synchronization)

{% endraw %}
