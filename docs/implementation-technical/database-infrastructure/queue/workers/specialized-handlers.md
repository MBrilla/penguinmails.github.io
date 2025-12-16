---
title: "Specialized Job Handlers"
description: "Email processing, sending, warmup, and bounce handling implementations"
last_modified_date: "2025-01-15"
level: "2"
persona: "Backend Engineers"
---

# Specialized Job Handlers

Implementations for processing different job types including email ingestion, sending, warmup, and bounce handling.

## Email Processing

```pseudo
async function processIncomingEmail(payload) {
  const { tenant_id, email_account_id, direction, type, from, to, subject, body, headers, messageId } = payload

  try {
    // Generate unique storage key
    const storageKey = `email_${messageId}_${Date.now()}`

    // Store email content
    const contentObject = await this.database.emailContent.create({
      data: {
        storage_key: storageKey,
        tenant_id: tenant_id,
        content_text: body.plain || body.html,
        content_html: body.html || convertToHtml(body.plain),
        headers: headers,
        raw_size_bytes: calculateSize(body),
        created: new Date(),
        retention_days: 2555 // 7 years
      }
    })

    // Create message reference
    const messageRef = await this.database.emailMessages.create({
      data: {
        storage_key: storageKey,
        tenant_id: tenant_id,
        email_account_id: email_account_id,
        direction: direction,
        message_type: type || 'email',
        from_email: from,
        to_email: Array.isArray(to) ? to.join(',') : to,
        subject: subject,
        status: 'received',
        processed: new Date()
      }
    })

    // Process attachments if present
    if (payload.attachments && payload.attachments.length > 0) {
      await this.processAttachments(payload.attachments, storageKey)
    }

    return {
      success: true,
      storageKey,
      contentObjectId: contentObject.storageKey,
      messageRefId: messageRef.storageKey
    }

  } catch (error) {
    throw new Error(`Email processing failed: ${error.message}`)
  }
}
```

## Email Sending

```pseudo
async function processEmailSending(payload) {
  const { campaign_id, recipient_id, email_template, personalizations } = payload

  try {
    // Get campaign and template data
    const campaign = await this.database.campaigns.findUnique({
      where: { id: campaign_id }
    })

    const template = await this.database.emailTemplates.findUnique({
      where: { id: email_template }
    })

    // Personalize email content
    const personalizedContent = await this.personalizeEmail(
      template.content,
      personalizations
    )

    // Send email via email service
    const sendResult = await this.emailService.sendEmail({
      to: personalizations.email,
      subject: this.interpolate(template.subject, personalizations),
      html: personalizedContent.html,
      text: personalizedContent.text,
      campaign_id: campaign_id,
      tenant_id: campaign.tenant_id
    })

    // Update delivery status
    await this.database.emailDeliveries.create({
      data: {
        campaign_id: campaign_id,
        recipient_id: recipient_id,
        message_id: sendResult.messageId,
        status: 'sent',
        sent_at: new Date(),
        provider_response: sendResult.response
      }
    })

    return {
      success: true,
      messageId: sendResult.messageId,
      deliveryId: sendResult.deliveryId
    }

  } catch (error) {
    throw new Error(`Email sending failed: ${error.message}`)
  }
}
```

## Warmup Processing

```pseudo
async function processWarmupJob(payload) {
  const { email_account_id, warmup_type, sequence_step } = payload

  try {
    // Get warmup configuration
    const warmupConfig = await this.database.warmupConfigs.findUnique({
      where: { email_account_id }
    })

    // Execute warmup action based on type and step
    switch (warmup_type) {
      case 'inbound':
        return await this.executeInboundWarmup(email_account_id, sequence_step)

      case 'outbound':
        return await this.executeOutboundWarmup(email_account_id, sequence_step)

      case 'engagement':
        return await this.executeEngagementWarmup(email_account_id, sequence_step)

      default:
        throw new Error(`Unknown warmup type: ${warmup_type}`)
    }

  } catch (error) {
    throw new Error(`Warmup processing failed: ${error.message}`)
  }
}
```

## Bounce Processing

```pseudo
async function processBounceJob(payload) {
  const { message_id, bounce_type, bounce_reason, recipient_email } = payload

  try {
    // Get original delivery record
    const delivery = await this.database.emailDeliveries.findUnique({
      where: { message_id },
      include: { campaign: true }
    })

    if (!delivery) {
      throw new Error(`Delivery record not found for message: ${message_id}`)
    }

    // Update delivery status
    await this.database.emailDeliveries.update({
      where: { id: delivery.id },
      data: {
        status: 'bounced',
        bounced_at: new Date(),
        bounce_type: bounce_type,
        bounce_reason: bounce_reason
      }
    })

    // Update campaign metrics
    await this.updateCampaignBounceMetrics(delivery.campaign_id, bounce_type)

    // Handle bounce based on type
    if (bounce_type === 'hard') {
      await this.handleHardBounce(delivery, recipient_email, bounce_reason)
    } else {
      await this.handleSoftBounce(delivery, recipient_email, bounce_reason)
    }

    return {
      success: true,
      deliveryId: delivery.id,
      bounceType: bounce_type
    }

  } catch (error) {
    throw new Error(`Bounce processing failed: ${error.message}`)
  }
}
```

---

**Related Documents:**
- [Overview](overview.md)
- [Job Processing](job-processing.md)
- [Operations & Scaling](operations.md)
