---
title: "State & Security"
description: State management, error handling, security, and accessibility
last_modified_date: 2025-01-15
level: 3
persona: ["frontend-developer"]
keywords: [state, security, errors, accessibility, API-keys]
---

# State & Security

State management, error handling, security considerations, and accessibility for API key management.

## State Management

```typescript
interface APIKeyListState {
  apiKeys: APIKey[];
  loading: boolean;
  error: string | null;
  selectedKeyId: string | null;
  showCreateModal: boolean;
  showDetailsModal: boolean;
  showRegenerateModal: boolean;
  showRevokeModal: boolean;
}

interface APIKey {
  key_id: string;
  name: string;
  masked_key: string;
  permissions: string[];
  rate_limit: number;
  status: 'active' | 'revoked';
  created_at: string;
  last_used: string | null;
  request_count: number;
  error_count: number;
}
```

## Actions

```typescript
// Load API keys
async function loadAPIKeys(): Promise<void> {
  setState({ loading: true, error: null });
  try {
    const response = await fetch('/api/v1/platform/api-keys');
    const data = await response.json();
    setState({ apiKeys: data.api_keys, loading: false });
  } catch (error) {
    setState({ error: error.message, loading: false });
  }
}

// Create API key
async function createAPIKey(name: string, permissions: string[]): Promise<string> {
  const response = await fetch('/api/v1/platform/api-keys', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ name, permissions })
  });
  const data = await response.json();
  return data.api_key;
}

// Regenerate API key
async function regenerateAPIKey(keyId: string): Promise<string> {
  const response = await fetch(`/api/v1/platform/api-keys/${keyId}/regenerate`, {
    method: 'POST'
  });
  const data = await response.json();
  return data.api_key;
}

// Revoke API key
async function revokeAPIKey(keyId: string): Promise<void> {
  await fetch(`/api/v1/platform/api-keys/${keyId}`, {
    method: 'DELETE'
  });
  await loadAPIKeys();
}
```

## Error Handling

### API Errors

| Status | Response | Action |
|--------|----------|--------|
| 401 | Unauthorized | Redirect to login, show toast |
| 403 | Forbidden | Show permission error, disable actions |
| 429 | Rate Limited | Show retry message, disable buttons |
| 500 | Server Error | Show error message, log to monitoring |

### Validation Errors

| Field | Error | Message |
|-------|-------|---------|
| Name | Empty | "Name is required" |
| Name | Too long | "Name must be 50 characters or less" |
| Permissions | None selected | "Select at least one permission" |

### Network Errors

- **Connection failed**: Show retry button
- **Timeout**: Show retry button with message

## Security Considerations

### API Key Display

- Display full key only once in success modal
- Require explicit action to view
- Warn user to store securely
- Never display full key again (only masked)

### Copy to Clipboard

- Use secure clipboard API (`navigator.clipboard.writeText()`)
- Show toast confirmation
- Consider clearing clipboard after 60 seconds

### Download .env File

- Include comment: "# DO NOT COMMIT THIS FILE TO VERSION CONTROL"
- Suggest adding `.env` to `.gitignore`
- Link to security best practices

### Modal Security

- Disable backdrop click to close success modal
- Require explicit "Done" button click
- Show confirmation if user tries to close without copying

### Regenerate Confirmation

- Require explicit confirmation
- Show warning about immediate revocation
- Display key name and masked value for verification

## Accessibility

### Keyboard Navigation

- Tab through all interactive elements
- Enter/Space to activate buttons
- Escape to close modals
- Arrow keys to navigate table rows

### Screen Reader Support

- ARIA labels for all buttons and inputs
- ARIA live regions for toast notifications
- ARIA descriptions for complex interactions
- Semantic HTML (table, form, button elements)

### Visual Accessibility

- High contrast colors (WCAG AA compliant)
- Focus indicators on all interactive elements
- Error messages with icons and text
- Loading states with spinners and text
