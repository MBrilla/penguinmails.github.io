---
title: "Create Webhook Route"
description: "Route specification for the multi-step webhook creation form"
last_modified_date: "2025-01-20"
level: 3
persona: Developers
keywords: webhooks, create, configuration, events, security
---

# Create Webhook Route

**Route:** `/dashboard/settings/webhooks/create`

**Access:** Admin, Developer

**User Story:** *"As a developer, I want to create a webhook to receive real-time notifications, so I can integrate PenguinMails with my application."*

## Step 1: Basic Configuration

**Webhook Name:**
- Text input (e.g., "CRM Contact Sync")
- Help text: "Choose a descriptive name to identify this webhook."

**Description (Optional):**
- Textarea for additional notes about webhook purpose

**Endpoint URL:**
- URL input (e.g., `https://yourapp.com/webhooks/penguinmails`)
- Validation: Must be HTTPS (HTTP allowed for localhost testing)
- Help text: "Your server endpoint that will receive webhook events."

**Status:**
- Radio buttons:
  - ○ Active (Start receiving events immediately)
  - ○ Paused (Create but don't send events yet)

## Step 2: Event Selection

**Event Categories (Expandable Sections):**

**Email Events:**
- ☑ email.sent - Email queued for sending
- ☑ email.delivered - Email successfully delivered
- ☑ email.opened - Recipient opened email
- ☑ email.clicked - Recipient clicked link
- ☑ email.bounced - Email bounced (hard or soft)
- ☐ email.spam_reported - Marked as spam
- ☐ email.unsubscribed - Recipient unsubscribed

**Campaign Events:**
- ☐ campaign.launched - Campaign started sending
- ☐ campaign.completed - Campaign finished
- ☐ campaign.paused - Campaign paused by user

**Contact Events:**
- ☐ contact.created - New contact added
- ☐ contact.updated - Contact information updated
- ☐ contact.unsubscribed - Contact unsubscribed from all

**Quick Select Buttons:**
- "Select All": Check all events
- "Email Events Only": Select all email events
- "Clear All": Uncheck all events

**Event Preview:**
"You've selected 5 events. [View sample payloads →](/docs/design/routes/#event-samples)"

## Step 3: Event Filters (Optional)

**Advanced Filtering Section (Expandable):**

**Filter by Workspace:**
- Dropdown: Select specific workspaces (default: All)
- Multi-select: Choose multiple workspaces

**Filter by Campaign:**
- Dropdown: Select specific campaigns (default: All)
- Help text: "Only receive events from selected campaigns."

**Filter by Contact Tags:**
- Tag input: Enter tags (e.g., "vip", "premium")
- Help text: "Only receive events for contacts with these tags."

**Custom JSON Filters (Advanced):**
```json
{
  "contact.tags": ["vip", "premium"],
  "campaign.type": "promotional",
  "email.opened_count": { "$gte": 3 }
}
```

**"Test Filter" Button**: Validate filter syntax

## Step 4: Security & Retry Configuration

**Signature Verification:**
- Auto-generated secret: `whsec_abc123...` (shown after creation)
- Help text: "Use this secret to verify webhook signatures."

**Retry Configuration:**
- Enable Retries: Toggle (default: On)
- Max Retry Attempts: Number input (1-10, default: 5)
- Retry Delays: "1m, 5m, 15m, 1h, 24h" (exponential backoff)

**Timeout:**
- Response Timeout: Number input (1-30 seconds, default: 5)
- Help text: "Webhook must respond within this time."

**IP Whitelist (Optional):**
- Webhook Source IPs: Display PenguinMails IP ranges
- Copy Button: Copy IP list for firewall configuration

## Step 5: Test Before Saving

**Test Webhook Section:**
- "Send Test Event" Button: Sends sample event to endpoint
- Event Type: Dropdown (select which event to test)
- Use Sample Data: Checkbox (default: On)

**Test Result Display (Success):**
```text
Sending test event to https://yourapp.com/webhooks...

✓ Test successful!
  Status: 200 OK
  Response Time: 142ms
  Response Body: "OK"
```

**Test Result Display (Error):**
```text
✗ Test failed
  Status: 500 Internal Server Error
  Error: Connection timeout

[Retry Test] [Save Anyway]
```

**"Create Webhook" Button**: Saves webhook configuration

## Webhook Created Success

**Success Modal:**
- Title: "Webhook Created Successfully"
- Webhook ID: `wh_abc123`
- Secret Key: `whsec_def456...` (shown once)
- Warning: "Save this secret key. You won't be able to see it again."

**Copy Buttons:**
- Copy Webhook ID
- Copy Secret Key

**Next Steps:**
- [View Webhook Details →](/dashboard/settings/webhooks/wh_abc123)
- [View Code Examples →](/docs/design/routes/#code-examples)
- [Test Webhook →](/dashboard/settings/webhooks/wh_abc123/test)

## Technical Integration

- **Secret Generation**: Cryptographically secure random string
- **Signature**: HMAC-SHA256 with secret key
- **Storage**: Secret hashed before storing in database

## User Journey Context

One-time setup per integration, occasional updates for new event types.

## Related Documentation

- [Webhook Creation Guide](/docs/features/integrations/webhook-system#create-your-first-webhook)
- [Event Reference](/docs/features/integrations/webhook-system#event-reference)
