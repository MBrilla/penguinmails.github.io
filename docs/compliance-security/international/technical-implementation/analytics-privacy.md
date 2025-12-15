---
title: "Analytics & Privacy Controls"
last_modified_date: "2025-12-15"
level: "3"
persona: "Developers"
---

# Analytics & Privacy Controls

Consent-gated analytics and privacy-preserving reporting.

## Consent-Gated Analytics Dashboard

```javascript
// analytics-service.js
export class AnalyticsService {
    async getCampaignMetrics(campaignId, userConsents) {
        const consentStatuses = await this.getConsentStatuses(userConsents);
        const consentingUsers = consentStatuses.filter(status => status.isValid);

        // Only show metrics for users who provided consent
        if (consentingUsers.length === 0) {
            return {
                openRate: null,
                clickRate: null,
                reason: 'No consent provided for tracking'
            };
        }

        // Fetch anonymized metrics
        const metrics = await this.fetchAnonymizedMetrics(campaignId);

        return {
            openRate: metrics.openRate,
            clickRate: metrics.clickRate,
            totalRecipients: consentingUsers.length,
            consentingRecipients: consentingUsers.length,
            disclaimer: 'Metrics shown only for users who provided consent for tracking'
        };
    }

    async fetchAnonymizedMetrics(campaignId) {
        return await this.database.query(`
            SELECT
                COUNT(CASE WHEN opened = true THEN 1 END) as opens,
                COUNT(CASE WHEN clicked = true THEN 1 END) as clicks,
                COUNT(*) as total
            FROM email_engagement
            WHERE campaign_id = $1
            AND consent_verified = true
        `, [campaignId]);
    }
}
```

## Privacy-Preserving Reporting

```javascript
// privacy-reporting.js
export class PrivacyReporting {
    generateComplianceReport(dateRange) {
        return {
            consentMetrics: {
                totalContacts: this.getTotalContacts(dateRange),
                consentingContacts: this.getConsentingContacts(dateRange),
                consentRate: this.calculateConsentRate(dateRange),
                withdrawalsThisPeriod: this.getConsentWithdrawals(dateRange)
            },
            dataSubjectRights: {
                accessRequests: this.getAccessRequests(dateRange),
                rectificationRequests: this.getRectificationRequests(dateRange),
                erasureRequests: this.getErasureRequests(dateRange),
                averageResponseTime: this.calculateAverageResponseTime(dateRange)
            },
            complianceStatus: {
                gdprCompliant: this.assessGDPRCompliance(),
                eprivacyCompliant: this.assessEPrivacyCompliance(),
                lastAuditDate: this.getLastAuditDate(),
                nextAuditDue: this.getNextAuditDate()
            }
        };
    }
}
```

---

[← Back to Technical Implementation](./README)
