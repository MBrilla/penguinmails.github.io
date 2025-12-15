---
title: "E2E Testing Standards"
description: "End-to-end workflow testing with Playwright"
last_modified_date: "2025-12-15"
level: "3"
persona: "QA Engineers"
keywords: "E2E testing, Playwright, user workflows, mobile testing"
---

# E2E Testing Standards

End-to-end tests validate complete user workflows through the application interface. These tests use Playwright to simulate real user interactions across browsers and devices.

## Test Setup

```typescript
// tests/e2e/campaign-workflow.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Campaign Workflow E2E Tests', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/login');
    await page.fill('[data-testid="email"]', 'test@example.com');
    await page.fill('[data-testid="password"]', 'testpass123');
    await page.click('[data-testid="login-button"]');

    await expect(page.locator('[data-testid="dashboard-header"]')).toBeVisible();
  });
});
```

## Campaign Creation Workflow

```typescript
test('complete campaign creation and sending workflow', async ({ page }) => {
  await page.click('[data-testid="create-campaign"]');
  await expect(page.locator('[data-testid="campaign-editor"]')).toBeVisible();

  await page.fill('[data-testid="campaign-name"]', 'E2E Test Campaign');
  await page.fill('[data-testid="campaign-subject"]', 'E2E Test Subject Line');
  await page.fill('[data-testid="content-html"]', '<h1>E2E Test Campaign</h1>');

  await page.click('[data-testid="add-recipient"]');
  await page.fill('[data-testid="recipient-email"]', 'e2e-test@example.com');
  await page.fill('[data-testid="recipient-name"]', 'E2E Test User');
  await page.click('[data-testid="save-recipient"]');

  await page.check('[data-testid="analytics-enabled"]');
  await page.check('[data-testid="track-opens"]');
  await page.check('[data-testid="track-clicks"]');

  await page.click('[data-testid="save-campaign"]');

  await expect(page.locator('[data-testid="success-message"]'))
    .toContainText('Campaign created successfully');

  await page.click('[data-testid="campaigns-nav"]');
  await expect(page.locator('text=E2E Test Campaign')).toBeVisible();

  await page.click('[data-testid="send-campaign"]');
  await page.click('[data-testid="confirm-send"]');

  await expect(page.locator('[data-testid="send-confirmation"]'))
    .toContainText('Campaign sent successfully');
});
```

## AI Optimization Workflow

```typescript
test('AI optimization workflow', async ({ page }) => {
  await page.click('[data-testid="create-campaign"]');

  await page.fill('[data-testid="campaign-name"]', 'AI Optimization Test');
  await page.fill('[data-testid="campaign-subject"]', 'Basic Subject Line');
  await page.fill('[data-testid="content-html"]', '<h1>AI Test Content</h1>');

  await page.click('[data-testid="add-recipient"]');
  await page.fill('[data-testid="recipient-email"]', 'ai-test@example.com');
  await page.click('[data-testid="save-recipient"]');

  await page.check('[data-testid="ai-optimization"]');
  await page.click('[data-testid="optimize-with-ai"]');

  await expect(page.locator('[data-testid="optimization-loading"]')).toBeVisible();
  await expect(page.locator('[data-testid="optimization-complete"]')).toBeVisible();

  const optimizedSubject = await page.inputValue('[data-testid="campaign-subject"]');
  expect(optimizedSubject).not.toBe('Basic Subject Line');

  await expect(page.locator('[data-testid="optimization-score"]')).toBeVisible();
  const scoreText = await page.textContent('[data-testid="optimization-score"]');
  expect(scoreText).toMatch(/\d+% improvement/);
});
```

## Mobile Responsiveness

```typescript
test('mobile responsive campaign creation', async ({ page }) => {
  await page.setViewportSize({ width: 375, height: 667 });

  await page.click('[data-testid="mobile-menu"]');
  await page.click('[data-testid="create-campaign-mobile"]');

  await expect(page.locator('[data-testid="mobile-editor"]')).toBeVisible();

  await page.fill('[data-testid="campaign-name-mobile"]', 'Mobile Test Campaign');
  await page.fill('[data-testid="campaign-subject-mobile"]', 'Mobile Test Subject');
  await page.fill('[data-testid="content-text-mobile"]', 'Mobile test content');

  await page.click('[data-testid="add-recipient-mobile"]');
  await page.fill('[data-testid="recipient-email-mobile"]', 'mobile-test@example.com');
  await page.click('[data-testid="save-recipient-mobile"]');

  await page.click('[data-testid="save-campaign-mobile"]');

  await expect(page.locator('[data-testid="mobile-success"]'))
    .toContainText('Campaign created');
});
```

## Guidelines

1. **User Journey Focus**: Test complete workflows from start to finish
2. **Realistic Data**: Use realistic test data matching production patterns
3. **Cross-Browser**: Run tests in Chrome, Firefox, and Safari
4. **Mobile Testing**: Test responsive layouts at various viewport sizes
5. **Performance**: Monitor page load times and interaction responsiveness

## Running E2E Tests

```bash
npm run test:e2e           # Run all E2E tests
npm run test:e2e:ui        # Run with Playwright UI
npm run test:e2e:headed    # Run in headed browser mode
npm run test:e2e:chrome    # Run in Chrome only
npm run test:e2e:mobile    # Run mobile viewport tests
```

## Related Documentation

- [Testing Overview](README)
- [Integration Testing](integration-testing)
- [Best Practices](best-practices)
