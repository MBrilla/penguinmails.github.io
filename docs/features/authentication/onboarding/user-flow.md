---
title: "Onboarding User Flow"
description: "Step-by-step walkthrough of the new user onboarding journey"
last_modified_date: "2025-11-19"
level: "2"
persona: "Product Managers, UX Designers"
---

# Onboarding User Flow

Detailed step-by-step walkthrough of the new user onboarding journey from signup to first campaign.

## Step 1: Sign Up (1 min)

**Fields:**
- Email address
- Password
- Company name
- User name

**Welcome Screen:**

```text
Welcome to PenguinMails!

Let's get your professional email infrastructure
set up in under 10 minutes.

[Start Setup] [Skip Tour]
```

## Step 2: Create Workspace (1 min)

```text
Create Your First Workspace

Workspaces help you organize campaigns by client,
project, or brand.

Workspace Name: [_______________]
Example: "Main Company" or "Client: Acme Corp"

[Continue]
```

**Tooltip:** "You can create unlimited workspaces. Start with one for now."

## Step 3: Add Domain (2 min)

```text
Step 3 of 6: Add Your Sending Domain

This is the domain you'll send emails from
(e.g., yourdomain.com)

Domain: [_______________]

☐ I have access to DNS settings for this domain

[Continue] [Need help?]
```

**If "Need help?" clicked:**
- Show video: "What is a sending domain?"
- Link to: Domain setup guide

## Step 4: Verify Domain (2 min)

```text
Step 4 of 6: Verify Domain Ownership

Add this DNS record to verify you own this domain:

TXT Record:
  Host: _penguinmails-verify
  Value: abc123xyz...

[Copy Record] [I've Added the Record]

OR

[Auto-Configure with Cloudflare] [Auto-Configure with Route53]
```

**Auto-detection:**
- System polls for DNS record every 10 seconds
- Shows "Verifying..." animation
- Auto-advances when verified

## Step 5: Set Up Payment (1 min)

```text
Step 5 of 6: Choose Your Plan

○ Starter - $49/mo
  Up to 5,000 emails/month
  1 workspace

● Professional - $99/mo  [RECOMMENDED]
  Up to 25,000 emails/month
  Unlimited workspaces
  Priority support

○ Business - $249/mo
  Up to 100,000 emails/month
  Everything in Professional
  Dedicated infrastructure

[Start 14-Day Free Trial] No credit card required
```

**After trial started:**

```text
✓ Trial Started - 14 days remaining

You won't be charged until your trial ends.
Add payment method anytime in Settings.

[Continue to Infrastructure Setup]
```

## Step 6: Provision Infrastructure (2-3 min)

```text
Step 6 of 6: Launch Your Email Infrastructure

We're setting up your professional email infrastructure:

✓ VPS server provisioning
⏳ Installing SMTP server (2 min remaining)
○ Configuring DNS records
○ Installing SSL certificates
○ Creating your first email account

[Watch Setup Video] while you wait
```

**Progress bar animates** showing real-time status.

**Complete:**

```text
🎉 Infrastructure Ready!

Your email infrastructure is live at:
mail.yourdomain.com

Next: Create your first email account

[Create Email Account]
```

## Step 7: Create First Email Account (1 min)

```text
Create Your First Sending Account

Email Address: [___]@yourdomain.com
Password: [Generate Secure Password]

[Create Account & Finish Setup]
```

## Step 8: Success & Next Steps (30 sec)

```text
🎉 Congratulations! You're All Set

Your email infrastructure is ready to send.

Quick Actions:
☐ Create your first campaign
☐ Import your contact list
☐ Set up email warmup (recommended)

[Go to Dashboard] [Take the Product Tour]
```

## Related Documentation

- [Onboarding Overview](/docs/features/authentication/onboarding/overview) - Main page
- [Advanced Features](/docs/features/authentication/onboarding/advanced-features) - Checklists and milestones
- [Technical Implementation](/docs/features/authentication/onboarding/implementation) - Database and services
