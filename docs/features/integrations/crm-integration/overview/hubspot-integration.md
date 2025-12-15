---
title: "HubSpot Integration"
description: "OAuth authentication, property mapping, timeline events, and workflow triggers for HubSpot"
last_modified_date: "2025-01-20"
level: 3
persona: Developers
keywords: HubSpot, OAuth, contacts, companies, timeline events, workflows
---

# HubSpot Integration

HubSpot is the secondary priority CRM for Q1 2026, providing contact and company sync with timeline events and workflow integration.

## OAuth Authentication Flow

```yaml
hubspot_oauth:
  authorization_endpoint: https://app.hubspot.com/oauth/authorize
  token_endpoint: https://api.hubapi.com/oauth/v1/token
  scopes:
    - contacts  # Read/write contacts
    - timeline  # Create timeline events
    - automation  # Trigger workflows
  flow:
    1. User clicks "Connect HubSpot"
    2. Redirect to HubSpot OAuth
    3. User selects HubSpot account
    4. Receive authorization code
    5. Exchange for access + refresh tokens
    6. Store encrypted tokens
```

## Object Mapping

### Contacts

```yaml
hubspot_contact:
  standard_properties:
    email → email
    firstname → first_name
    lastname → last_name
    company → company
    phone → phone
    jobtitle → job_title
    city → city
    state → state
    country → country

  custom_properties:
    penguinmails_lead_score → lead_score
    penguinmails_last_email_opened → last_email_opened
    penguinmails_total_opens → total_opens
    penguinmails_total_clicks → total_clicks
    penguinmails_unsubscribed → unsubscribed
    penguinmails_last_campaign → last_campaign_id
```

### Companies

```yaml
hubspot_company:
  properties:
    name → company_name
    domain → company_domain
    industry → industry
    numberofemployees → company_size
    annualrevenue → annual_revenue
```

### Timeline Events

```yaml
hubspot_timeline_event:
  event_types:
    - email_sent
    - email_opened
    - email_clicked
    - email_replied
    - email_bounced

  event_template:
    eventTypeId: "penguinmails_email_activity"
    email: contact_email
    timestamp: event_timestamp
    extraData:
      campaign_name: campaign_name
      subject_line: subject
      action: action_type
```

## Sync Rules

```yaml
sync_rules:
  contact_sync:
    direction: bidirectional
    frequency: real_time

    penguinmails_to_hubspot:
      - on: email_sent
        action: create_timeline_event
        event_type: email_sent

      - on: email_opened
        action: update_property
        property: penguinmails_last_email_opened

      - on: email_clicked
        action: create_timeline_event
        event_type: email_clicked
        trigger_workflow: true  # Optional

      - on: high_engagement
        condition: lead_score >= 75
        action: trigger_workflow
        workflow_id: "sales_qualified_lead"

    hubspot_to_penguinmails:
      - on: contact_created
        action: import_contact

      - on: contact_updated
        action: update_contact

      - on: list_membership_changed
        action: update_segments
```

## API Integration

```typescript
class HubSpotService {
  async syncContact(contactId: string): Promise<void> {
    const contact = await db.contacts.findById(contactId);

    // Find or create HubSpot contact
    const hsContact = await this.findHubSpotContact(contact.email);

    const properties = {
      email: contact.email,
      firstname: contact.firstName,
      lastname: contact.lastName,
      company: contact.company,
      penguinmails_lead_score: contact.leadScore,
      penguinmails_total_opens: contact.totalOpens,
      penguinmails_total_clicks: contact.totalClicks,
    };

    if (hsContact) {
      await this.updateHubSpotContact(hsContact.vid, properties);
    } else {
      await this.createHubSpotContact(properties);
    }
  }

  async createTimelineEvent(
    email: string,
    eventType: string,
    campaignName: string
  ): Promise<void> {
    await this.hubspotClient.timeline.create({
      eventTypeId: `penguinmails_${eventType}`,
      email,
      timestamp: Date.now(),
      extraData: {
        campaign_name: campaignName,
        action: eventType,
      },
    });
  }

  async triggerWorkflow(
    contactId: string,
    workflowId: string
  ): Promise<void> {
    const hsContact = await this.findHubSpotContact(contactId);

    await this.hubspotClient.workflows.enroll(workflowId, {
      contactVid: hsContact.vid,
    });
  }
}
```

## Related Documentation

- [Salesforce Integration](./salesforce-integration.md)
- [Sync Logic](./sync-logic.md)
- [Technical Security](./technical-security.md)
