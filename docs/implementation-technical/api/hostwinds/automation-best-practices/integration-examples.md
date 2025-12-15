---
title: "Integration Examples"
last_modified_date: "2025-12-15"
level: "3"
persona: "Developers, Platform Engineers"
---

# Integration Examples

Complete automation examples and metrics collection patterns.

## Complete Automation Example

```javascript
class HostwindsAutomation {
  constructor(apiKey) {
    this.apiKey = apiKey;
    this.rateLimiter = new RateLimiter(2);
    this.circuitBreaker = new CircuitBreaker();
    this.logger = new HostwindsAPILogger();
  }

  async createAndConfigureInstance(config) {
    const { template, rid, locationId, srvrname, rdnsHostname } = config;

    try {
      // Step 1: Create instance
      this.logger.logRequest('add_instance', config);
      const createResponse = await this.callAPI('add_instance', {
        template,
        rid,
        location_id: locationId,
        srvrname
      });

      // Extract service ID from response
      const serviceid = createResponse.serviceid;

      // Step 2: Poll for active status
      const instance = await pollInstanceStatus(serviceid, 'active');

      // Step 3: Configure rDNS
      await this.callAPI('set_rdns', {
        serviceid,
        ip: instance.main_ip,
        rdns: rdnsHostname
      });

      // Step 4: Set firewall profile
      await this.callAPI('set_instance_firewall', {
        serviceid,
        name: 'smtp'
      });

      // Step 5: Update database
      await this.updateDatabase(serviceid, instance);

      return {
        serviceid,
        instance,
        status: 'configured'
      };

    } catch (error) {
      this.logger.logError('createAndConfigureInstance', error, config);
      throw error;
    }
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

  async updateDatabase(serviceid, instance) {
    await db.query(`
      INSERT INTO vps_instances (
        hostwinds_instance_id,
        main_ip,
        approximate_cost,
        status
      ) VALUES ($1, $2, $3, $4)
    `, [serviceid, instance.main_ip, 9.99, instance.status]);
  }
}
```

## Monitoring and Observability

### Metrics to Track

**API Performance**:
- Request latency (p50, p95, p99)
- Error rate by action
- Retry rate
- Circuit breaker state changes

**Business Metrics**:
- Instance creation success rate
- Average provisioning time
- Cost per instance
- API call volume by action

**Operational Metrics**:
- Failed validations
- Timeout rate
- Rate limit hits
- Polling duration

### Example Metrics Collection

```javascript
class MetricsCollector {
  constructor() {
    this.metrics = {
      requests: new Map(),
      errors: new Map(),
      latencies: []
    };
  }

  recordRequest(action, duration, success) {
    const key = `${action}_${success ? 'success' : 'error'}`;
    this.metrics.requests.set(key, (this.metrics.requests.get(key) || 0) + 1);
    this.metrics.latencies.push({ action, duration, timestamp: Date.now() });
  }

  recordError(action, errorType) {
    const key = `${action}_${errorType}`;
    this.metrics.errors.set(key, (this.metrics.errors.get(key) || 0) + 1);
  }

  getMetrics() {
    return {
      requests: Object.fromEntries(this.metrics.requests),
      errors: Object.fromEntries(this.metrics.errors),
      latency: this.calculateLatencyStats()
    };
  }

  calculateLatencyStats() {
    const latencies = this.metrics.latencies.map(l => l.duration);
    latencies.sort((a, b) => a - b);

    return {
      p50: latencies[Math.floor(latencies.length * 0.5)],
      p95: latencies[Math.floor(latencies.length * 0.95)],
      p99: latencies[Math.floor(latencies.length * 0.99)],
      avg: latencies.reduce((a, b) => a + b, 0) / latencies.length
    };
  }
}
```

---

[← Back to Automation Best Practices](./README)
