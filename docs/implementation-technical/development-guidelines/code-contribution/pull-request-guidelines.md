---
title: "Pull Request Guidelines"
description: "PR template, description requirements, and review timeline standards for contributions"
last_modified_date: "2025-01-10"
level: "2"
persona: "Developers"
keywords: [pull-request, pr-template, review-timeline, merge-requirements]
---

# Pull Request Guidelines

Pull requests are the primary mechanism for contributing changes. Following these guidelines ensures efficient review and merge processes.

## PR Template

Use this template when creating pull requests:

```markdown
## Description

Brief description of changes and motivation.

## Type of Change

- [ ] Bug fix (non-breaking change which fixes an issue)
- [ ] New feature (non-breaking change which adds functionality)
- [ ] Breaking change (fix or feature that would cause existing functionality to not work as expected)
- [ ] Documentation update
- [ ] Performance improvement
- [ ] Code refactoring

## Testing

- [ ] Unit tests pass locally
- [ ] Integration tests updated
- [ ] Manual testing completed
- [ ] Performance impact assessed

## Checklist

- [ ] Code follows style guidelines
- [ ] Self-review completed
- [ ] Documentation updated
- [ ] Changelog updated
- [ ] No security vulnerabilities

## Screenshots (if applicable)

Add screenshots to help explain your changes.

## Additional Notes

Any additional information or context.
```

## Before Submitting

Complete these steps before creating a pull request:

1. **Update your branch** - Ensure your branch is up to date with upstream
2. **Run tests** - All tests must pass locally
3. **Check linting** - Code must pass all linting checks
4. **Update documentation** - Update relevant documentation
5. **Add tests** - Include tests for new functionality
6. **Self-review** - Review your own changes first

## PR Description Requirements

### Clear Title

Use conventional commit format for PR titles:

```text
feat(ai): add email content optimization algorithm
fix(api): resolve validation error in campaign creation
docs(api): update campaign analytics endpoint documentation
```

### Detailed Description

Your description should explain:

- **What** - What changes are included
- **Why** - Motivation for the changes
- **How** - Technical approach taken

### Visual Changes

Include screenshots or GIFs for:

- UI changes
- New features with user-facing components
- Before/after comparisons

### Testing Description

Document testing performed:

- Specific scenarios tested
- Edge cases considered
- Performance impact (if relevant)

### Breaking Changes

Clearly highlight any breaking changes:

```markdown
## Breaking Changes

This PR changes the response format for `/api/v1/campaigns`:
- `created_at` is now ISO 8601 format (was Unix timestamp)
- `status` enum values changed to lowercase

Migration guide: Update all clients to handle new format.
```

## Review Timeline

| Phase | Timeline | Description |
|-------|----------|-------------|
| Initial review | 24-48 hours | First reviewer assigned and begins review |
| Follow-up reviews | Within 24 hours | Response to author updates |
| Final approval | Same day | After all feedback addressed |
| Merge | Automated | Once all checks pass |

## Merge Requirements

Before merging:

- [ ] All reviewers approved
- [ ] All automated checks pass
- [ ] No unresolved conversations
- [ ] Branch is up to date with target

## Merge Strategy

| Strategy | Use Case |
|----------|----------|
| Squash and merge | Most PRs - clean history |
| Merge commit | Large PRs with meaningful commit history |
| Rebase and merge | Linear history requirement |

## After Merge

Post-merge responsibilities:

1. Delete feature branch
2. Verify deployment (if applicable)
3. Monitor for regressions
4. Update related issues/tickets

## Related Documentation

- [Overview](/docs/implementation-technical/development-guidelines/code-contribution/overview) - Development workflow
- [Contribution Types](/docs/implementation-technical/development-guidelines/code-contribution/contribution-types) - Bug fixes, features
- [Code Review](/docs/implementation-technical/development-guidelines/code-contribution/code-review) - Review process
- [Testing Requirements](/docs/implementation-technical/development-guidelines/testing-requirements) - Test standards
- [Code Standards](/docs/implementation-technical/development-guidelines/code-standards/overview) - Coding conventions

---

**Document Classification:** Level 2 - Developer Implementation
**Implementation Access:** Developers
