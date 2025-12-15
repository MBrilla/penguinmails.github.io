---
title: "Error Handling and Logging"
last_modified_date: "2025-12-15"
level: "3"
persona: "Developers, Platform Engineers"
---

# Error Handling and Logging

Response validation, error classification, and logging best practices for Hostwinds API.

## Critical: Check the `result` Field

> **CAUTION**: HTTP 200 Does Not Mean Success. Hostwinds API returns HTTP 200 for both successful and failed requests. Always check the `result` field in the response body.

```javascript
const response = await fetch(HOSTWINDS_API_URL, {
  method: 'POST',
  body: formData
});

// ❌ WRONG: Assuming 200 = success
if (response.status === 200) {
  const data = await response.json();
  // Process data...
}

// ✅ CORRECT: Check result field
if (response.status === 200) {
  const data = await response.json();
  if (data.result === 'error') {
    throw new Error(`Hostwinds API Error: ${data.message}`);
  }
  // Process successful data...
}
```

## Error Response Handling

### Standard Error Format

```json
[
  {
    "result": "error",
    "action": "Add Instance",
    "message": "Location Id is invalid.",
    "ERROR": "342432"
  }
]
```

### Error Handling Strategy

1. **Prioritize the `result` and `message` Fields**:
   - Always check if `result === "error"`
   - Use the `message` field for human-readable debugging
   - Log the full error object for troubleshooting

2. **Handling Proprietary Error Codes**:
   - Log the internal `ERROR` code for Hostwinds support reference
   - **Do not** use this code to drive automation logic
   - Error codes may change without notice

3. **Client vs. Server Errors**:
   - **Client Data Errors**: Invalid input (e.g., "invalid serviceid", "Location Id is invalid")
     - **Do not retry** - Fix the input data
     - Log the error and alert developers
   - **Transient Errors**: Network issues or temporary server problems
     - **Implement exponential backoff**
     - **Retry up to 3 times**
     - Log retry attempts

## Logging Best Practices

```javascript
class HostwindsAPILogger {
  logRequest(action, params) {
    console.log({
      timestamp: new Date().toISOString(),
      type: 'request',
      action,
      params: this.sanitizeParams(params),
      correlationId: this.generateCorrelationId()
    });
  }

  logResponse(action, response, duration) {
    console.log({
      timestamp: new Date().toISOString(),
      type: 'response',
      action,
      result: response.result,
      message: response.message,
      duration,
      correlationId: this.getCurrentCorrelationId()
    });
  }

  logError(action, error, context) {
    console.error({
      timestamp: new Date().toISOString(),
      type: 'error',
      action,
      error: error.message,
      stack: error.stack,
      context,
      correlationId: this.getCurrentCorrelationId()
    });
  }

  sanitizeParams(params) {
    const sanitized = { ...params };
    delete sanitized.API; // Never log API keys
    return sanitized;
  }
}
```

---

[← Back to Automation Best Practices](./README)
