---
title: "Workflow and State Management"
last_modified_date: "2025-12-15"
level: "3"
persona: "Developers, Platform Engineers"
---

# Workflow and State Management

Service ID validation, asynchronous operations, and billing workflow patterns.

## Service ID Validation

> **IMPORTANT**: Always validate service IDs before executing instance-specific operations.

```javascript
async function safeInstanceOperation(serviceid, operation) {
  // Step 1: Validate service ID exists
  const isValid = await checkServiceId(serviceid);
  if (!isValid) {
    throw new Error(`Invalid service ID: ${serviceid}`);
  }

  // Step 2: Get current instance state
  const instance = await getInstance(serviceid);
  if (instance.status === 'suspended') {
    throw new Error(`Instance ${serviceid} is suspended`);
  }

  // Step 3: Execute operation
  return await operation(serviceid);
}
```

## Asynchronous Operation Handling

Many Hostwinds operations are asynchronous. The success response only means the task was **queued**, not completed.

### Operations Requiring Polling

- `add_instance` - Server creation
- `recreate` - Server recreation
- `reinstall_instance` - OS reinstall
- `upgrade_server` - Resource upgrades
- `regenerate_networking` - Network rebuild

### Polling Strategy

```javascript
async function pollInstanceStatus(serviceid, expectedStatus, options = {}) {
  const {
    maxAttempts = 60,
    intervalMs = 30000, // 30 seconds
    timeoutMs = 1800000  // 30 minutes
  } = options;

  const startTime = Date.now();

  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    // Check timeout
    if (Date.now() - startTime > timeoutMs) {
      throw new Error(`Timeout waiting for status: ${expectedStatus}`);
    }

    // Get current status
    const instance = await getInstance(serviceid);

    // Check for error states
    if (instance.status === 'error') {
      throw new Error(`Instance entered error state: ${instance.disp_status}`);
    }

    // Check for expected status
    if (instance.status === expectedStatus) {
      return instance;
    }

    // Log progress
    console.log(`Attempt ${attempt}/${maxAttempts}: Status is ${instance.status}, waiting for ${expectedStatus}`);

    // Wait before next attempt
    await sleep(intervalMs);
  }

  throw new Error(`Max attempts reached waiting for status: ${expectedStatus}`);
}

function sleep(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}
```

## Billing Workflow (Two-Step Process)

> **WARNING**: Do not apply upgrades before invoice payment. The two-step upgrade process requires invoice payment between steps.

```javascript
async function upgradeInstanceWorkflow(serviceid, rid) {
  // Step 1: Check for existing upgrade orders
  const existingOrder = await checkUpgradeOrder(serviceid);
  if (existingOrder.success) {
    throw new Error('Pending upgrade order exists. Complete or cancel it first.');
  }

  // Step 2: Create upgrade order and invoice
  const order = await upgradeInstance({ serviceid, rid });
  console.log(`Upgrade order created. Invoice ID: ${order.invoiceid}`);

  // Step 3: Wait for invoice payment (external process)
  await waitForInvoicePayment(order.invoiceid);

  // Step 4: Apply the upgrade
  const upgrade = await upgradeServer({ serviceid });

  // Step 5: Poll for RESIZED status
  const instance = await pollInstanceStatus(serviceid, 'RESIZED');

  // Step 6: Confirm the upgrade
  await confirmUpgrade(serviceid);

  // Step 7: Verify final status
  await pollInstanceStatus(serviceid, 'active');

  console.log(`Upgrade completed successfully for service ${serviceid}`);
}
```

---

[← Back to Automation Best Practices](./README)
