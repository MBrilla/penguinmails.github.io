---
title: "Database Schema"
description: "PostgreSQL database schema for campaign management with multi-tenant support"
last_modified_date: "2025-01-20"
level: 3
persona: Developers
keywords: database schema, PostgreSQL, campaigns, multi-tenant, Drizzle ORM
---

# Campaign Management Database Schema

The campaign management system uses PostgreSQL with multi-tenant architecture. All tables include tenant_id and workspace_id columns for data isolation. The schema supports complex campaign workflows with conditional branching and comprehensive analytics.

## Campaigns Table

The campaigns table stores the primary campaign configuration including sending settings and scheduling.

```sql
CREATE TABLE campaigns (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  workspace_id UUID NOT NULL REFERENCES workspaces(id),
  name VARCHAR(255) NOT NULL,
  description TEXT,
  campaign_type VARCHAR(50) NOT NULL, -- 'broadcast', 'drip', 'triggered', 'ab_test'
  status VARCHAR(50) NOT NULL DEFAULT 'draft', -- 'draft', 'scheduled', 'active', 'paused', 'completed'
  
  -- Sending Configuration
  segment_id UUID REFERENCES segments(id),
  from_email VARCHAR(255) NOT NULL,
  from_name VARCHAR(255) NOT NULL,
  reply_to_email VARCHAR(255),
  
  -- Scheduling
  send_strategy VARCHAR(50) NOT NULL DEFAULT 'immediate', -- 'immediate', 'scheduled', 'optimized'
  start_date TIMESTAMP WITH TIME ZONE,
  end_date TIMESTAMP WITH TIME ZONE,
  timezone VARCHAR(100) DEFAULT 'UTC',
  
  -- Settings
  track_opens BOOLEAN DEFAULT true,
  track_clicks BOOLEAN DEFAULT true,
  
  -- Metrics (cached aggregates)
  total_contacts INTEGER DEFAULT 0,
  emails_sent INTEGER DEFAULT 0,
  emails_opened INTEGER DEFAULT 0,
  emails_clicked INTEGER DEFAULT 0,
  emails_replied INTEGER DEFAULT 0,
  
  -- Timestamps
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  launched_at TIMESTAMP WITH TIME ZONE,
  completed_at TIMESTAMP WITH TIME ZONE,
  created_by UUID REFERENCES users(id),
  
  CONSTRAINT valid_campaign_type CHECK (campaign_type IN ('broadcast', 'drip', 'triggered', 'ab_test')),
  CONSTRAINT valid_status CHECK (status IN ('draft', 'scheduled', 'active', 'paused', 'completed'))
);
```

## Campaign Steps Table

Each campaign consists of one or more steps that define the sequence of actions.

```sql
CREATE TABLE campaign_steps (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  campaign_id UUID NOT NULL REFERENCES campaigns(id) ON DELETE CASCADE,
  step_order INTEGER NOT NULL,
  step_type VARCHAR(50) NOT NULL, -- 'email', 'wait', 'condition', 'action'
  
  -- Email Step Fields
  template_id UUID REFERENCES templates(id),
  subject_line VARCHAR(500),
  email_content TEXT,
  
  -- Wait Step Fields
  wait_duration JSONB, -- { value: 2, unit: 'days' }
  
  -- Condition Step Fields
  condition_type VARCHAR(100),
  condition_rules JSONB,
  true_next_step_id UUID REFERENCES campaign_steps(id),
  false_next_step_id UUID REFERENCES campaign_steps(id),
  
  -- Action Step Fields
  action_type VARCHAR(100),
  action_config JSONB,
  
  -- Analytics
  emails_sent INTEGER DEFAULT 0,
  emails_opened INTEGER DEFAULT 0,
  emails_clicked INTEGER DEFAULT 0,
  
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  
  CONSTRAINT valid_step_type CHECK (step_type IN ('email', 'wait', 'condition', 'action'))
);
```

## Campaign Contacts Table

Tracks each contact's progress through a campaign.

```sql
CREATE TABLE campaign_contacts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  campaign_id UUID NOT NULL REFERENCES campaigns(id) ON DELETE CASCADE,
  contact_id UUID NOT NULL REFERENCES contacts(id) ON DELETE CASCADE,
  
  status VARCHAR(50) NOT NULL DEFAULT 'enrolled', -- 'enrolled', 'active', 'paused', 'completed', 'bounced', 'unsubscribed'
  current_step_id UUID REFERENCES campaign_steps(id),
  steps_completed INTEGER DEFAULT 0,
  
  -- Timing
  enrolled_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  last_email_sent_at TIMESTAMP WITH TIME ZONE,
  next_email_scheduled_at TIMESTAMP WITH TIME ZONE,
  completed_at TIMESTAMP WITH TIME ZONE,
  
  -- Engagement
  total_emails_sent INTEGER DEFAULT 0,
  total_opens INTEGER DEFAULT 0,
  total_clicks INTEGER DEFAULT 0,
  
  UNIQUE(campaign_id, contact_id)
);
```

## Campaign Analytics Table

Stores aggregated metrics for reporting and trend analysis.

```sql
CREATE TABLE campaign_analytics (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  campaign_id UUID NOT NULL REFERENCES campaigns(id) ON DELETE CASCADE,
  date DATE NOT NULL,
  
  emails_sent INTEGER DEFAULT 0,
  emails_delivered INTEGER DEFAULT 0,
  emails_opened INTEGER DEFAULT 0,
  emails_clicked INTEGER DEFAULT 0,
  emails_replied INTEGER DEFAULT 0,
  emails_bounced INTEGER DEFAULT 0,
  unsubscribes INTEGER DEFAULT 0,
  
  open_rate DECIMAL(5,2),
  click_rate DECIMAL(5,2),
  reply_rate DECIMAL(5,2),
  bounce_rate DECIMAL(5,2),
  
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  
  UNIQUE(campaign_id, date)
);
```

## Campaign Approvals Table

Manages approval workflows for campaigns requiring review.

```sql
CREATE TABLE campaign_approvals (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  campaign_id UUID NOT NULL REFERENCES campaigns(id) ON DELETE CASCADE,
  
  approval_status VARCHAR(50) NOT NULL DEFAULT 'pending', -- 'pending', 'approved', 'rejected'
  requested_by UUID REFERENCES users(id),
  approved_by UUID REFERENCES users(id),
  
  requested_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  decided_at TIMESTAMP WITH TIME ZONE,
  comments TEXT,
  
  CONSTRAINT valid_approval_status CHECK (approval_status IN ('pending', 'approved', 'rejected'))
);
```

## Indexes

Performance indexes optimize common query patterns for campaign processing.

```sql
-- Campaign lookups
CREATE INDEX idx_campaigns_tenant_status ON campaigns(tenant_id, status);
CREATE INDEX idx_campaigns_workspace ON campaigns(workspace_id);

-- Step sequencing
CREATE INDEX idx_campaign_steps_campaign_order ON campaign_steps(campaign_id, step_order);

-- Contact processing
CREATE INDEX idx_campaign_contacts_campaign_status ON campaign_contacts(campaign_id, status);
CREATE INDEX idx_campaign_contacts_next_scheduled ON campaign_contacts(next_email_scheduled_at)
  WHERE status = 'active';

-- Analytics queries
CREATE INDEX idx_campaign_analytics_campaign_date ON campaign_analytics(campaign_id, date);
```

## Related Documentation

- [Execution Engine](./execution-engine.md) - Campaign processing logic
- [API Endpoints](./api-endpoints.md) - REST API documentation
- [Contact Segmentation](/docs/features/leads/contact-segmentation) - Audience targeting
