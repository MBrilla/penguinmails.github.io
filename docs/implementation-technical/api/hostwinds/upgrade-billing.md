---
title: "Hostwinds Upgrade, Billing, and Pricing API"
description: "Detailed API contract for Hostwinds server upgrades, billing management, and pricing queries"
last_modified_date: "2025-11-19"
level: "3"
persona: "Documentation Users"
---

# Hostwinds Upgrade, Billing, and Pricing API

**For Platform Engineers and Finance Teams**: Complete API reference for Hostwinds resource upgrades, billing cycle management, and cost planning

**Parent Documentation**: [Hostwinds API Overview](/docs/implementation-technical/api/hostwinds/overview)

---

## Pricing and Plan Information

### get_price_list (Get Available Resources with Prices)

**Resource Prices Options Object**:

```json
{
  "id": "integer",
  "name": "string",
  "ram": "string",
  "cpu": "string",
  "disk": "string",
  "bandwidth": "string",
  "monthly": "string",
  "quarterly": "string",
  "semiannually": "string",
  "annually": "string",
  "biennially": "string",
  "triennially": "string",
  "mysort": "integer",
  "current": "integer",
  "promo": "object",
  "percent": "integer",
  "rid": "integer",
  "mtotal": "string",
  "cycles": "object",
  "firsts": "array",
  "resource": "string // or null"
}


```

| Spec | Detail |
| :---- | :---- |
| **Action** | `get_price_list` |
| **URL Params** | None |
| **Data Params** | **Required:** None. **Optional:** `serviceid` (Number), `drive_type` (String, Default: "ssd"), `managed` (Boolean, Default: 1), `disk` (Number). |
| **Success Response** | **Code:** 200. **Content:** `{ "success": {<resource_prices_options_object>, <resource_prices_options_object>...}}` |
| **Error Response** | **Code:** 200. **Content:** Standard Error Response. |
| **Notes** | Returns both **fixed plans** and **custom plans**. Custom plans include the nested pricing object, which defines granular options for components like RAM and CPU. |

**Use Cases**:

- Populate `vps_instances.approximate_cost` field

- Cost planning and budgeting

- Upgrade path analysis

- Pricing page generation

---

### get_billingcycle_prices (Get Billing Cycle Pricing)

| Spec | Detail |
| :---- | :---- |
| **Action** | `get_billingcycle_prices` |
| **URL Params** | None |
| **Data Params** | **Required:** None. **Optional:** `serviceid` (Number). |
| **Success Response** | **Code:** 200. **Content:** `{}` (Expected to contain an object mapping billing cycles to prices and due dates) |
| **Notes** | If `serviceid` is provided, the API returns prices specific to that instance's potential upgrade. If omitted, it likely returns general pricing list data. |

---

## Upgrade Workflow (Two-Step Process)

> [!IMPORTANT]
> **Two-Step Upgrade Process**: Hostwinds upgrades require two separate API calls:
>
> 1. `upgrade_instance` - Creates the upgrade order and invoice
> 2. `upgrade_server` - Applies the upgrade after invoice payment

### Step 1: upgrade_instance (Create Upgrade Order/Invoice)

| Spec | Detail |
| :---- | :---- |
| **Action** | `upgrade_instance` |
| **URL Params** | None |
| **Data Params** | **Required:** `serviceid` (Integer), `rid` (Number). **Optional:** `ram`, `cpu`, `disk`, `bandwidth` (Numbers, for Custom types). |
| **Success Response** | **Code:** 200. **Content:** `[ { "result": "success", "action": "Upgrade Instance", "message": "Service has been upgraded!" } ]` |
| **Error Response** | **Code:** 200. **Content:** `[ { "result": "error", "action": "Upgrade Instance", "message": "Hourly products are disabled! Please contact customer service." } ]` |
| **Notes** | **Creates the invoice. The upgrade is *not* applied to the server until the invoice is paid and `upgrade_server` is called.** |

**Parameters**:

- `rid` - Resource ID from `get_price_list`

- `ram`, `cpu`, `disk`, `bandwidth` - For custom plans only

---

### Step 2: upgrade_server (Apply Upgrade to Server)

