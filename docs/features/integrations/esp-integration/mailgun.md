---
title: "Mailgun Integration"
description: "Powerful bulk email delivery for marketing campaigns"
last_modified_date: "2025-01-15"
level: "2"
persona: "Operations Teams, Developers"
---

# Mailgun Integration

Powerful bulk email delivery for marketing campaigns and high-volume sending.

## When to Use Mailgun

Mailgun excels for marketing and bulk email:

- **Marketing campaigns** - Newsletters, promotions
- **High volume** - Thousands of emails per campaign
- **Advanced segmentation** - A/B testing, targeting
- **Email validation** - List cleaning and verification

Mailgun is NOT recommended for transactional emails (use Postmark instead).

## Setup Guide

### Step 1: Create Mailgun Account

1. Sign up at [mailgun.com](https://www.mailgun.com)
2. Choose US or EU region data center
3. Verify email and billing info

### Step 2: Add Sending Domain

1. Go to **Sending** → **Domains**
2. Click **Add New Domain**
3. Enter your domain (e.g., `mail.yourdomain.com`)
4. Add provided DNS records (SPF, DKIM, CNAME)
5. Wait for verification

### Step 3: Get API Credentials

1. Go to **Settings** → **API Keys**
2. Copy **Private API Key**
3. Note your **Domain Name**

### Step 4: Configure in PenguinMails

Navigate to Settings → Integrations → ESP Integration:

```
ESP Provider: Mailgun
API Key: [paste private key]
Domain: mail.yourdomain.com
Region: US / EU
Track Opens: ☑ Yes
Track Clicks: ☑ Yes
Unsubscribe Handling: ☑ Automatic

[Save Configuration]
```

## Mailgun Features

**Powerful API** - Full programmatic control over email sending.

**Email validation** - Pre-send list verification to reduce bounces.

**A/B testing** - Test subject lines, content, and send times.

**Scheduled sending** - Time-zone optimized delivery for global audiences.

**Tagging & segmentation** - Organize campaigns for detailed analytics.

**Route filtering** - Advanced email routing logic for complex workflows.

## Pricing

Mailgun offers tiered pricing:

- **Flex Plan:** $35/month base + $1/1,000 emails
- **Foundation:** $55/month for 50k emails
- **Volume discounts** for >100k emails/month

## Best Use Cases

| Email Type | Recommendation |
|------------|----------------|
| Weekly newsletter | Highly recommended |
| Product announcements | Highly recommended |
| Promotional campaigns | Highly recommended |
| A/B testing | Highly recommended |
| Password reset | Use Postmark instead |
| Order confirmation | Use Postmark instead |

## Troubleshooting

**Issue:** ESP rejects API call
**Solution:** Verify API key, check domain verification status in Mailgun dashboard.

**Issue:** Emails not delivering
**Solution:** Check DNS records (SPF, DKIM, CNAME), verify domain in Mailgun.

**Issue:** High bounce rate
**Solution:** Use Mailgun's email validation to clean your list before sending.

**Issue:** Low open rates
**Solution:** Improve subject lines, use A/B testing to optimize engagement.

---

**Related Documents:**
- [ESP Integration Overview](overview.md)
- [Postmark Integration](postmark.md)
- [Advanced Configuration](advanced-config.md)
