---
title: "Resilience Patterns"
last_modified_date: "2025-12-15"
level: "3"
persona: "Developers, Platform Engineers"
---

# Resilience Patterns

Exponential backoff, circuit breaker, and retry logic for robust API automation.

## Exponential Backoff

```javascript
async function exponentialBackoff(fn, options = {}) {
  const {
    maxRetries = 3,
    initialDelayMs = 1000,
    maxDelayMs = 30000,
    backoffMultiplier = 2
  } = options;

  let lastError;

  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error;

      // Don't retry on client errors
      if (isClientError(error)) {
        throw error;
      }

      // Don't retry if max attempts reached
      if (attempt === maxRetries) {
        break;
      }

      // Calculate delay with exponential backoff
      const delay = Math.min(
        initialDelayMs * Math.pow(backoffMultiplier, attempt),
        maxDelayMs
      );

      console.log(`Attempt ${attempt + 1} failed. Retrying in ${delay}ms...`);
      await sleep(delay);
    }
  }

  throw new Error(`Max retries (${maxRetries}) exceeded. Last error: ${lastError.message}`);
}

function isClientError(error) {
  const clientErrorMessages = [
    'invalid serviceid',
    'Location Id is invalid',
    'A valid serviceid is required'
  ];
  return clientErrorMessages.some(msg =>
    error.message.toLowerCase().includes(msg.toLowerCase())
  );
}
```

## Circuit Breaker Pattern

```javascript
class CircuitBreaker {
  constructor(options = {}) {
    this.failureThreshold = options.failureThreshold || 5;
    this.resetTimeout = options.resetTimeout || 60000; // 1 minute
    this.state = 'CLOSED'; // CLOSED, OPEN, HALF_OPEN
    this.failureCount = 0;
    this.lastFailureTime = null;
  }

  async execute(fn) {
    if (this.state === 'OPEN') {
      if (Date.now() - this.lastFailureTime > this.resetTimeout) {
        this.state = 'HALF_OPEN';
      } else {
        throw new Error('Circuit breaker is OPEN');
      }
    }

    try {
      const result = await fn();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }

  onSuccess() {
    this.failureCount = 0;
    this.state = 'CLOSED';
  }

  onFailure() {
    this.failureCount++;
    this.lastFailureTime = Date.now();

    if (this.failureCount >= this.failureThreshold) {
      this.state = 'OPEN';
      console.error(`Circuit breaker opened after ${this.failureCount} failures`);
    }
  }
}
```

## Using Both Patterns Together

```javascript
class ResilientHostwindsClient {
  constructor(apiKey) {
    this.apiKey = apiKey;
    this.circuitBreaker = new CircuitBreaker();
    this.rateLimiter = new RateLimiter(2);
  }

  async callAPI(action, params) {
    return await this.circuitBreaker.execute(async () => {
      return await this.rateLimiter.execute(async () => {
        return await exponentialBackoff(async () => {
          return await hostwindsAPI(action, { ...params, API: this.apiKey });
        });
      });
    });
  }
}
```

---

[← Back to Automation Best Practices](./README)
