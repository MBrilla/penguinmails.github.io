---
title: "Salesforce Integration"
description: "OAuth authentication, object mapping, sync rules, and API implementation for Salesforce"
last_modified_date: "2025-01-20"
level: 3
persona: Developers
keywords: Salesforce, OAuth, contacts, leads, activities, sync
---

# Salesforce Integration

Salesforce is the priority CRM integration for Q1 2026, providing bi-directional sync for contacts, leads, and activity tracking.

## OAuth Authentication Flow

```yaml
salesforce_oauth:
  authorization_endpoint: https://login.salesforce.com/services/oauth2/authorize
  token_endpoint: https://login.salesforce.com/services/oauth2/token
  scopes:
    - api  # Access to Salesforce APIs
    - refresh_token  # Offline access
    - full  # Full access to data
  flow:
    1. User clicks "Connect Salesforce"
    2. Redirect to Salesforce login
    3. User authorizes PenguinMails app
    4. Receive authorization code
    5. Exchange for access + refresh tokens
    6. Store encrypted tokens in database
```

## Object Mapping

### Contacts

```yaml
salesforce_contact:
  standard_fields:
    Email → email
    FirstName → first_name
    LastName → last_name
    Company → company
    Phone → phone
    Title → job_title
    MailingCity → city
    MailingState → state
    MailingCountry → country
    LeadSource → lead_source

  custom_fields:
    PenguinMails_Lead_Score__c → lead_score
    PenguinMails_Last_Email_Opened__c → last_email_opened
    PenguinMails_Total_Opens__c → total_opens
    PenguinMails_Total_Clicks__c → total_clicks
    PenguinMails_Unsubscribed__c → unsubscribed
```

### Leads

```yaml
salesforce_lead:
  standard_fields:
    Email → email
    FirstName → first_name
    LastName → last_name
    Company → company
    Phone → phone
    Title → job_title
    City → city
    State → state
    Country → country
    Status → lead_status
    Rating → lead_rating

  custom_fields:
    PenguinMails_Engagement_Score__c → engagement_score
    PenguinMails_Last_Campaign__c → last_campaign_id
```

### Activities (Tasks)

```yaml
salesforce_task:
  fields:
    Subject: "Email: {campaign_name}"
    Description: "Email {action} by {contact_name}"
    WhoId: contact_id
    ActivityDate: event_timestamp
    Status: "Completed"
    Type: "Email"

  activity_types:
    - Email Sent
    - Email Opened
    - Email Clicked
    - Email Replied
    - Email Bounced
```

## Sync Rules

```yaml
sync_rules:
  contact_sync:
    direction: bidirectional
    frequency: real_time  # via webhooks
    fallback: hourly_batch

    penguinmails_to_salesforce:
      - on: email_sent
        action: create_task

      - on: email_opened
        action: update_custom_field
        field: PenguinMails_Last_Email_Opened__c

      - on: email_clicked
        action: increment_field
        field: PenguinMails_Total_Clicks__c

      - on: unsubscribed
        action: update_field
        field: PenguinMails_Unsubscribed__c
        value: true

    salesforce_to_penguinmails:
      - on: contact_created
        action: import_contact
        add_to_segment: "Salesforce Contacts"

      - on: contact_updated
        action: update_contact
        conflict_resolution: salesforce_wins

      - on: contact_deleted
        action: soft_delete_contact
```

## API Integration

```typescript
class SalesforceService {
  async syncContact(contactId: string): Promise<void> {
    const contact = await db.contacts.findById(contactId);

    // Check if contact exists in Salesforce
    const sfContact = await this.findSalesforceContact(contact.email);

    if (sfContact) {
      // Update existing
      await this.updateSalesforceContact(sfContact.Id, {
        PenguinMails_Lead_Score__c: contact.leadScore,
        PenguinMails_Last_Email_Opened__c: contact.lastEmailOpened,
        PenguinMails_Total_Opens__c: contact.totalOpens,
        PenguinMails_Total_Clicks__c: contact.totalClicks,
      });
    } else {
      // Create new
      await this.createSalesforceContact({
        Email: contact.email,
        FirstName: contact.firstName,
        LastName: contact.lastName,
        Company: contact.company,
        PenguinMails_Lead_Score__c: contact.leadScore,
      });
    }
  }

  async logEmailActivity(
    contactId: string,
    activityType: string,
    campaignName: string
  ): Promise<void> {
    const sfContact = await this.findSalesforceContact(contactId);

    await this.createTask({
      WhoId: sfContact.Id,
      Subject: `Email: ${campaignName}`,
      Description: `Email ${activityType}`,
      ActivityDate: new Date().toISOString(),
      Status: 'Completed',
      Type: 'Email',
    });
  }
}
```

## Related Documentation

- [HubSpot Integration](./hubspot-integration.md)
- [Sync Logic](./sync-logic.md)
- [Technical Security](./technical-security.md)
