---
title: "Webhook Overview Route"
description: "Route specification for the webhook list page with status monitoring and health metrics"
last_modified_date: "2025-01-20"
level: 3
persona: Developers
keywords: webhooks, overview, status monitoring, health metrics
---

# Webhook Overview Route

**Route:** `/dashboard/settings/webhooks`

**Access:** Admin, Developer

**User Story:** *"As a developer, I want to see all configured webhooks and their health status, so I can quickly identify and fix issues."*

## Overview Stats

The top row displays key metrics for quick health assessment.

**Metrics Cards:**
- **Active Webhooks**: Count of active webhooks (e.g., "3")
- **Events Delivered Today**: Daily delivery count (e.g., "12,345")
- **Success Rate**: Percentage of successful deliveries (e.g., "99.2%")
- **Failing Webhooks**: Count with issues (red if > 0)

## Webhook List

Each webhook displays as a card with status and quick actions.

**Webhook Card Contents:**
- **Webhook Name**: "CRM Contact Sync"
- **Endpoint**: `https://yourapp.com/webhooks/penguinmails`
- **Status Badge**:
  - **Active** (Green): Receiving events successfully
  - **Paused** (Gray): Manually disabled
  - **Failing** (Red): Recent delivery failures
  - **Inactive** (Gray): Auto-paused after 100 failures

**Event Subscriptions:**
- Summary: "Subscribed to 7 events: email.opened, email.clicked, ..."
- **"View All" Link**: Expands full event list

**Health Indicators:**
- **Success Rate**: "99.2%" (Last 24 hours)
- **Last Delivery**: "2 minutes ago"
- **Failed Deliveries**: "3 in last 24 hours"

**Quick Actions:**
- **Test** button: Send test event
- **Pause/Resume** toggle
- **View Details** link
- **Delete** button (with confirmation)

**"+ Create Webhook" Button**: Navigate to webhook creation

## Quick Filters

**Filter Bar:**
- **Status**: All, Active, Paused, Failing
- **Events**: Filter by subscribed event types
- **Search**: Search by name or endpoint URL

## ESP Webhooks Notice

**Info Banner:**
"Looking for ESP webhooks (Postmark, Mailgun)? [Configure ESP webhooks →](/dashboard/settings/integrations/esp/webhooks)"

## User Journey Context

Regular monitoring (daily/weekly) to ensure webhook health.

## Related Documentation

- [Webhook System Overview](/docs/features/integrations/webhook-system)
- [Event Reference](/docs/features/integrations/webhook-system#event-reference)
