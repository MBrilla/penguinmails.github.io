---
title: "Testing Requirements"
description: "Testing standards and quality assurance documentation hub"
last_modified_date: "2025-12-15"
level: "2"
persona: "Quality Assurance"
keywords: "testing, unit tests, integration tests, E2E, quality assurance"
---

# Testing Requirements

This documentation covers testing standards, quality requirements, and validation guidelines for PenguinMails. All code contributions must meet these testing thresholds before merge.

## Coverage Requirements

| Category | Minimum Coverage |
|----------|-----------------|
| Overall codebase | 80% |
| Critical paths | 90% |
| Business logic | 95% |
| API endpoints | 100% request/response |
| New code | Must maintain existing percentage |

## Test Directory Structure

```
tests/
├── unit/                          # Unit tests
│   ├── test_email_service.py      # Service layer tests
│   ├── test_ai_optimizer.py       # AI component tests
│   └── test_models.py             # Model tests
├── integration/                   # Integration tests
│   ├── test_api_campaigns.py      # API endpoint tests
│   ├── test_database_operations.py# Database integration
│   └── test_external_services.py  # External API integration
├── e2e/                          # End-to-end tests
│   ├── test_campaign_workflow.py  # User workflow tests
│   └── test_analytics_dashboard.py# Dashboard tests
├── fixtures/                     # Test data
│   ├── sample_campaigns.json      # Sample campaign data
│   └── user_profiles.json         # User profile data
└── conftest.py                   # Pytest configuration
```

## Testing Documentation

### [Unit Testing](unit-testing)

Service layer testing with mocked dependencies. Covers email service, AI optimizer, and model validation with Vitest examples.

### [Integration Testing](integration-testing)

API endpoint testing with database integration. Covers campaign creation, listing, and management with mock server patterns.

### [E2E Testing](e2e-testing)

Complete user workflow testing with Playwright. Covers campaign creation, AI optimization, and mobile responsiveness.

### [Best Practices](best-practices)

Testing guidelines, test data management, and fixture organization. Includes validation test cases and data generators.

## Running Tests

```bash
npm test                    # Run all tests
npm run test:unit           # Run unit tests only
npm run test:integration    # Run integration tests only
npm run test:e2e            # Run E2E tests only
npm run test:coverage       # Run tests with coverage
npm run test:watch          # Run tests in watch mode
npm run test:ci             # Run tests on CI
```

## Related Documentation

- [Code Standards](/docs/implementation-technical/development-guidelines/code-standards/overview)
- [Documentation Contributions](/docs/implementation-technical/development-guidelines/documentation-contributions)
