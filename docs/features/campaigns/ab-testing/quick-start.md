---
title: "A/B Testing Quick Start"
last_modified_date: "2025-12-15"
level: "1"
persona: "All Users"
---

# A/B Testing Quick Start

Create your first A/B test in 5 minutes.

## Step 1: Create Campaign with A/B Test

1. Navigate to **Campaigns → Create New Campaign**
2. Enable **"A/B Testing"** toggle
3. Choose test type:
   - Subject Line
   - Email Content
   - Sender Name
   - Send Time

## Step 2: Configure Variations

### Example: Subject Line Test

```text
Variant A (Control):
Subject: "Limited Time: 50% Off All Products"

Variant B:
Subject: "Flash Sale: Save 50% Today Only"

Variant C (optional):
Subject: "Don't Miss Out: Half Price Sale Ends Tonight"
```

### Test Parameters

- **Test sample size**: 30% (10% per variant + 10% holdout)
- **Winner deployment**: Remaining 70%
- **Test duration**: 4 hours
- **Win criteria**: Highest open rate

## Step 3: Review Test Plan

System displays your test configuration:

```text
Total Audience: 10,000 contacts

Test Phase:
  - Variant A: 1,000 contacts (10%)
  - Variant B: 1,000 contacts (10%)
  - Variant C: 1,000 contacts (10%)
  - Duration: 4 hours

Winner Deployment:
  - Winning variant: 7,000 contacts (70%)
  - Sent after test completion
```

## Step 4: Launch Test

1. Review variations and parameters
2. Click **"Launch A/B Test"**
3. Test emails sent to sample groups
4. Monitor real-time results

## Step 5: Automatic Winner Deployment

After 4 hours, the system automatically selects and deploys the winner:

```text
Test Results:
  Variant A: 22% open rate
  Variant B: 28% open rate ← WINNER (26% lift)
  Variant C: 24% open rate

Winner selected: Variant B
Deploying to remaining 7,000 contacts...
```

**Result:** 26% improvement in opens vs. original subject line.

## What to Test

### Subject Line Testing

- Test one element at a time (length, emoji, personalization)
- Try different tones (urgent vs. casual)
- Test questions vs. statements
- Experiment with personalization

**Examples:**

```text
Control: "New Features Released"
Test:    "{{firstName}}, Check Out What's New"

Control: "Weekly Newsletter - November 25"
Test:    "🎉 Your Weekly Insights Are Here"
```

### Sender Name Testing

```text
Control: "PenguinMails Team"
Test:    "Sarah from PenguinMails"
```

### Send Time Testing

```text
Variant A: 9:00 AM Tuesday
Variant B: 2:00 PM Tuesday
Variant C: 6:00 PM Tuesday
```

---

[← Back to A/B Testing](./README)
