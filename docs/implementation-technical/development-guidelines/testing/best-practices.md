---
title: "Testing Best Practices"
description: "Testing guidelines, fixtures, and data management"
last_modified_date: "2025-12-15"
level: "3"
persona: "Quality Assurance"
keywords: "test fixtures, test data, validation, best practices"
---

# Testing Best Practices

This document covers testing guidelines, test data management, and fixture organization for consistent and maintainable test suites.

## Unit Testing Guidelines

1. **Test Isolation**: Each test runs independently without shared mutable state
2. **Arrange-Act-Assert**: Clear separation of setup, execution, and verification
3. **Mock External Dependencies**: SMTP clients, databases, and third-party APIs
4. **Test Edge Cases**: Empty strings, null values, boundary conditions
5. **Descriptive Names**: Test names describe the scenario and expected outcome

## Integration Testing Guidelines

1. **Test Real Dependencies**: Use actual database connections when practical
2. **Clean State**: Reset database state between tests to prevent interference
3. **Error Handling**: Test failure scenarios and error response formats
4. **Performance**: Monitor response times for endpoints under load

## E2E Testing Guidelines

1. **User Journey Focus**: Test complete workflows from start to finish
2. **Realistic Data**: Use data patterns matching production usage
3. **Cross-Browser**: Test in Chrome, Firefox, and Safari
4. **Mobile Testing**: Verify responsive layouts at multiple viewport sizes
5. **Performance**: Monitor page load times and interaction latency

## Test Data Management

### Sample Campaign Data

```typescript
// tests/fixtures/sample-campaigns.ts
export const sampleCampaignData: CampaignData = {
  name: 'Test Campaign',
  subject: 'Test Subject Line',
  content: {
    html: '<h1>Test Campaign</h1><p>This is a test campaign.</p>',
    text: 'Test Campaign - This is a test campaign.'
  },
  recipients: [
    {
      email: 'test@example.com',
      personalization: { name: 'Test User' }
    }
  ],
  settings: {
    analyticsEnabled: true,
    trackOpens: true,
    trackClicks: true,
    aiOptimization: false
  }
};
```

### Bulk Recipient Generation

```typescript
export function createBulkRecipients(count: number = 250): BulkRecipients {
  return Array.from({ length: count }, (_, i) => ({
    email: `user${i + 1}@example.com`,
    personalization: {
      name: `User ${i + 1}`,
      company: `Company ${i + 1}`
    }
  }));
}
```

### Invalid Data for Negative Testing

```typescript
export const invalidRecipientData: CampaignData = {
  name: 'Invalid Campaign',
  subject: 'Test Subject',
  content: { html: '<p>Test</p>', text: 'Test' },
  recipients: [
    { email: 'invalid-email' },
    { name: 'No Email User' },
    { email: '' },
    { email: 'missing@company', personalization: { name: '' } }
  ]
};
```

## Campaign Templates

```typescript
export const campaignTemplates = {
  welcomeCampaign: {
    name: 'Welcome New Users',
    subject: 'Welcome to Our Platform, {{name}}!',
    content: {
      html: '<h1>Welcome {{name}}!</h1><p>We are excited to have you.</p>',
      text: 'Welcome {{name}}! We are excited to have you.'
    }
  },

  promotionalCampaign: {
    name: 'Black Friday Sale',
    subject: 'Limited Time: 50% Off Everything!',
    content: {
      html: '<h1>Black Friday Sale!</h1><p>Use code: BLACKFRIDAY</p>',
      text: 'Black Friday Sale! Use code: BLACKFRIDAY'
    },
    settings: { trackOpens: true, trackClicks: true }
  },

  newsletterCampaign: {
    name: 'Weekly Newsletter',
    subject: 'Your Weekly Tech Update',
    content: {
      html: '<h1>Tech News This Week</h1><p>Latest updates.</p>',
      text: 'Tech News This Week - Latest updates.'
    }
  }
};
```

## Test Data Generators

```typescript
export class TestDataGenerator {
  static generateRecipients(count: number): RecipientData[] {
    return Array.from({ length: count }, (_, i) => ({
      email: `test${i + 1}@example.com`,
      personalization: {
        name: `Test User ${i + 1}`,
        company: `Test Company ${i + 1}`
      }
    }));
  }

  static generateCampaignName(index: number): string {
    const prefixes = ['Test', 'Demo', 'Sample', 'Example', 'Mock'];
    const suffixes = ['Campaign', 'Email', 'Newsletter', 'Update'];
    const prefix = prefixes[index % prefixes.length];
    const suffix = suffixes[Math.floor(index / prefixes.length) % suffixes.length];
    return `${prefix} ${suffix} ${index + 1}`;
  }

  static generateEmailContent(type: 'welcome' | 'promotional' | 'newsletter'): EmailContent {
    const templates = {
      welcome: {
        html: '<h1>Welcome {{name}}!</h1><p>Thank you for joining.</p>',
        text: 'Welcome {{name}}! Thank you for joining.'
      },
      promotional: {
        html: '<h1>Special Offer!</h1><p>Get 25% off.</p>',
        text: 'Special Offer! Get 25% off.'
      },
      newsletter: {
        html: '<h1>Weekly Update</h1><p>This week updates.</p>',
        text: 'Weekly Update - This week updates.'
      }
    };
    return templates[type];
  }
}
```

## Validation Test Cases

```typescript
export const validationTestCases = {
  validEmails: [
    'user@example.com',
    'test.user@company.co.uk',
    'name+tag@domain.com',
    'user123@test-domain.org'
  ],

  invalidEmails: [
    'invalid-email',
    '@domain.com',
    'user@',
    'user..double.dot@example.com',
    'user@domain',
    'user@.domain.com'
  ],

  edgeCaseNames: [
    '',
    'A',
    'a'.repeat(256),
    'Campaign with "quotes"',
    'Campaign with <script>tags</script>',
    'Campaign with special chars: !@#$%^&*()'
  ]
};
```

## Test Constants

```typescript
export const TEST_CONSTANTS = {
  VALID_EMAIL: 'test@example.com',
  INVALID_EMAIL: 'invalid-email',
  EMPTY_EMAIL: '',
  MAX_RECIPIENTS: 10000,
  BATCH_SIZE: 100,
  CAMPAIGN_NAME_MAX_LENGTH: 255,
  SUBJECT_LINE_MAX_LENGTH: 100,
  EMAIL_CONTENT_MAX_LENGTH: 50000
};
```

## Test Fixtures Usage

```typescript
import { testFixtures, TEST_CONSTANTS, validationTestCases } from './sample-campaigns';

describe('Campaign Validation', () => {
  it('should validate email addresses correctly', () => {
    validationTestCases.validEmails.forEach(email => {
      expect(isValidEmail(email)).toBe(true);
    });

    validationTestCases.invalidEmails.forEach(email => {
      expect(isValidEmail(email)).toBe(false);
    });
  });
});
```

## Related Documentation

- [Testing Overview](README)
- [Unit Testing](unit-testing)
- [Integration Testing](integration-testing)
- [E2E Testing](e2e-testing)
