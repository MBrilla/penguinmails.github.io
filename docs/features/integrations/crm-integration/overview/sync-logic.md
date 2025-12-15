---
title: "Sync Logic & Conflict Resolution"
description: "Bi-directional sync patterns, field mapping configuration, and conflict resolution strategies"
last_modified_date: "2025-01-20"
level: 3
persona: Developers
keywords: sync, bi-directional, field mapping, conflict resolution
---

# Sync Logic & Conflict Resolution

This document covers the bi-directional synchronization logic between PenguinMails and CRM platforms, including field mapping configuration and conflict resolution strategies.

## Field Mapping Configuration

```yaml
field_mapping:
  mapping_types:
    - direct  # 1:1 field mapping
    - formula  # Calculated fields
    - conditional  # Based on conditions
    - aggregation  # Sum, count, etc.

  examples:
    direct_mapping:
      crm_field: Email
      penguinmails_field: email
      direction: bidirectional

    formula_mapping:
      crm_field: Full_Name__c
      formula: "{{first_name}} {{last_name}}"
      direction: to_crm

    conditional_mapping:
      crm_field: Lead_Status__c
      condition: "lead_score >= 75"
      value_if_true: "Hot"
      value_if_false: "Warm"
      direction: to_crm

    aggregation_mapping:
      crm_field: Total_Email_Engagement__c
      formula: "SUM(opens + clicks)"
      direction: to_crm
```

## Bi-Directional Sync Logic

```typescript
interface SyncRule {
  direction: 'to_crm' | 'from_crm' | 'bidirectional';
  frequency: 'real_time' | 'hourly' | 'daily';
  conflictResolution: 'crm_wins' | 'penguinmails_wins' | 'newest_wins' | 'manual';
  filters?: any;
}

class CRMSyncService {
  async syncBidirectional(
    integration: Integration,
    syncRule: SyncRule
  ): Promise<void> {
    // Sync from CRM to PenguinMails
    if (syncRule.direction === 'from_crm' || syncRule.direction === 'bidirectional') {
      await this.syncFromCRM(integration);
    }

    // Sync from PenguinMails to CRM
    if (syncRule.direction === 'to_crm' || syncRule.direction === 'bidirectional') {
      await this.syncToCRM(integration);
    }
  }

  private async syncFromCRM(integration: Integration): Promise<void> {
    const lastSyncTime = integration.lastSyncAt;

    // Get updated contacts from CRM
    const updatedContacts = await this.getCRMUpdates(
      integration,
      lastSyncTime
    );

    for (const crmContact of updatedContacts) {
      const localContact = await db.contacts.findByEmail(crmContact.email);

      if (localContact) {
        // Handle conflict
        await this.handleConflict(
          localContact,
          crmContact,
          integration.conflictResolution
        );
      } else {
        // Create new contact
        await db.contacts.create({
          ...this.mapCRMFields(crmContact, integration.fieldMapping),
          source: 'crm_sync',
          crmId: crmContact.id,
        });
      }
    }
  }

  private async syncToCRM(integration: Integration): Promise<void> {
    const lastSyncTime = integration.lastSyncAt;

    // Get updated contacts from PenguinMails
    const updatedContacts = await db.contacts.findAll({
      where: {
        updatedAt: { [Op.gt]: lastSyncTime },
        tenantId: integration.tenantId,
      },
    });

    for (const contact of updatedContacts) {
      if (contact.crmId) {
        // Update existing CRM contact
        await this.updateCRMContact(
          integration,
          contact.crmId,
          this.mapLocalFields(contact, integration.fieldMapping)
        );
      } else {
        // Create new CRM contact
        const crmId = await this.createCRMContact(
          integration,
          this.mapLocalFields(contact, integration.fieldMapping)
        );

        await db.contacts.update(contact.id, { crmId });
      }
    }
  }
}
```

## Conflict Resolution Strategies

```yaml
conflict_resolution_strategies:
  crm_wins:
    description: "CRM data always overwrites PenguinMails data"
    use_case: "CRM is source of truth for contact info"

  penguinmails_wins:
    description: "PenguinMails data always overwrites CRM data"
    use_case: "PenguinMails is source of truth for engagement"

  newest_wins:
    description: "Most recently updated record wins"
    use_case: "Balanced approach, respects latest changes"

  manual:
    description: "Flag conflicts for manual resolution"
    use_case: "High-value data requiring human review"

  field_level:
    description: "Different strategies per field"
    example:
      email: crm_wins
      lead_score: penguinmails_wins
      company: newest_wins
      custom_fields: manual
```

## Conflict Resolution Implementation

```typescript
class ConflictResolver {
  async handleConflict(
    localContact: Contact,
    crmContact: any,
    strategy: string
  ): Promise<void> {
    switch (strategy) {
      case 'crm_wins':
        await this.applyCRMData(localContact, crmContact);
        break;

      case 'penguinmails_wins':
        await this.applyCRMData(crmContact, localContact);
        break;

      case 'newest_wins':
        if (crmContact.updatedAt > localContact.updatedAt) {
          await this.applyCRMData(localContact, crmContact);
        } else {
          await this.applyCRMData(crmContact, localContact);
        }
        break;

      case 'manual':
        await this.flagForManualReview(localContact, crmContact);
        break;

      case 'field_level':
        await this.applyFieldLevelResolution(localContact, crmContact);
        break;
    }
  }

  private async applyFieldLevelResolution(
    localContact: Contact,
    crmContact: any
  ): Promise<void> {
    const fieldRules = {
      email: 'crm_wins',
      firstName: 'crm_wins',
      lastName: 'crm_wins',
      company: 'newest_wins',
      leadScore: 'penguinmails_wins',
      totalOpens: 'penguinmails_wins',
      totalClicks: 'penguinmails_wins',
    };

    const updates: any = {};

    for (const [field, strategy] of Object.entries(fieldRules)) {
      if (strategy === 'crm_wins') {
        updates[field] = crmContact[field];
      } else if (strategy === 'newest_wins') {
        updates[field] = crmContact.updatedAt > localContact.updatedAt
          ? crmContact[field]
          : localContact[field];
      }
      // penguinmails_wins: keep local value (no update)
    }

    await db.contacts.update(localContact.id, updates);
  }
}
```

## Related Documentation

- [Salesforce Integration](./salesforce-integration.md)
- [HubSpot Integration](./hubspot-integration.md)
- [Technical Security](./technical-security.md)
