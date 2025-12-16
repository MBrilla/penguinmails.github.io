---
title: "Advanced Onboarding Features"
description: "Interactive checklists, contextual tooltips, video tutorials, and milestones"
last_modified_date: "2025-11-19"
level: "2"
persona: "Product Managers, UX Designers"
---

# Advanced Onboarding Features

Interactive checklists, contextual tooltips, video tutorials, and progress milestones for PenguinMails onboarding.

## Interactive Checklist

Persistent checklist in sidebar:

```text
Getting Started ────────○○○ 70%

✓ Create workspace
✓ Add domain
✓ Verify domain
✓ Create email account
⏳ Import contacts (20/100)
○ Create first campaign
○ Send first email

[Collapse]
```

**Features:**
- Persists across sessions
- Click items to jump to relevant page
- Progress percentage updates in real-time
- Dismissible after completion

## Contextual Tooltips

**First-time visitor tooltips:**

```text
[i] Workspaces
    Organize your campaigns by client,
    brand, or project.

    [Got It] [Learn More]
```

**Smart triggering:**
- Show on first page visit only
- Don't show if user has interacted with feature
- Allow dismissal (never show again)
- Max 3 tooltips per page

## Interactive Tutorial Mode

**Activated by:** "Take the Product Tour" button

**Flow:**
1. Dim entire UI
2. Highlight specific element
3. Show explanation overlay
4. Guide through workflow
5. Allow skipping or pausing

**Example:**

```text
[Spotlight on "Create Campaign" button]

Step 1 of 5: Creating Your First Campaign

Click here to create a new email campaign.

[Next] [Skip Tour]
```

## Video Tutorials

**Embedded at key moments:**
- Domain setup: "How to add DNS records"
- Email warmup: "Why warmup matters"
- Campaign creation: "Your first campaign"

**Format:**
- 60-90 seconds each
- Skippable
- Closed captions
- Hosted on CDN for fast loading

## Progress Milestones

**Achievement notifications:**

```text
🎊 Milestone Unlocked: First Email Sent!

You've sent your first email through PenguinMails.
Keep going!

[View Campaign Analytics]
```

**Milestones:**
- ✓ First workspace created
- ✓ First domain verified
- ✓ First email account created
- ✓ First 10 contacts imported
- ✓ First campaign created
- ✓ First email sent
- ✓ First email opened
- ✓ First link clicked

## Personalized Recommendations

Based on user behavior:

```text
👋 We noticed you haven't set up email warmup yet.

Warmup helps build sender reputation and improves
deliverability. It only takes 2 minutes.

[Set Up Warmup Now] [Remind Me Later]
```

## Team Onboarding

For workspace invites:

```text
Welcome to [Workspace Name]!

You've been invited as a Workspace Member.
Here's what you can do:

✓ View campaigns
✓ Create and edit templates
○ Can't: Modify billing or infrastructure

[View Permissions] [Explore Dashboard]
```

## Related Documentation

- [Onboarding Overview](/docs/features/authentication/onboarding/overview) - Main page
- [User Flow](/docs/features/authentication/onboarding/user-flow) - Step-by-step guide
- [Technical Implementation](/docs/features/authentication/onboarding/implementation) - Database and services
