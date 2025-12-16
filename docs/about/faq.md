---
title: "Frequently Asked Questions"
description: "Common questions about cold email deliverability, PenguinMails, and migration"
last_modified_date: "2025-11-19"
level: "1"
persona: "Documentation Users"
---

# Frequently Asked Questions

Common questions about cold email deliverability, PenguinMails, and migration from other platforms.

## Why do my legitimate sales emails land in spam?

Email infrastructure causes most spam folder placement, not email content. Receiving servers evaluate your sender reputation, DNS configuration, warmup history, and sending patterns before they even read your message.

Misconfigured DNS records prevent servers from verifying your identity. New domains without warmup history trigger automatic suspicion. Mixed email types from one domain create inconsistent engagement patterns. Volume spikes from zero to one hundred emails per day match spam bot behavior.

PenguinMails addresses infrastructure issues automatically. DNS configuration happens correctly during initial setup. Warmup runs for 21 days before cold sending begins. Dedicated infrastructure keeps cold email separate from other types. Volume scales gradually based on your domain age and engagement rates.

## How is PenguinMails different from Mailchimp for cold email?

Mailchimp shuts down accounts for sending cold email because their terms of service prohibit it. PenguinMails is purpose-built for cold email and will never shut you down for using the platform as designed.

> "Mailchimp can't be used to do cold emails. It's against their policy."

Mailchimp optimizes for newsletter subscribers who opted in, transactional emails like receipts, and marketing to existing customers. Cold outreach to prospects violates their terms. They monitor content and terminate accounts doing outbound prospecting without warning or appeal.

PenguinMails provides infrastructure control that Mailchimp doesn't offer. You get VPS servers, dedicated IP addresses, and full DNS management. Compliance automation for GDPR and CAN-SPAM is built into the platform, not bolted on as an afterthought.

## What happens if my account gets shut down?

Your account won't get shut down for cold email because that's what PenguinMails is built for. Other platforms prohibit cold email in their terms of service and shut down accounts without warning.

> "They responded by claiming I had violated their regulations - without specifying how - and asked me to find another provider."

Mailchimp, SendGrid, and Postmark explicitly prohibit cold email in their legal terms. They monitor account activity and terminate users doing outbound prospecting. The termination process offers no warning period, no explanation of specific violations, and no appeal process.

PenguinMails operates differently because of how infrastructure is provisioned. Each customer gets isolated infrastructure where deliverability results from your own practices, not shared IP pools. If spam complaints or bounces spike, we alert you and provide tools to diagnose the problem.

## Do I need to warm up my email account before sending cold emails?

Yes, warmup is required for all new domains and inboxes. Skipping warmup leads to instant spam folder placement even when DNS records are configured correctly.

> "Warmup actually works - sounds like BS but letting inboxes send/receive normal emails for 2-3 weeks before cold emailing makes a huge difference."

New domains and inboxes have zero sender reputation with receiving servers. Gmail and Outlook treat unknown senders as suspicious by default.

**Warmup timelines:**
- Brand new domains: 21 days minimum
- Domains aged six months or more: 14 days
- High-volume senders: Ongoing warmup indefinitely

PenguinMails automates 21-day warmup with real engagement patterns. Many warmup tools use obviously automated messages that Gmail now detects and penalizes.

> "Skip the warm up tools, they get flagged fast and hurt inboxing."

Our warmup mimics genuine human conversations instead of sending template phrases seen millions of times.

## Can I migrate from Instantly/Smartlead/Lemlist without disrupting active campaigns?

Yes, migration happens with zero downtime through a parallel running approach. Your current tool continues sending while PenguinMails infrastructure warms up, then campaigns switch over gradually.

**Migration process (2-3 weeks):**

1. Export your campaign data from the existing tool including contacts, sequences, and templates
2. Import to PenguinMails where we map your campaign structure to our format
3. Run both platforms in parallel while warming PenguinMails infrastructure
4. Switch campaigns one at a time to verify delivery
5. Deactivate the old tool only after confirming all campaigns work correctly

Contact lists, campaign sequences, unsubscribe lists, and personalization variables all transfer successfully. Historical send metrics and domain reputation cannot transfer because they're tied to your previous infrastructure.

---

## Get Started

- **30-Day Free Trial** - Full access to all features
- **Dedicated Onboarding** - Personal setup assistance
- **Expert Support** - Real humans who understand email deliverability
- **Proven Results** - 95%+ deliverability guaranteed

**[Start Your Free Trial](https://app.penguinmails.com)** | **[Schedule a Demo](https://calendly.com/penguinmails)**

## Related Documentation

- [PenguinMails Overview](/docs/about/overview) - Main platform page
- [Problems & Solutions](/docs/about/problems-solutions) - Why emails fail
- [Target Customers](/docs/about/target-customers) - Who we serve
- [Differentiators](/docs/about/differentiators) - What makes us different
