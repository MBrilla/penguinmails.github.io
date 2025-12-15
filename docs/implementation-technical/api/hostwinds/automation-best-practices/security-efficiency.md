---
title: "Security and Efficiency"
last_modified_date: "2025-12-15"
level: "3"
persona: "Developers, Platform Engineers"
---

# Security and Efficiency

Secure credential storage, input validation, and rate limiting patterns.

## Secure Credential Storage

> **CAUTION**: Never hardcode API keys in source code or commit them to version control.

### Recommended Approaches

**1. Environment Variables**:

```javascript
// .env file (never commit this)
HOSTWINDS_API_KEY=your_api_key_here

// Application code
const apiKey = process.env.HOSTWINDS_API_KEY;
if (!apiKey) {
  throw new Error('HOSTWINDS_API_KEY environment variable not set');
}
```

**2. Secrets Manager (AWS, Azure, GCP)**:

```javascript
const { SecretsManagerClient, GetSecretValueCommand } = require('@aws-sdk/client-secrets-manager');

async function getHostwindsAPIKey() {
  const client = new SecretsManagerClient({ region: 'us-east-1' });
  const response = await client.send(
    new GetSecretValueCommand({ SecretId: 'hostwinds/api-key' })
  );
  return JSON.parse(response.SecretString).apiKey;
}
```

**3. Encrypted Configuration**:

```javascript
const crypto = require('crypto');

function decryptAPIKey(encryptedKey, masterKey) {
  const decipher = crypto.createDecipher('aes-256-cbc', masterKey);
  let decrypted = decipher.update(encryptedKey, 'hex', 'utf8');
  decrypted += decipher.final('utf8');
  return decrypted;
}
```

## Input Validation

Validate all inputs before sending to Hostwinds API to avoid common errors.

```javascript
class HostwindsInputValidator {
  static validateServiceId(serviceid) {
    if (!serviceid || typeof serviceid !== 'number') {
      throw new Error('Invalid service ID: must be a number');
    }
    if (serviceid <= 0) {
      throw new Error('Invalid service ID: must be positive');
    }
    return serviceid;
  }

  static validateIPAddress(ip) {
    const ipRegex = /^(\d{1,3}\.){3}\d{1,3}$/;
    if (!ipRegex.test(ip)) {
      throw new Error(`Invalid IP address: ${ip}`);
    }
    const octets = ip.split('.').map(Number);
    if (octets.some(octet => octet < 0 || octet > 255)) {
      throw new Error(`Invalid IP address: ${ip}`);
    }
    return ip;
  }

  static validateLocationId(locationId) {
    const validLocations = [1, 2, 3, 4, 5]; // Update with actual valid IDs
    if (!validLocations.includes(locationId)) {
      throw new Error(`Invalid location ID: ${locationId}`);
    }
    return locationId;
  }

  static validateHostname(hostname) {
    const hostnameRegex = /^[a-z0-9]([a-z0-9-]{0,61}[a-z0-9])?(\.[a-z0-9]([a-z0-9-]{0,61}[a-z0-9])?)*$/i;
    if (!hostnameRegex.test(hostname)) {
      throw new Error(`Invalid hostname: ${hostname}`);
    }
    return hostname;
  }
}
```

## Rate Limiting

Implement rate limiting to prevent hitting Hostwinds API limits and causing transient errors.

```javascript
class RateLimiter {
  constructor(requestsPerSecond = 2) {
    this.requestsPerSecond = requestsPerSecond;
    this.lastRequestTime = 0;
    this.queue = [];
  }

  async throttle() {
    const now = Date.now();
    const timeSinceLastRequest = now - this.lastRequestTime;
    const minInterval = 1000 / this.requestsPerSecond;

    if (timeSinceLastRequest < minInterval) {
      const delay = minInterval - timeSinceLastRequest;
      await sleep(delay);
    }

    this.lastRequestTime = Date.now();
  }

  async execute(fn) {
    await this.throttle();
    return await fn();
  }
}

// Usage
const rateLimiter = new RateLimiter(2); // 2 requests per second

async function callHostwindsAPI(action, params) {
  return await rateLimiter.execute(async () => {
    return await hostwindsAPI(action, params);
  });
}
```

---

[← Back to Automation Best Practices](./README)
