---
title: "Feature Tables"
description: "Feature-specific table schemas"
last_modified_date: "2025-12-04"
level: "3"
keywords: "feature tables, schema, relationships"
---

# Feature Tables

## Template Organization

```sql
-- Template-to-folder mapping (UUID → BIGINT optimization)
CREATE TABLE template_folders (
    id BIGINT PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
    template_id UUID REFERENCES templates(id) ON DELETE CASCADE,
    folder_id BIGINT REFERENCES folders(id) ON DELETE CASCADE,
    updated TIMESTAMP WITH TIME ZONE,
    created TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    UNIQUE(template_id, folder_id)
);

-- Template tagging system (UUID → BIGINT optimization)
CREATE TABLE tags (
    id BIGINT PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
    value TEXT NOT NULL,
    description TEXT,
    name TEXT,
    tenant_id UUID,
    created TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated TIMESTAMP WITH TIME ZONE,
    is_default BOOLEAN DEFAULT FALSE
);

CREATE TABLE template_tags (
    id BIGINT PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
    template_id UUID REFERENCES templates(id) ON DELETE CASCADE,
    tag_id BIGINT REFERENCES tags(id) ON DELETE CASCADE,
    tenant_id UUID NOT NULL,
    updated TIMESTAMP WITH TIME ZONE,
    created TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    UNIQUE(template_id, tag_id)
);

```

#### Eta Template Rendering Storage

```sql
CREATE TABLE email_templates (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL,
    name TEXT NOT NULL,
    subject TEXT NOT NULL,
    body TEXT NOT NULL,          -- raw .eta template body
    async BOOLEAN DEFAULT FALSE, -- if template uses async includes
    created TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    UNIQUE (tenant_id, name)
);


```

```sql
CREATE TABLE template_dictionaries (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL,
    key TEXT NOT NULL,
    value JSONB NOT NULL,
    created TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_template_dictionaries_tenant ON template_dictionaries(tenant_id);


```

---

### Campaign Execution System (OLTP Metadata)

#### **campaign_sequence_steps** - Campaign Execution Orchestration

*(Note: Email message analytics moved to Content Database as `email_messages` and `email_content`)*

---

## Campaign Management

### **Campaign Orchestration**

#### **campaigns** - Campaign Definitions

```sql
CREATE TABLE campaigns (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL,
    company_id UUID REFERENCES companies(id) ON DELETE CASCADE,
    name VARCHAR(200) NOT NULL,
    description TEXT,
    status VARCHAR(50) CHECK (status IN ('draft', 'scheduled', 'sending', 'completed', 'paused')),
    scheduled TIMESTAMP WITH TIME ZONE,
    completed TIMESTAMP WITH TIME ZONE,
    created TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);


```

#### **campaign_sequence_steps** - Individual Campaign Steps

```sql
CREATE TABLE campaign_sequence_steps (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    campaign_id UUID REFERENCES campaigns(id) ON DELETE CASCADE,
    step_order INTEGER NOT NULL,
    step_type VARCHAR(50) CHECK (step_type IN ('email', 'wait', 'condition')),
    email_account_id UUID REFERENCES email_accounts(id),
    template_id UUID REFERENCES templates(id),
    subject_override VARCHAR(500),
    from_name_override VARCHAR(100),
    wait_duration_hours INTEGER,
    wait_until_date TIMESTAMP WITH TIME ZONE,
    condition_type VARCHAR(50),
    condition_value VARCHAR(100),
    send_condition TEXT,
    is_active BOOLEAN DEFAULT TRUE,
    executed_count INTEGER DEFAULT 0,
    last_executed TIMESTAMP WITH TIME ZONE,
    created TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);


```

**Step Types:**

- **email**: Send email using template

- **wait**: Delay before next step

- **condition**: Conditional logic based on lead behavior

---

## Billing & Subscription Management

### **Subscription Plans**

#### **plans** - Subscription Plan Definitions

```sql
CREATE TABLE plans (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL,
    slug VARCHAR(100) NOT NULL UNIQUE,
    description TEXT,
    max_users INTEGER,
    max_domains INTEGER,
    max_campaigns_per_month INTEGER,
    api_rate_limit INTEGER,
    max_email_accounts INTEGER,
    max_emails_per_month INTEGER,
    max_warmups INTEGER,
    max_contacts INTEGER,
    max_storage_gb INTEGER,
    api_access BOOLEAN DEFAULT FALSE,
    priority_support BOOLEAN DEFAULT FALSE,
    advanced_analytics BOOLEAN DEFAULT FALSE,
    white_label BOOLEAN DEFAULT FALSE,
    price_monthly INTEGER,
    price_yearly INTEGER,
    notes TEXT,  -- Additional plan details and notes
    is_active BOOLEAN DEFAULT TRUE,
    stripe_product_id VARCHAR(255),  -- Stripe Product reference for dashboard/portal links
    created TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Stripe Product integration index
CREATE INDEX idx_plans_stripe_product
    ON plans(stripe_product_id)
    WHERE stripe_product_id IS NOT NULL;


```

##### Stripe Product Integration

The `stripe_product_id` field links PenguinMails plans to Stripe Products for billing management:

**Purpose**:

- Enables "View Plan in Stripe" or "Manage Subscription" redirects to Stripe Dashboard
- Product ID remains stable even when prices change in Stripe
- Links to Stripe Customer Portal for self-service billing

**Why NOT storing `stripe_price_id`**:

