---
title: "Plan Management Routes"
description: "Admin routes for managing subscription plans and pricing"
last_modified_date: "2025-12-16"
level: "2"
persona: "Technical Teams"
---

# Plan Management Routes

## `/dashboard/admin/plans` - Plan Management

**User Story**: *"As a product manager, I want to manage subscription plans and pricing without code changes, so I can launch new tiers for Black Friday."*

### Plans Table

- **Columns**: Name, Slug, Price (Monthly/Yearly), Stripe Product ID, Status (Active/Inactive), Subscribers, Actions.
- **Status Indicators**:
  - **Green**: Active (Publicly purchasable)
  - **Grey**: Inactive (Hidden from new customers)
- **Actions**:
  - **"Create New Plan" Button**: Navigates to plan creation form
  - **Row Actions**: Edit plan details, Toggle active status

**Related Documentation**:
- [Plan Management Feature](/docs/features/admin/plan-management)
- [Billing Schema](/docs/implementation-technical/database-infrastructure/oltp-database/schema-guide)
- [Stripe Integration](/docs/features/payments/stripe-integration)

**Technical Integration**:
- **Stripe Product Sync**: Plans store `stripe_product_id` to generate checkout links
- **Feature Gating**: Plans table defines tenant limits (max emails, max users, etc.)
- **Cache Invalidation**: Updates trigger cache invalidation for public-facing pricing pages

---

## `/dashboard/admin/plans/[id]` - Edit Plan

**User Story**: *"As an admin, I want to update feature limits for the Pro plan or deactivate a seasonal plan."*

### Plan Details Form

- **General Info**:
  - Name (e.g., "Pro Plan")
  - Slug (e.g., "pro", used in URLs)
  - Description (marketing copy)

- **Stripe Mapping**:
  - `stripe_product_id` (Required): Links to Stripe Product
  - **Note**: Prices managed in Stripe, but displayed here for reference

- **Feature Limits**:
  - Max Users per tenant
  - Max Domains per tenant
  - Max Campaigns/Month
  - Max Emails/Month
  - Storage Limit (GB)

- **Feature Flags**:
  - API Access enabled
  - Priority Support
  - White Label branding
  - Advanced Analytics

- **Lifecycle Control**:
  - **Is Active Toggle**: Controls visibility in purchase UI

**Validation Rules**:
- `slug` must be unique across all plans
- `stripe_product_id` format validation
- Feature limits must be positive integers

**Related Documentation**:
- [Subscription Management](/docs/features/payments/subscription-management)
- [Feature Gating Implementation](/docs/implementation-technical/feature-gating)

**Technical Integration**:
- **Database**: `plans` table in OLTP database
- **Cache Strategy**: Heavily cached for public pages, admin updates invalidate cache tags
- **Webhooks**: Stripe product updates can sync back to PenguinMails
