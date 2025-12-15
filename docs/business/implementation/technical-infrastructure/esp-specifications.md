---
title: "ESP Technical Specifications"
description: "SendGrid, Mailgun, Postmark, and Amazon SES API and architecture details"
last_modified_date: "2025-11-10"
level: "2"
persona: "Technical Teams, Integration Engineers"
---

# ESP Technical Specifications

Detailed API specifications, rate limits, and technical architecture for major Email Service Providers.

## SendGrid Technical Architecture

### API Technical Specifications

- **API Rate Limits**:
  - Web API: 1,000 requests/hour per API key
  - Web API v3: 100 requests/5 minutes per API key
  - Web API v2: 1,000 requests/hour per API key
  - Event Webhook: No rate limit (push notifications)

- **Email Sending Limits**:
  - Essentials/Pro: Up to 12x contact limit per month
  - Marketing Campaign: Up to 5x contact limit per send
  - Transactional: No monthly limits (rate limited)

- **Webhook Capabilities**:
  - Event Webhook: Real-time event notifications
  - Inbound Parse: Email-to-webhook integration
  - Delivery Webhook: Delivery status updates

### Authentication and Security

- **SPF/DKIM Support**: Automatic setup and management
- **DMARC**: Basic reporting and alignment checking
- **IP Whitelisting**: Enterprise feature for security
- **Two-Factor Authentication**: Available for account security
- **API Key Management**: Granular permission control

### Deliverability Technical Features

- **IP Warmup**: Automated warmup protocols
- **Reputation Monitoring**: Real-time sender score tracking
- **Blacklist Monitoring**: Proactive RBL checking
- **ISP Feedback Loops**: Direct feedback from major ISPs
- **Domain Authentication**: Automatic SPF/DKIM/DMARC setup

## Mailgun Technical Architecture

### API Technical Specifications

- **API Rate Limits**:
  - API calls: 2,000 requests/hour per user
  - Email sending: 2,000 requests/hour per user
  - Events API: 10,000 requests/hour per user

- **Email Sending Limits**:
  - Foundation: Up to 5x contact limit per send
  - Scale: Up to 12x contact limit per send
  - Transactional: No monthly limits

- **Integration Capabilities**:
  - RESTful API with comprehensive endpoints
  - SMTP relay with authentication
  - Webhooks for event notifications
  - Email parsing and forwarding

### Technical Features

- **Deliverability Tools**:
  - Advanced bounce classification
  - Spam complaint monitoring
  - Engagement tracking and analytics
  - IP reputation management

- **Automation and Workflows**:
  - Advanced campaign automation
  - Conditional branching logic
  - A/B testing with statistical significance
  - Time-based scheduling and optimization

### Cold Email Optimization

- **Dedicated IP Management**: Automatic IP rotation and warmup
- **List Hygiene**: Built-in bounce handling and validation
- **Compliance Tools**: CAN-SPAM and GDPR compliance features
- **Deliverability Monitoring**: Real-time inbox placement tracking

## Postmark Technical Architecture

### API Technical Specifications

- **API Rate Limits**:
  - 500 requests per minute per token
  - Burst allowance: 10 requests per second
  - Email sending: Rate limited by plan

- **Email Sending Limits**:
  - Pro: Up to 5x contact limit per send
  - Platform: Up to 8x contact limit per send
  - Ultra: Up to 10x contact limit per send
  - Transactional: No monthly limits

### Transactional Email Focus

- **Message Handling**:
  - Unlimited email volume for transactional
  - 10,000 messages per API call limit
  - Retry logic with exponential backoff
  - Delivery status tracking and reporting

- **Reliability Features**:
  - 99.99% uptime SLA
  - Multiple data center redundancy
  - Real-time failover capabilities
  - Comprehensive delivery analytics

### Security and Compliance

- **Data Protection**: SOC 2 Type II certified
- **Encryption**: TLS 1.2+ for data in transit
- **Compliance**: GDPR, CCPA, and CAN-SPAM compliant
- **Audit Logging**: Comprehensive activity logging

## Amazon SES Technical Architecture

### API Technical Specifications

- **Service Limits**:
  - Sending quota: 200 emails
  - 24-hour sending limit: 50,000 emails (default)
  - API requests: 14 requests/second

- **Scaling Capabilities**:
  - Auto-scaling sending quota
  - CloudWatch monitoring and alarms
  - SNS notifications for events
  - CloudWatch Logs integration

### Technical Integration

- **AWS Ecosystem**:
  - CloudWatch for monitoring and logging
  - SNS for event notifications
  - S3 for email archiving and logging
  - IAM for access control and permissions

- **Email Infrastructure**:
  - IPv4 and IPv6 support
  - Dedicated IP addresses available
  - SMTP and API sending options
  - Bounce and complaint handling

### Advanced Features

- **Identity Management**:
  - Easy domain verification process
  - Automatic SPF/DKIM/DMARC setup
  - Identity policy management
  - Cross-account sending capabilities

- **Monitoring and Analytics**:
  - Real-time sending metrics
  - Delivery and bounce statistics
  - Reputation monitoring
  - Reputation dashboard with graphs

## Related Documentation

- [Technical Infrastructure Overview](/docs/business/implementation/technical-infrastructure/overview) - Summary and navigation
- [VPS Provider Analysis](/docs/business/implementation/technical-infrastructure/vps-providers) - Self-hosted options
- [Performance Optimization](/docs/business/implementation/technical-infrastructure/performance-optimization) - Tuning and capacity
