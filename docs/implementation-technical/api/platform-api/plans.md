---
title: "Platform Admin API - Plans"
description: "Manage subscription plans, feature limits, and Stripe mappings"
last_modified_date: "2025-12-16"
level: "3"
persona: "Technical Teams"
---

# Platform Admin API - Plans

**Manage subscription plans, feature limits, and Stripe mappings.**

## Base URL

`https://api.penguinmails.com/api/v1/platform/admin/plans`

## Authentication

Requires `Bearer Token` with `Super Admin` role.

## Related Documentation

- **UI Route**: [Platform Administration](/docs/design/routes/platform-admin) - Complete admin dashboard specifications including plan management views.

---

## Endpoints

### List All Plans

`GET /`

Retrieve a list of all subscription plans, including inactive ones.

**Response**:

```json
{
  "data": [
    {
      "id": "uuid-1234",
      "name": "Pro Plan",
      "slug": "pro",
      "stripe_product_id": "prod_StripeID123",
      "is_active": true,
      "price_monthly": 2900,
      "price_yearly": 29000,
      "limits": {
        "max_users": 5,
        "max_domains": 3,
        "max_emails_per_month": 50000
      }
    },
    {
      "id": "uuid-5678",
      "name": "Legacy Starter",
      "slug": "starter-legacy",
      "is_active": false
    }
  ]
}
```

### Get Plan Details

`GET /{id}`

Retrieve details for a specific plan.

### Create Plan

`POST /`

Create a new subscription plan definition.

**Body**:

```json
{
  "name": "Enterprise Tier",
  "slug": "enterprise",
  "description": "For large organizations",
  "stripe_product_id": "prod_Ent123",
  "price_monthly": 9900,
  "price_yearly": 99000,
  "is_active": false,
  "max_users": 20,
  "max_domains": 10,
  "max_campaigns_per_month": 1000,
  "max_emails_per_month": 200000,
  "api_access": true,
  "priority_support": true
}
```

### Update Plan

`PUT /{id}`

Update an existing plan. Note: Changing limits affects all current subscribers unless grandfathering logic is applied at the application level.

**Body**:

```json
{
  "name": "Enterprise Tier (2025)",
  "is_active": true,
  "max_domains": 15
}
```

## Error Handling

- `400 Bad Request`: Invalid slug or missing required fields.
- `401 Unauthorized`: Missing or invalid token.
- `403 Forbidden`: User is not a Super Admin.
- `404 Not Found`: Plan ID does not exist.
