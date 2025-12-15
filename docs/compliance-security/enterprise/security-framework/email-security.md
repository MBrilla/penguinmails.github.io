---
title: "Email Security"
last_modified_date: "2025-12-15"
level: "2"
persona: "Documentation Users"
---

# Email Security

SPF, DKIM, DMARC configuration and email warm-up security practices.

## SPF, DKIM, DMARC Configuration

### DNS Records

```dns
# SPF Record
TXT @ "v=spf1 include:_spf.penguinmails.com ~all"

# DKIM Record
TXT mailu._domainkey.penguinmails.com "v=DKIM1; k=rsa; p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQE..."

# DMARC Record
TXT _dmarc.penguinmails.com "v=DMARC1; p=quarantine; rua=mailto:dmarc@penguinmails.com"
```

### Email Authentication

```javascript
// Email sending with authentication headers
const sendEmail = async (emailData) => {
  const mailOptions = {
    from: emailData.from,
    to: emailData.to,
    subject: emailData.subject,
    html: emailData.content,
    headers: {
      'DKIM-Signature': generateDKIMSignature(emailData),
      'Authentication-Results': 'spf=pass smtp.mailfrom=penguinmails.com'
    }
  };

  return await smtpTransporter.sendMail(mailOptions);
};
```

## Email Warm-up Security

### Reputation Management

```javascript
// Safe warm-up algorithm
const emailWarmup = {
  // Daily volume limits based on reputation
  calculateDailyLimit: (reputationScore, daysActive) => {
    const baseLimit = 10; // Start with 10 emails
    const maxLimit = Math.min(1000, daysActive * 50); // Scale up gradually

    const reputationMultiplier = reputationScore / 100;

    return Math.floor(baseLimit * reputationMultiplier + maxLimit * (1 - reputationMultiplier));
  },

  // Monitor bounce rates and adjust
  checkBounceRate: (sentCount, bouncedCount) => {
    const bounceRate = bouncedCount / sentCount;

    if (bounceRate > 0.1) { // 10% bounce rate
      return { action: 'pause', reason: 'High bounce rate' };
    } else if (bounceRate > 0.05) { // 5% bounce rate
      return { action: 'reduce_volume', reason: 'Moderate bounce rate' };
    }

    return { action: 'continue', reason: 'Healthy bounce rate' };
  }
};
```

---

[← Back to Security Framework](./README)