| Spec | Detail |
| :---- | :---- |
| **Action** | `upgrade_server` |
| **URL Params** | None |
| **Data Params** | **Required:** `serviceid` (Integer). **Optional:** `disk` (Number), `upgrade_drive_type` (Boolean, Default: 0). |
| **Success Response** | **Code:** 200. **Content:** `[ { "result": "success", "action": "Upgrade Server", "message": "Server Server#12345 is being upgraded." } ]` |
| **Error Response** | **Code:** 200. **Content:** `[ { "result": "error", "action": "Upgrade Server", "message": "A valid serviceid is required.", "ERROR": "721334" } ]` |
| **Notes** | **Applies the change *after* the invoice is paid. Disk, RAM, and CPU changes may lead to a RESIZED status, requiring `confirm_upgrade` or `revert_upgrade`.** |

---

## Upgrade Validation and Confirmation

### check_upgrade_order (Check Existing Upgrade Order)

| Spec | Detail |
| :---- | :---- |
| **Action** | `check_upgrade_order` |
| **URL Params** | None |
| **Data Params** | **Required:** `serviceid` (Integer). |
| **Success Response** | **Code:** 200. **Content:** `{ "success": { "id": 98, "orderid": 550, "invoiceid": 416, "duedate": "2018-06-01", "total": "4.00" } }` |
| **Notes** | This check is critical before initiating a new upgrade, as **only one upgrade order is allowed at a time**. If successful, the response provides the details of the pending upgrade/invoice. |

---

### check_upgrade_reboot (Check if Upgrade Requires Reboot)

| Spec | Detail |
| :---- | :---- |
| **Action** | `check_upgrade_reboot` |
| **URL Params** | None |
| **Data Params** | **Required:** `serviceid` (Integer). |
| **Success Response** | **Code:** 200. **Content:** `{ "success" = true }` |
| **Notes** | This check is a point-in-time assessment, specifically covering **RAM and CPU changes**. A true response indicates a reboot will likely be required for the upgrade to take effect. |

---

### confirm_upgrade (Confirm Instance Upgrade)

| Spec | Detail |
| :---- | :---- |
| **Action** | `confirm_upgrade` |
| **URL Params** | None |
| **Data Params** | **Required:** `serviceid` (Integer). |
| **Success Response** | **Code:** 200. **Content:** `[ { "result": "success", "action": "Confirm Upgrade", "message": "Upgrade Confirmed!" } ]` |
| **Notes** | This action should be run when the instance status is **RESIZED**, typically following a successful `upgrade_server` call, especially if disk, RAM, or CPU changes required server migration. |

---

### revert_upgrade (Revert Post-Resize Upgrade)

| Spec | Detail |
| :---- | :---- |
| **Action** | `revert_upgrade` |
| **URL Params** | None |
| **Data Params** | **Required:** `serviceid` (Integer). |
| **Success Response** | **Code:** 200. **Content:** `[ { "result": "success", "action": "Revert Upgrade", "message": "Upgrade Reverted!" } ]` |
| **Error Response** | **Code:** 200. **Content:** `[ { "result": "error", "action": "Revert Upgrade", "message": "A valid serviceid is required." } ]` |
| **Notes** | **Must be run when the instance status is RESIZED to undo the changes made by `upgrade_server`.** |

---

## Billing Cycle Management

### change_billing_cycle (Change Subscription Period)

| Spec | Detail |
| :---- | :---- |
| **Action** | `change_billing_cycle` |
| **URL Params** | None |
| **Data Params** | **Required:** `serviceid` (Integer). **Optional:** `billingcycle` (String). |
| **Success Response** | **Code:** 200. **Content:** `[ { "result": "success", "action": "Change Billing Cycle", "message": "Service has been upgraded!" } ]` |
| **Error Response** | **Code:** 200. **Content:** `[ { "result": "error", "action": "Change Billing Cycle", "message": "Hourly products are disabled! Please contact customer service." } ]` |
| **Notes** | If `billingcycle` is omitted, the default value is "monthly". |

**Valid Billing Cycles**:

- `hourly` (if enabled)

- `monthly`

- `quarterly`

- `semiannually`

- `annually`

- `biennially`

- `triennially`

---

## Service Level Management

### upgrade_to_managed (Upgrade to Managed Service)

