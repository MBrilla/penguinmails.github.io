---
title: "Postmark Integration"
description: "Premium transactional email delivery with Postmark ESP"
last_modified_date: "2025-01-15"
level: "2"
persona: "Operations Teams, Developers"
---

# Postmark Integration

Premium transactional email delivery with 99.96% deliverability rate.

## When to Use Postmark

Postmark excels for critical transactional emails:

- **Password resets** - Critical delivery required
- **Account notifications** - Must reach inbox
- **Order confirmations** - Transactional content
- **Welcome emails** - First impression matters

Postmark is NOT recommended for bulk marketing (use Mailgun or built-in SMTP instead).

## Setup Guide

### Step 1: Create Postmark Account

1. Sign up at [postmarkapp.com](https://postmarkapp.com)
2. Verify your email
3. Create a "Server" (sending environment)

### Step 2: Get API Key

1. Go to **Servers** → **API Tokens**
2. Copy **Server API Token**
3. Save for PenguinMails configuration

### Step 3: Configure in PenguinMails

Navigate to Settings → Integrations → ESP Integration:

```
ESP Provider: Postmark
API Token: [paste token]
Default Sender: noreply@yourdomain.com
Track Opens: ☑ Yes
Track Links: ☑ Yes

[Save Configuration]
```

### Step 4: Verify Domain

1. Postmark provides DNS records (DKIM, Return-Path)
2. Add records to your DNS
3. Wait for verification (~1 hour)
4. Start sending!

## Postmark Features

**Fast delivery** - Sub-second sending for time-critical emails.

**Detailed analytics** - Open, click, and bounce tracking with comprehensive reporting.

**Bounce categorization** - Distinguishes hard bounces from soft bounces for accurate list management.

**Spam analysis** - Explains why emails might be flagged by spam filters.

**Templates** - HTML templates with variable substitution for consistent branding.

**Webhooks** - Real-time delivery notifications for immediate status updates.

## Pricing

Postmark uses pay-as-you-go pricing:

- **$1.25 per 1,000 emails**
- First 100 emails/month free
- Volume discounts available for high-volume senders

## Best Use Cases

| Email Type | Recommendation |
|------------|----------------|
| Password reset | Highly recommended |
| Welcome email | Highly recommended |
| Order confirmation | Highly recommended |
| Account alerts | Highly recommended |
| Newsletter | Use Mailgun instead |
| Promotional | Use Mailgun instead |

## Troubleshooting

**Issue:** ESP rejects API call
**Solution:** Verify API key is correct and domain verification is complete.

**Issue:** Emails not delivering
**Solution:** Check DNS records, verify sender domain is properly authenticated.

**Issue:** Low open rates
**Solution:** Verify tracking is enabled, check subject lines for spam triggers.

---

**Related Documents:**
- [ESP Integration Overview](overview.md)
- [Mailgun Integration](mailgun.md)
- [Advanced Configuration](advanced-config.md)
