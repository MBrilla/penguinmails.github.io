---
title: "Consent Management System"
last_modified_date: "2025-12-15"
level: "3"
persona: "Developers"
---

# Consent Management System

Real-time validation API and consent verification service.

## Real-Time Validation API

```javascript
// React Hook Form integration for consent validation
import { useForm } from 'react-hook-form';
import { z } from 'zod';
import { consentService } from '@/services/consent';

const emailCampaignSchema = z.object({
    recipients: z.array(z.object({
        email: z.string().email(),
        consentVerified: z.boolean().refine(val => val === true, {
            message: "Valid consent required for all recipients"
        })
    }))
});

export function EmailCampaignForm() {
    const { register, handleSubmit, watch, formState: { errors } } = useForm({
        resolver: zodResolver(emailCampaignSchema)
    });

    const validateConsent = async (email) => {
        const consentStatus = await consentService.verifyConsent(email);
        return consentStatus.isValid;
    };

    return (
        <form onSubmit={handleSubmit(onSubmit)}>
            {/* Form implementation */}
        </form>
    );
}
```

## Consent Verification Service

```javascript
// consent-service.js
export class ConsentService {
    async verifyConsent(email) {
        try {
            const response = await fetch('/api/consent/verify', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ email })
            });

            const result = await response.json();

            return {
                isValid: result.consentValid,
                consentTimestamp: result.consentTimestamp,
                consentSource: result.consentSource,
                withdrawalDate: result.withdrawalDate
            };
        } catch (error) {
            console.error('Consent verification failed:', error);
            return { isValid: false };
        }
    }

    async recordConsent(email, consentData) {
        return await fetch('/api/consent/record', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ email, ...consentData })
        });
    }

    async withdrawConsent(email, reason = 'user_requested') {
        return await fetch('/api/consent/withdraw', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ email, reason })
        });
    }
}
```

---

[← Back to Technical Implementation](./README)
