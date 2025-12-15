---
title: "Email Provider Integration"
last_modified_date: "2025-12-15"
level: "3"
persona: "Developers"
---

# Email Provider Integration

GDPR-compliant email API integration for SendGrid and Postmark.

## SendGrid Integration

```javascript
// sendgrid-service.js
import sgMail from '@sendgrid/mail';

export class SendGridEmailService {
    constructor() {
        sgMail.setApiKey(process.env.SENDGRID_API_KEY);
    }

    async sendEmail(emailData) {
        // Verify consent before sending
        const consentStatus = await this.consentService.verifyConsent(emailData.to);
        if (!consentStatus.isValid) {
            throw new Error('Consent verification failed');
        }

        const msg = {
            to: emailData.to,
            from: emailData.from,
            subject: emailData.subject,
            html: emailData.content,
            customArgs: {
                consentVerified: 'true',
                consentTimestamp: consentStatus.consentTimestamp
            },
            trackingSettings: {
                clickTracking: { enable: consentStatus.isValid },
                openTracking: { enable: consentStatus.isValid }
            }
        };

        try {
            const result = await sgMail.send(msg);
            return result;
        } catch (error) {
            console.error('SendGrid error:', error);
            throw error;
        }
    }

    async updateSubscriptionPreferences(email, preferences) {
        return await fetch('/api/sendgrid/subscriptions', {
            method: 'PUT',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ email, preferences })
        });
    }
}
```

## Postmark Privacy Controls

```javascript
// postmark-service.js
export class PostmarkEmailService {
    async sendTransactionalEmail(emailData) {
        const consentVerified = await this.consentService.verifyConsent(emailData.To);

        if (!consentVerified.isValid) {
            throw new Error('Marketing consent required');
        }

        const emailData = {
            From: emailData.From,
            To: emailData.To,
            Subject: emailData.Subject,
            HtmlBody: emailData.HtmlBody,
            TrackOpens: consentVerified.isValid,
            TrackLinks: consentVerified.isValid
        };

        return await this.postmarkClient.sendEmail(emailData);
    }
}
```

---

[← Back to Technical Implementation](./README)