| Spec | Detail |
| :---- | :---- |
| **Action** | `upgrade_to_managed` |
| **URL Params** | None |
| **Data Params** | **Required:** `serviceid` (Integer). |
| **Success Response** | **Code:** 200. **Content:** `[ { "result": "success", "action": "Upgrade to Managed", "message": "Service has been upgraded!" } ]` |
| **Error Response** | **Code:** 200. **Content:** `[ { "result": "error", "action": "Upgrade to Managed", "message": "Hourly products are disabled! Please contact customer service." } ]` |
| **Notes** | **Changes the service level to fully managed.** |

---

### downgrade_to_unmanaged (Downgrade to Unmanaged)

| Spec | Detail |
| :---- | :---- |
| **Action** | `downgrade_to_unmanaged` |
| **URL Params** | None |
| **Data Params** | **Required:** `serviceid` (Integer). |
| **Success Response** | **Code:** 200. **Content:** `[ { "result": "success", "action": "Downgrade to Unmanaged", "message": "Service has been downgraded!" } ]` |
| **Error Response** | **Code:** 200. **Content:** Standard Error Response. |
| **Notes** | Changes the service management level, transferring operational responsibility (e.g., updates, security) back to the user/automation script. |

---

## Integration with PenguinMails

### Cost Modeling Workflow

**Baseline Configuration**:

- Default plan: Hostwinds Unmanaged Linux VPS 1 CPU / 2 GB / 50 GB / 2 TB at **$9.99/month**

- IP cost: **$4.99/month per dedicated IP**

**Automated Cost Tracking**:

1. Call `get_price_list` on instance creation

2. Normalize pricing to monthly equivalent

3. Populate `vps_instances.approximate_cost`

4. Reconcile monthly against Hostwinds invoices

5. Update on upgrades or billing cycle changes

**Database Integration**:

- `vps_instances.approximate_cost` - Populated from pricing APIs

- `smtp_ip_addresses.approximate_cost` - $4.99/month baseline

- Periodic reconciliation against actual invoices

See [Infrastructure Overview](/docs/business/implementation/infrastructure-overview) for complete cost modeling details.

---

## Upgrade Workflow Example

```javascript
// Step 1: Check for existing upgrade orders
const existingOrder = await checkUpgradeOrder(serviceid);
if (existingOrder.success) {
  console.log('Pending upgrade exists:', existingOrder);
  return;
}

// Step 2: Get available plans
const pricelist = await getPriceList({ serviceid, drive_type: 'ssd' });
const targetPlan = pricelist.find(p => p.ram === '4' && p.cpu === '2');

// Step 3: Create upgrade order
const order = await upgradeInstance({
  serviceid,
  rid: targetPlan.rid
});

// Step 4: Wait for invoice payment (external process)
await waitForInvoicePayment(order.invoiceid);

// Step 5: Check if reboot required
const needsReboot = await checkUpgradeReboot(serviceid);

// Step 6: Apply upgrade
const upgrade = await upgradeServer({ serviceid });

// Step 7: Poll for RESIZED status
await pollInstanceStatus(serviceid, 'RESIZED');

// Step 8: Confirm upgrade
await confirmUpgrade(serviceid);

// Step 9: Update database
await updateVPSCost(serviceid, targetPlan.monthly);


```

---

## Best Practices

### Upgrade Planning

- Always check for existing upgrade orders first

- Verify invoice payment before applying upgrades

- Check reboot requirements and schedule accordingly

- Test upgrade path in staging environment

### Cost Management

- Query pricing regularly to detect changes

- Normalize all pricing to monthly equivalent

- Reconcile modeled costs with actual invoices monthly

- Document pricing assumptions and constants

### Billing Optimization

- Analyze cost savings for annual vs monthly billing

- Consider managed vs unmanaged based on team capacity

- Monitor upgrade costs vs new instance costs

- Track total cost of ownership including labor

---

## Related Documentation

- [Hostwinds API Overview](/docs/implementation-technical/api/hostwinds/overview) - Main API overview

- [Hostwinds Server Management API](/docs/implementation-technical/api/hostwinds/server-management) - Instance operations

- [Infrastructure Overview](/docs/business/implementation/infrastructure-overview) - Cost modeling details

- [Database Migration Guide](/docs/implementation-technical/database-infrastructure/operations/database-migration-guide) - Cost field implementation

---

**Keywords**: Hostwinds upgrades, billing cycles, pricing API, cost management, resource scaling, service levels
