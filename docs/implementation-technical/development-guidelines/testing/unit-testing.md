---
title: "Unit Testing Standards"
description: "Service layer testing with mocked dependencies"
last_modified_date: "2025-12-15"
level: "3"
persona: "Backend Engineers"
keywords: "unit testing, Vitest, mocks, service layer, validation"
---

# Unit Testing Standards

Unit tests validate individual components in isolation using mocked dependencies. These tests run quickly and provide immediate feedback during development.

## Test Structure

Unit tests follow the Arrange-Act-Assert pattern for clarity. Mock external dependencies to isolate the unit under test.

```typescript
// tests/unit/test-email-service.ts
import { describe, it, expect, beforeEach, afterEach, vi } from 'vitest';
import { EmailService } from '../../app/services/email-service';
import { EmailStatus } from '../../app/models/email';
import { EmailDeliveryError, ValidationError } from '../../app/exceptions';

vi.mock('../../app/services/smtp-client');
vi.mock('../../app/services/template-service');
vi.mock('../../app/services/analytics-service');

describe('EmailService', () => {
  let emailService: EmailService;
  let mockSmtpClient: ReturnType<typeof vi.fn>;
  let mockAnalyticsService: ReturnType<typeof vi.fn>;

  beforeEach(() => {
    mockSmtpClient = vi.fn();
    mockAnalyticsService = vi.fn();

    emailService = new EmailService(
      mockSmtpClient as any,
      mockAnalyticsService as any
    );
  });

  afterEach(() => {
    vi.clearAllMocks();
  });
});
```

## Email Service Tests

### Successful Email Send

```typescript
it('should send email successfully', async () => {
  const emailData: EmailData = {
    to: 'test@example.com',
    subject: 'Test Subject',
    content: { html: '<p>Test content</p>', text: 'Test content' },
    from: { name: 'Test Sender', email: 'sender@example.com' }
  };

  const expectedMessageId = 'msg_123456';
  mockSmtpClient.sendEmail = vi.fn().mockResolvedValue({
    messageId: expectedMessageId,
    status: 'sent'
  });

  const result = await emailService.sendEmail(emailData);

  expect(result.messageId).toBe(expectedMessageId);
  expect(result.status).toBe(EmailStatus.SENT);
  expect(result.sentAt).toBeDefined();

  expect(mockSmtpClient.sendEmail).toHaveBeenCalledTimes(1);
  expect(mockAnalyticsService.trackDelivery).toHaveBeenCalledTimes(1);
});
```

### Validation Error Handling

```typescript
it('should throw ValidationError for invalid recipient email', async () => {
  const emailData: EmailData = {
    to: 'invalid-email',
    subject: 'Test Subject',
    content: { html: '<p>Test</p>', text: 'Test' }
  };

  await expect(emailService.sendEmail(emailData)).rejects.toThrow(ValidationError);

  expect(mockSmtpClient.sendEmail).not.toHaveBeenCalled();
  expect(mockAnalyticsService.trackDelivery).not.toHaveBeenCalled();
});
```

### Bulk Email Processing

```typescript
it('should process bulk emails with proper batch processing', async () => {
  const recipients = Array.from({ length: 250 }, (_, i) => ({
    email: `user${i}@example.com`,
    name: `User ${i}`
  }));

  const emailData: EmailData = {
    subject: 'Bulk Test Email',
    content: { html: '<p>Bulk content</p>', text: 'Bulk content' }
  };

  const batchResults = Array.from({ length: 250 }, (_, i) => ({
    messageId: `msg_${i}`,
    status: 'sent'
  }));

  mockSmtpClient.sendEmailBatch = vi.fn().mockResolvedValue(batchResults);

  const results = await emailService.sendBulkEmails(recipients, emailData);

  expect(results).toHaveLength(250);
  expect(results.every(result => result.status === EmailStatus.SENT)).toBe(true);

  const expectedBatches = Math.ceil(250 / 100);
  expect(mockSmtpClient.sendEmailBatch).toHaveBeenCalledTimes(expectedBatches);
});
```

### Delivery Failure Handling

```typescript
it('should handle email delivery failures gracefully', async () => {
  const emailData: EmailData = {
    to: 'test@example.com',
    subject: 'Test Subject',
    content: { html: '<p>Test content</p>', text: 'Test content' }
  };

  mockSmtpClient.sendEmail = vi.fn().mockRejectedValue(
    new Error('SMTP connection failed')
  );

  await expect(emailService.sendEmail(emailData)).rejects.toThrow(EmailDeliveryError);

  expect(mockAnalyticsService.trackFailure).toHaveBeenCalledTimes(1);
});
```

### Content Validation

```typescript
it('should validate email content before sending', async () => {
  const emailData = {
    to: 'test@example.com',
    subject: '',
    content: { html: '', text: '' }
  };

  await expect(emailService.sendEmail(emailData)).rejects.toThrow(ValidationError);

  expect(mockSmtpClient.sendEmail).not.toHaveBeenCalled();
});
```

## Supporting Types

```typescript
interface EmailData {
  to: string;
  subject: string;
  content: {
    html: string;
    text: string;
  };
  from?: {
    name: string;
    email: string;
  };
}

interface EmailResult {
  messageId: string;
  status: EmailStatus;
  sentAt: Date;
}
```

## Guidelines

1. **Test Isolation**: Each test runs independently without shared state
2. **Mock External Dependencies**: SMTP clients, databases, and external APIs
3. **Test Edge Cases**: Empty strings, null values, boundary conditions
4. **Descriptive Names**: Test names describe the scenario and expected outcome
5. **Arrange-Act-Assert**: Clear separation of setup, execution, and verification

## Related Documentation

- [Testing Overview](README)
- [Integration Testing](integration-testing)
- [Best Practices](best-practices)
