---
title: "Integration Testing Standards"
description: "API endpoint testing with database integration"
last_modified_date: "2025-12-15"
level: "3"
persona: "Backend Engineers"
keywords: "integration testing, API testing, database, MSW, campaign API"
---

# Integration Testing Standards

Integration tests validate component interactions including API endpoints, database operations, and external service integrations. These tests use mocked external services while testing real application logic.

## Test Setup

```typescript
// tests/integration/test-campaign-api.ts
import { describe, it, expect, beforeAll, afterAll, beforeEach } from 'vitest';
import { setupServer } from 'msw/node';
import { http, HttpResponse } from 'msw';

const mockDatabase = {
  campaigns: {
    create: vi.fn(),
    findMany: vi.fn(),
    findById: vi.fn(),
    update: vi.fn()
  },
  users: {
    findByEmail: vi.fn(),
    create: vi.fn()
  }
};

vi.mock('../../app/database/connection', () => ({
  Database: {
    getInstance: () => mockDatabase
  }
}));

describe('Campaign API Integration Tests', () => {
  let testUser: TestUser;

  beforeAll(async () => {
    await setupTestDatabase();
    testUser = await createTestUser();
  });

  afterAll(async () => {
    await cleanupTestDatabase();
  });

  beforeEach(() => {
    vi.clearAllMocks();
  });
});
```

## Campaign Creation Tests

### Successful Creation

```typescript
it('should create campaign successfully', async () => {
  const campaignData: CreateCampaignRequest = {
    name: 'Test Campaign',
    subject: 'Test Subject Line',
    content: {
      html: '<h1>Test Campaign</h1><p>This is a test campaign.</p>',
      text: 'Test Campaign - This is a test campaign.'
    },
    recipients: [
      {
        email: 'recipient@example.com',
        personalization: { name: 'Test Recipient' }
      }
    ],
    settings: {
      analyticsEnabled: true,
      trackOpens: true,
      trackClicks: true,
      aiOptimization: false
    }
  };

  mockDatabase.campaigns.create.mockResolvedValue({
    id: 'camp_123',
    ...campaignData,
    status: 'draft',
    createdAt: new Date(),
    updatedAt: new Date(),
    metrics: { sent: 0, delivered: 0, opened: 0, clicked: 0 }
  });

  const result = await createCampaign(campaignData, testUser.id);

  expect(result.success).toBe(true);
  expect(result.data.name).toBe(campaignData.name);
  expect(result.data.status).toBe('draft');
  expect(result.data.metrics.sent).toBe(0);

  expect(mockDatabase.campaigns.create).toHaveBeenCalledTimes(1);
});
```

### Duplicate Name Rejection

```typescript
it('should reject campaign creation with duplicate name', async () => {
  const campaignData: CreateCampaignRequest = {
    name: 'Duplicate Name Test',
    subject: 'Test Subject',
    content: { html: '<p>Test</p>', text: 'Test' },
    recipients: [{ email: 'test@example.com' }]
  };

  mockDatabase.campaigns.create.mockRejectedValue(
    new Error('Campaign name already exists')
  );

  await expect(createCampaign(campaignData, testUser.id))
    .rejects.toThrow('Campaign name already exists');
});
```

### Field Validation

```typescript
it('should validate required fields', async () => {
  const invalidData: CreateCampaignRequest = {
    name: '',
    subject: 'Test Subject',
    content: { html: '<p>Test</p>', text: 'Test' },
    recipients: []
  };

  await expect(createCampaign(invalidData, testUser.id))
    .rejects.toThrow('Campaign name is required');
});
```

## Campaign Listing Tests

### Pagination

```typescript
describe('Campaign Listing', () => {
  beforeEach(async () => {
    const campaigns = Array.from({ length: 5 }, (_, i) => ({
      id: `camp_${i}`,
      name: `Test Campaign ${i + 1}`,
      subject: `Test Subject ${i + 1}`,
      status: 'draft',
      userId: testUser.id
    }));

    mockDatabase.campaigns.findMany.mockResolvedValue(campaigns);
  });

  it('should list campaigns with pagination', async () => {
    const paginationParams: PaginationParams = { page: 1, perPage: 3 };

    const mockCampaigns = Array.from({ length: 3 }, (_, i) => ({
      id: `camp_${i}`,
      name: `Test Campaign ${i + 1}`
    }));

    mockDatabase.campaigns.findMany.mockResolvedValue(mockCampaigns);

    const result = await getCampaigns(testUser.id, paginationParams);

    expect(result.success).toBe(true);
    expect(result.data).toHaveLength(3);
    expect(result.meta.pagination.page).toBe(1);
    expect(result.meta.pagination.perPage).toBe(3);
    expect(result.meta.pagination.total).toBe(5);
  });
});
```

## Campaign Management Tests

### Update Operations

```typescript
it('should update campaign successfully', async () => {
  const campaignId = 'camp_123';
  const updateData: UpdateCampaignRequest = {
    name: 'Updated Campaign Name',
    subject: 'Updated Subject'
  };

  const existingCampaign = {
    id: campaignId,
    name: 'Original Campaign',
    status: 'draft',
    userId: testUser.id
  };

  mockDatabase.campaigns.findById.mockResolvedValue(existingCampaign);
  mockDatabase.campaigns.update.mockResolvedValue({
    ...existingCampaign,
    ...updateData,
    updatedAt: new Date()
  });

  const result = await updateCampaign(campaignId, updateData, testUser.id);

  expect(result.success).toBe(true);
  expect(result.data.name).toBe(updateData.name);
});
```

### Authorization Checks

```typescript
it('should prevent unauthorized campaign updates', async () => {
  const campaignId = 'camp_456';
  const updateData = { name: 'Updated Name' };
  const campaign = {
    id: campaignId,
    userId: 'different_user_id'
  };

  mockDatabase.campaigns.findById.mockResolvedValue(campaign);

  await expect(updateCampaign(campaignId, updateData, testUser.id))
    .rejects.toThrow('Unauthorized');
});
```

## Supporting Functions

```typescript
async function setupTestDatabase(): Promise<void> {
  console.log('Setting up test database...');
}

async function cleanupTestDatabase(): Promise<void> {
  console.log('Cleaning up test database...');
}

async function createTestUser(): Promise<TestUser> {
  return {
    id: 'user_123',
    email: 'test@example.com',
    token: 'mock_jwt_token'
  };
}
```

## Supporting Types

```typescript
interface TestUser {
  id: string;
  email: string;
  token: string;
}

interface CreateCampaignRequest {
  name: string;
  subject: string;
  content: { html: string; text: string };
  recipients: Array<{
    email: string;
    personalization?: Record<string, unknown>;
  }>;
  settings?: {
    analyticsEnabled?: boolean;
    trackOpens?: boolean;
    trackClicks?: boolean;
    aiOptimization?: boolean;
  };
}

interface UpdateCampaignRequest {
  name?: string;
  subject?: string;
  content?: { html?: string; text?: string };
}

interface PaginationParams {
  page: number;
  perPage: number;
}

interface PaginationMeta {
  page: number;
  perPage: number;
  total: number;
  totalPages: number;
}
```

## Guidelines

1. **Test Real Dependencies**: Use actual database operations where practical
2. **Clean State**: Reset database state between tests to prevent interference
3. **Error Handling**: Test failure scenarios and error responses
4. **Performance**: Monitor response times for endpoints under load
5. **Authorization**: Verify access control for all protected endpoints

## Related Documentation

- [Testing Overview](README)
- [Unit Testing](unit-testing)
- [E2E Testing](e2e-testing)
