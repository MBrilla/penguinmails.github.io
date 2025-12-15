---
title: "Contact Segmentation Technical Implementation"
description: "Database schema, rules engine, background jobs, and API endpoints for contact segmentation"
last_modified_date: "2025-12-15"
level: "3"
parent: "contact-segmentation"
---

# Technical Implementation

Complete technical reference for implementing contact segmentation, including database schema, rules engine, and API endpoints.

---

## Database Schema

```sql
-- Segments table
CREATE TABLE segments (
  id UUID PRIMARY KEY,
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  workspace_id UUID REFERENCES workspaces(id),

  name VARCHAR(255) NOT NULL,
  description TEXT,

  -- Segment type
  segment_type VARCHAR(50), -- dynamic, static

  -- Rules (for dynamic segments)
  rules JSONB, -- Filtering criteria

  -- Metadata
  contact_count INTEGER DEFAULT 0,
  last_recalculated_at TIMESTAMP,

  -- Status
  is_active BOOLEAN DEFAULT TRUE,

  created_by UUID REFERENCES users(id),
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_segments_tenant ON segments(tenant_id);
CREATE INDEX idx_segments_workspace ON segments(workspace_id);
CREATE INDEX idx_segments_type ON segments(segment_type);

-- Segment membership (for static segments or cached dynamic)
CREATE TABLE segment_contacts (
  id UUID PRIMARY KEY,
  segment_id UUID NOT NULL REFERENCES segments(id),
  contact_id UUID NOT NULL REFERENCES contacts(id),

  -- For dynamic segments, track when added
  added_at TIMESTAMP DEFAULT NOW(),
  added_via VARCHAR(50), -- rule_match, manual, import

  UNIQUE(segment_id, contact_id)
);

CREATE INDEX idx_segment_contacts_segment ON segment_contacts(segment_id);
CREATE INDEX idx_segment_contacts_contact ON segment_contacts(contact_id);

-- Segment performance tracking
CREATE TABLE segment_analytics (
  id UUID PRIMARY KEY,
  segment_id UUID NOT NULL REFERENCES segments(id),

  -- Timeframe
  date DATE NOT NULL,

  -- Metrics
  contact_count INTEGER,
  emails_sent INTEGER DEFAULT 0,
  emails_delivered INTEGER DEFAULT 0,
  opens INTEGER DEFAULT 0,
  clicks INTEGER DEFAULT 0,
  conversions INTEGER DEFAULT 0,
  revenue DECIMAL(10,2) DEFAULT 0,

  -- Rates
  open_rate DECIMAL(5,2),
  click_rate DECIMAL(5,2),
  conversion_rate DECIMAL(5,2),

  created_at TIMESTAMP DEFAULT NOW(),

  UNIQUE(segment_id, date)
);

CREATE INDEX idx_segment_analytics_segment_date ON segment_analytics(segment_id, date);
```

---

## Segment Rules Engine

### Type Definitions

```typescript
interface SegmentRule {
  field: string; // contact.email, contact.created_at, customField.industry, etc.
  operator: 'equals' | 'not_equals' | 'contains' | 'greater_than' | 'less_than' | 'between' | 'in' | 'not_in';
  value: any;
  dataType: 'string' | 'number' | 'date' | 'boolean' | 'array';
}

interface SegmentConditionGroup {
  match: 'all' | 'any'; // AND or OR
  conditions: (SegmentRule | SegmentConditionGroup)[];
}

interface SegmentDefinition {
  match: 'all' | 'any';
  groups: SegmentConditionGroup[];
  exclusions?: SegmentConditionGroup[];
}
```

### Segment Engine Implementation