- Stripe manages all pricing via Checkout and Customer Portal
- PenguinMails UI displays `price_monthly` and `price_yearly` from local database  
- When user clicks "Subscribe" or "Change Plan", redirects to Stripe Checkout
- Stripe handles price selection, proration calculations, and payment collection
- **Single source of truth for prices**: Stripe

**Admin Workflow**:

1. Create Stripe Product via Stripe Dashboard
2. Create Prices (monthly, yearly) for the Product in Stripe
3. Copy Product ID into `plans.stripe_product_id` field
4. Set `is_active = true` to make plan available for purchase

##### Plan Lifecycle with `is_active`

The `is_active` field provides flexible plan management for seasonal and exclusive offerings:

**Active Plans** (`is_active = true`):

- Visible in tenant purchase/upgrade options
- Available for new subscriptions
- Default state for standard plans

**Inactive Plans** (`is_active = false`):

- **Hidden** from purchase UI
- **Existing subscribers can still renew** and maintain their subscription
- Provides exclusive access to grandfathered customers

**Use Cases**:

- **Seasonal Plans**: "Black Friday 2025" - deactivate after November, reactivate next year
- **Exclusive Plans**: "Founder Plan" - only for early adopters, inactive but renewable by existing customers
- **A/B Test Plans**: Test new pricing tier, deactivate if underperforming
- **Limited Offers**: "Summer Promo 2026" - activate for 3 months, then deactivate
- **Grandfathered Pricing**: Lock in special pricing for loyal customers while hiding from new sign-ups

**Benefits over "deprecated" flag**:

- More flexible (can reactivate easily)
- Supports temporary/seasonal plans naturally
- Existing subscribers retain exclusive access
- Simpler schema (reuses existing field)

#### Plans Table - Stripe Product Integration

**Stripe Product Mapping**:

The `stripe_product_id` field links PenguinMails plans to Stripe Products for dashboard and customer portal integration:

- **Purpose**: Enables "View Plan in Stripe" or "Manage Subscription" redirect links
- **Stability**: Product ID remains stable even when prices change
- **Workflow**: Admin creates Stripe Product, copies Product ID into `plans.stripe_product_id`

**Why NOT storing `stripe_price_id`**:

- Stripe manages pricing via Checkout and Customer Portal
- PenguinMails UI displays `price_monthly`/`price_yearly` from local `plans` table
- User clicks "Subscribe" → redirect to Stripe Checkout
- Stripe handles price selection, proration, and payment collection
- **Benefit**: Single source of truth for prices (Stripe)

**Admin Workflow**:

1. Create Stripe Product via Stripe Dashboard
2. Create Prices (monthly, yearly) for the Product in Stripe
3. Copy Product ID into `plans.stripe_product_id` field
4. Set `is_active = true` to make plan available for purchase

**Plan Lifecycle with `is_active`**:

- `is_active = true`: Plan visible in purchase options
- `is_active = false`: Plan hidden from purchase UI BUT existing subscribers can renew

**Use Cases for Inactive Plans**:

- **Seasonal Plans**: "Black Friday 2025" - deactivate after November, reactivate next year
- **Exclusive Plans**: "Founder Plan" - only for early adopters, inactive but renewable for existing subscribers
- **A/B Test Plans**: Test new tier, deactivate if underperforming
- **Limited Offers**: "Summer Promo 2026" - activate for 3 months, then deactivate

**Benefits**:

- More flexible than "deprecated" (which implies permanent status)
- Supports grandfathered pricing for loyal customers
- Easy to reactivate seasonal plans
- Existing subscribers retain access even when plan is inactive

---

#### **subscriptions** - Active Tenant Subscriptions

```sql
CREATE TABLE subscriptions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID REFERENCES tenants(id) ON DELETE CASCADE,
    plan_id UUID REFERENCES plans(id),
    pending_plan_id UUID REFERENCES plans(id),
    status VARCHAR(50) CHECK (status IN ('active', 'past_due', 'canceled', 'unpaid')),
    current_period_start TIMESTAMP WITH TIME ZONE,
    current_period_end TIMESTAMP WITH TIME ZONE,
    cancel_at_period_end BOOLEAN DEFAULT FALSE,
    cancel_reason TEXT,  -- User-provided cancellation reason
    cancel_date TIMESTAMP WITH TIME ZONE,  -- When cancellation was initiated
    billing_contact_user_id UUID REFERENCES users(id),
    stripe_subscription_id VARCHAR(255),
    created TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);


```

#### **subscription_addons** - Additional Features

```sql
CREATE TABLE subscription_addons (
    id VARCHAR(255) PRIMARY KEY,
    subscription_id UUID REFERENCES subscriptions(id) ON DELETE CASCADE,
    addon_type VARCHAR(50) NOT NULL,
    quantity INTEGER DEFAULT 1,
    price_monthly DECIMAL(10,2),
    stripe_price_id VARCHAR(255),
    is_active BOOLEAN DEFAULT TRUE,
    created TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);


```

#### **payments** - Payment Records

```sql
CREATE TABLE payments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    subscription_id UUID REFERENCES subscriptions(id) ON DELETE CASCADE,
    initiated_by_user_id UUID REFERENCES users(id),
    amount DECIMAL(10,2) NOT NULL,
    currency VARCHAR(3) DEFAULT 'USD',
    status VARCHAR(20) NOT NULL,
    processed TIMESTAMP WITH TIME ZONE,
    description TEXT,
    stripe_payment_intent_id VARCHAR(255),
    billing_period_start TIMESTAMP WITH TIME ZONE,
    billing_period_end TIMESTAMP WITH TIME ZONE,
    created TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);


```

---

## Infrastructure Management

### **VPS & IP Management**
