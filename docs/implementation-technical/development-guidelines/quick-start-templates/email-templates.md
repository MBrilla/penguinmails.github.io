---
title: "Email Templates"
last_modified_date: "2025-12-15"
level: "2"
persona: "Developers, Designers"
---

# Email Templates

Responsive email CSS styles and HTML layouts for PenguinMails campaigns.

## Responsive Email Template Styles

```css
/* PenguinMails Email Template Styles */
.email-container {
  max-width: 600px;
  margin: 0 auto;
  font-family: 'Helvetica Neue', Arial, sans-serif;
  background-color: #f8f9fa;
}

.email-header {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  padding: 40px 30px;
  text-align: center;
}

.email-header h1 {
  margin: 0;
  font-size: 28px;
  font-weight: bold;
}

.email-content {
  background: #ffffff;
  padding: 40px 30px;
  line-height: 1.6;
  color: #333333;
}

.email-content p {
  margin-bottom: 20px;
  font-size: 16px;
}

.cta-button {
  display: inline-block;
  background: linear-gradient(135deg, #4CAF50 0%, #45a049 100%);
  color: white !important;
  padding: 15px 30px;
  text-decoration: none;
  border-radius: 8px;
  font-weight: bold;
  font-size: 16px;
  margin: 25px 0;
}

.email-footer {
  background: #f8f9fa;
  padding: 30px;
  text-align: center;
  font-size: 14px;
  color: #666666;
  border-top: 1px solid #dee2e6;
}

/* Mobile Responsive Design */
@media only screen and (max-width: 600px) {
  .email-container {
    width: 100% !important;
    margin: 0 !important;
  }

  .email-header {
    padding: 30px 20px !important;
  }

  .email-content {
    padding: 30px 20px !important;
  }

  .cta-button {
    width: 100% !important;
    text-align: center !important;
  }
}

/* Dark mode support */
@media (prefers-color-scheme: dark) {
  .email-container {
    background-color: #1a1a1a;
  }

  .email-content {
    background: #2d2d2d;
    color: #e0e0e0;
  }

  .email-footer {
    background-color: #1a1a1a;
    color: #999999;
  }
}
```

## HTML Email Template

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{{subject}}</title>
</head>
<body style="margin: 0; padding: 0; background-color: #f8f9fa;">
  <div class="email-container">
    <!-- Header -->
    <div class="email-header">
      <h1>{{company_name}}</h1>
    </div>
    
    <!-- Content -->
    <div class="email-content">
      <p>Hi {{firstName}},</p>
      <p>{{message_body}}</p>
      
      <a href="{{cta_url}}" class="cta-button">
        {{cta_text}}
      </a>
      
      <p>Best regards,<br>The {{company_name}} Team</p>
    </div>
    
    <!-- Footer -->
    <div class="email-footer">
      <p>{{company_address}}</p>
      <p>
        <a href="{{unsubscribe_url}}">Unsubscribe</a> | 
        <a href="{{preferences_url}}">Email Preferences</a>
      </p>
    </div>
  </div>
</body>
</html>
```

## Template Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `{{firstName}}` | Recipient first name | John |
| `{{lastName}}` | Recipient last name | Smith |
| `{{email}}` | Recipient email | john@example.com |
| `{{company_name}}` | Your company name | Acme Inc |
| `{{subject}}` | Email subject line | Welcome! |
| `{{unsubscribe_url}}` | Unsubscribe link | Auto-generated |

---

[← Back to Quick Start Templates](./README)