```typescript
class SegmentEngine {
  async evaluateSegment(segmentId: string): Promise<Contact[]> {
    const segment = await db.segments.findById(segmentId);

    if (segment.segmentType === 'static') {
      return this.getStaticContacts(segmentId);
    }

    // Dynamic segment - evaluate rules
    const query = this.buildQuery(segment.rules);
    const contacts = await db.contacts.findAll(query);

    // Update cached membership
    await this.updateSegmentMembership(segmentId, contacts);

    return contacts;
  }

  private buildQuery(rules: SegmentDefinition): any {
    const conditions = this.buildConditions(rules.groups, rules.match);

    let query: any = {
      where: conditions,
    };

    // Apply exclusions
    if (rules.exclusions) {
      const exclusionConditions = this.buildConditions(
        rules.exclusions.groups,
        rules.exclusions.match
      );
      query.where = {
        [Op.and]: [
          conditions,
          { [Op.not]: exclusionConditions },
        ],
      };
    }

    return query;
  }

  private buildConditions(
    groups: SegmentConditionGroup[],
    match: 'all' | 'any'
  ): any {
    const operator = match === 'all' ? Op.and : Op.or;

    const groupConditions = groups.map(group => {
      return this.buildGroupConditions(group);
    });

    return { [operator]: groupConditions };
  }

  private buildGroupConditions(group: SegmentConditionGroup): any {
    const operator = group.match === 'all' ? Op.and : Op.or;

    const conditions = group.conditions.map(condition => {
      if ('field' in condition) {
        return this.buildRuleCondition(condition as SegmentRule);
      } else {
        return this.buildGroupConditions(condition as SegmentConditionGroup);
      }
    });

    return { [operator]: conditions };
  }

  private buildRuleCondition(rule: SegmentRule): any {
    const { field, operator, value } = rule;

    // Parse field path (e.g., "contact.email" or "customField.industry")
    const [entity, property] = field.split('.');

    let condition: any = {};

    if (entity === 'customField') {
      // Query JSONB custom fields
      condition = {
        [`customFields.${property}`]: this.getOperatorCondition(operator, value),
      };
    } else {
      // Standard field
      condition = {
        [property]: this.getOperatorCondition(operator, value),
      };
    }

    return condition;
  }

  private getOperatorCondition(operator: string, value: any): any {
    switch (operator) {
      case 'equals':
        return value;
      case 'not_equals':
        return { [Op.ne]: value };
      case 'contains':
        return { [Op.iLike]: `%${value}%` };
      case 'greater_than':
        return { [Op.gt]: value };
      case 'less_than':
        return { [Op.lt]: value };
      case 'between':
        return { [Op.between]: value }; // value = [min, max]
      case 'in':
        return { [Op.in]: value }; // value = array
      case 'not_in':
        return { [Op.notIn]: value };
      default:
        throw new Error(`Unknown operator: ${operator}`);
    }
  }

  private async updateSegmentMembership(
    segmentId: string,
    contacts: Contact[]
  ): Promise<void> {
    // Remove old memberships
    await db.segmentContacts.destroy({
      where: { segmentId },
    });

    // Add new memberships
    await db.segmentContacts.bulkCreate(
      contacts.map(contact => ({
        segmentId,
        contactId: contact.id,
        addedVia: 'rule_match',
      }))
    );

    // Update contact count
    await db.segments.update(segmentId, {
      contactCount: contacts.length,
      lastRecalculatedAt: new Date(),
    });
  }
}
```

---

## Background Jobs

```typescript
// Recalculate dynamic segments
cron.schedule('*/30 * * * *', async () => {  // Every 30 minutes
  const dynamicSegments = await db.segments.findAll({
    where: {
      segmentType: 'dynamic',
      isActive: true,
    },
  });

  for (const segment of dynamicSegments) {
    await segmentQueue.add('recalculate-segment', {
      segmentId: segment.id,
    });
  }
});

// Queue worker
segmentQueue.process('recalculate-segment', async (job) => {
  const { segmentId } = job.data;

  const engine = new SegmentEngine();
  await engine.evaluateSegment(segmentId);
});

// Calculate segment analytics daily
cron.schedule('0 2 * * *', async () => {  // 2 AM daily
  const segments = await db.segments.findAll({ where: { isActive: true } });

  for (const segment of segments) {
    await calculateSegmentAnalytics(segment.id);
  }
});
```

---

## API Endpoints

### Create Segment

```typescript
app.post('/api/segments', authenticate, async (req, res) => {
  const { name, description, segmentType, rules, contactIds } = req.body;

  const segment = await db.segments.create({
    tenantId: req.user.tenantId,
    workspaceId: req.body.workspaceId,
    name,
    description,
    segmentType,
    rules: segmentType === 'dynamic' ? rules : null,
    createdBy: req.user.id,
  });

  // For static segments, add contacts
  if (segmentType === 'static' && contactIds) {
    await db.segmentContacts.bulkCreate(
      contactIds.map(contactId => ({
        segmentId: segment.id,
        contactId,
        addedVia: 'manual',
      }))
    );

    await db.segments.update(segment.id, {
      contactCount: contactIds.length,
    });
  }

  // For dynamic segments, trigger calculation
  if (segmentType === 'dynamic') {
    await segmentQueue.add('recalculate-segment', {
      segmentId: segment.id,
    });
  }

  return res.json(segment);
});
```

### Get Segment Contacts

```typescript
app.get('/api/segments/:id/contacts', authenticate, async (req, res) => {
  const segment = await db.segments.findById(req.params.id);

  if (segment.tenantId !== req.user.tenantId) {
    return res.status(403).json({ error: 'Forbidden' });
  }

  const contacts = await db.contacts.findAll({
    include: [{
      model: db.segmentContacts,
      where: { segmentId: req.params.id },
    }],
    offset: req.query.offset || 0,
    limit: req.query.limit || 50,
  });

  return res.json({
    segment,
    contacts,
    total: segment.contactCount,
  });
});
```

---

## Related Documentation

- [Quick Start Guide](./quick-start-guide) - Create your first segment
- [Advanced Segmentation](./advanced-segmentation) - Complex rule patterns
- [Segment Analytics](./segment-analytics) - Performance tracking

---

**Related:** [Leads Management](/docs/features/leads/leads-management) | [Import/Export](/docs/features/leads/import-export/overview)
