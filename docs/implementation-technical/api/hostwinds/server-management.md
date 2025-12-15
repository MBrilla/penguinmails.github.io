---
title: "Hostwinds Server Instance Management API"
description: "Detailed API contract for Hostwinds VPS instance lifecycle management, operations, and maintenance"
last_modified_date: "2025-11-19"
level: "3"
persona: "Documentation Users"
---

# Hostwinds Server Management API

**For Platform Engineers and Automation Teams**: Complete API reference for Hostwinds server instance creation, management, and operations

**Parent Documentation**: [Hostwinds API Overview](/docs/implementation-technical/api/hostwinds/overview)

---

## Server Instance Object

The `Server_Instance_Object` is returned by most server management endpoints:

```json
{
  "serviceid": "integer",
  "userid": "integer",
  "location_id": "integer",
  "server_uuid": "string",
  "volume_uuid": "string, // or null",
  "password": "string",
  "port_uuid": "string",
  "main_ip": "string",
  "private_ip": "string",
  "ips": "array, // or null",
  "upgrade_available": "integer",
  "is_suspended": "integer",
  "configkey": "string",
  "ssh_keys": "array",
  "hourly": "integer",
  "bandwidth": "integer",
  "pswd_changed": "datetime(YYYY-MM-DD HH:MM:SS)",
  "srvrname": "string",
  "template_uuid": "string",
  "app_id": "integer, // or null",
  "locked": "integer",
  "hostname": "string",
  "product_id": "integer",
  "billingcycle": "string",
  "product_name": "string",
  "disk_type": "string",
  "upgrade_ram": "string",
  "upgrade_cpu": "string",
  "upgrade_disk": "string",
  "bandwidth_used": "integer",
  "mac_address": "string",
  "status": "string",
  "disp_status": "string",
  "image": "object",
  "ip_v6": "string",
  "flavor": "object",
  "vm_state": "string",
  "task_state": "string, // or null",
  "image_ref": "string",
  "ram": "integer",
  "cpu": "integer",
  "disk": "integer",
  "created_at": "datetime(YYYY-MM-DD HH:MM:SS)",
  "name": "string",
  "launched_at": "datetime(YYYY-MM-DD HH:MM:SS)",
  "updated_at": "datetime(YYYY-MM-DD HH:MM:SS)",
  "power_state": "integer",
  "host": "string"
}


```

---

## Instance Creation and Retrieval

### add_instance (Create Server)

| Spec | Detail |
| :---- | :---- |
| **Action** | `add_instance` |
| **URL Params** | None |
| **Data Params** | **Required:** `template`, `rid`, `location_id`.  **Optional:** `qty`, `keys`, `srvrname`, `billingcycle`, `cclass`, `ram`, `cpu`, `disk`, `bandwidth`. |
| **Success Response** | **Code:** 200. **Content:** `[ { "result": "success", "action": "Invoice Paid", ... }, { "result": "success", "action": "Add Instance", "message": "Server is being Created!" } ]` |

**Notes**:

- This is an asynchronous operation - the success response indicates the task was queued

- Poll `get_instance` to check creation status

- The response includes invoice payment confirmation

---

### get_instances (Retrieve All Servers)

| Spec | Detail |
| :---- | :---- |
| **Action** | `get_instances` |
| **URL Params** | None |
| **Data Params** | None |
| **Success Response** | **Code:** 200. **Content:** `[ { <Server_Instance Object> }, { <Server_Instance Object> }, ... ]` |

**Notes**:

- Returns all instances for the authenticated account

- Use this to validate `serviceid` before other operations

---

### get_instance (Retrieve Single Server)

| Spec | Detail |
| :---- | :---- |
| **Action** | `get_instance` |
| **URL Params** | None |
| **Data Params** | **Required:** `serviceid` (Integer) |
| **Success Response** | **Code:** 200. **Content:** `{ <Server_Instance Object> }` |

**Notes**:

- Use for polling instance status after async operations

- Check `status`, `vm_state`, and `task_state` fields

---

### get_instance_ids (Get List of Valid Instance IDs)

**Instance Object**:

```json
{
    "id": "integer",
    "hostname": "string",
    "srvrname": "string",
    "ips": "string",
    "dedicatedip": "string",
    "ip_v6": "string"
}


```

| Spec | Detail |
| :---- | :---- |
| **Action** | `get_instance_ids` |
| **URL Params** | None |
| **Data Params** | **Required:** None. **Optional:** `search` (String), `page` (String), `limit` (String), `location_id` (Number). |
| **Success Response** | **Code:** 200. **Content:** `[ { <instance_object>, <instance_object>, ......]` |
| **Notes** | This function retrieves a list of instances, similar to `get_instances`, but the primary keys are the instance IDs. It supports common querying features like **search, pagination, and location filtering**. |

---

## Instance Lifecycle Operations

### boot (Start Server)

| Spec | Detail |
| :---- | :---- |
| **Action** | `boot` |
| **URL Params** | None |
| **Data Params** | **Required:** `serviceid`. |
| **Success Response** | **Code:** 200. **Content:** `[ { "result": "success", "action": "Boot", "message": "Instance Server#12345 has been started!" } ]` |

---

### reboot (Reboot Server)

| Spec | Detail |
| :---- | :---- |
| **Action** | `reboot` |
| **URL Params** | None |
| **Data Params** | **Required:** `serviceid`. **Optional:** `time` (Integer). |
| **Success Response** | **Code:** 200. **Content:** `[ { "result": "success", "action": "Reboot", "message": "Instance rebooted" } ]` |

**Notes**:

- The `time` parameter allows scheduling the reboot

- Required after password changes to take effect

---

### shutdown (Shut Down Instance)

| Spec | Detail |
| :---- | :---- |
| **Action** | `shutdown` |
| **URL Params** | None |
| **Data Params** | **Required:** `serviceid` (Integer). |
| **Success Response** | **Code:** 200. **Content:** `[ { "result": "success", "action": "Shutdown", "message": "Instance Server#12345 has been shut down!" } ]` |
| **Error Response** | **Code:** 200. **Content:** `[ { "result": "error", "action": "Shutdown", "message": "A valid serviceid is required." } ]` |

---

### terminate (Permanently Delete Server)

| Spec | Detail |
| :---- | :---- |
| **Action** | `terminate` |
| **URL Params** | None |
| **Data Params** | **Required:** `serviceid` |
| **Success Response** | **Code:** 200. **Content:** `[ { "result": "success", "message": "Instance deleted" } ]` |

> [!CAUTION]
> This operation is **irreversible**. All data on the instance will be permanently deleted.

---

## Instance Maintenance and Recovery

### recreate (Recreate Instance from Scratch)

| Spec | Detail |
| :---- | :---- |
| **Action** | `recreate` |
| **URL Params** | None |
| **Data Params** | **Required:** `serviceid` (Integer). |
| **Success Response** | **Code:** 200. **Content:** `[ { "result": "success", "action": "Rercreate", "message": "Recreating Server#12345" } ]` |
| **Error Response** | **Code:** 200. **Content:** Standard Error Response. |
| **Notes** | This action effectively deletes and provisions the server again using its original specifications. **The instance's IP address may change** as a result of this operation. The success message indicates an **asynchronous task** has begun, requiring status polling. |

---

### reinstall_instance (Reinstall from Template)

| Spec | Detail |
| :---- | :---- |
| **Action** | `reinstall_instance` |
| **URL Params** | None |
| **Data Params** | **Required:** `serviceid` (Integer), `template` (String). |
| **Success Response** | **Code:** 200. **Content:** `[ { "result": "success", "action": "Reinstall Instance", "message": "Instance Server#183 is being rebuilt!" } ]` |
| **Notes** | The `template` field accepts complex IDs (`uuid`, `base_image_ref`). |

---

### repair_instance (Repair Instance)

| Spec | Detail |
| :---- | :---- |
| **Action** | `repair_instance` |
| **URL Params** | None |
| **Data Params** | **Required:** `serviceid` (Integer). |
| **Success Response** | **Code:** 200. **Content:** `[ { "result": "success", "action": "Repair Instance", "message": "Instance Server#12345 has been restarted!" } ]` |
| **Error Response** | **Code:** 200. **Content:** `[ { "result": "error", "action": "Repair Instance", "message": "A valid serviceid is required." } ]` |
| **Notes** | **The repair action results in a server restart.** |

---

### rescue (Enter Rescue Mode)

| Spec | Detail |
| :---- | :---- |
| **Action** | `rescue` |
| **URL Params** | None |
| **Data Params** | **Required:** `serviceid` (Integer). |
| **Success Response** | **Code:** 200. **Content:** `[ { "result": "success", "action": "Reboot in Rescue Mode", "message": "Server Server#1234 Rebooting." } ]` |
| **Error Response** | **Code:** 200. **Content:** `[ { "result": "error", "action": "Reboot in Rescue Mode", "message": "A valid serviceid is required." } ]` |
| **Notes** | **Reboots the server into a specialized repair/rescue environment.** |

---

### unrescue (Exit Rescue Mode)

| Spec | Detail |
| :---- | :---- |
| **Action** | `unrescue` |
| **URL Params** | None |
| **Data Params** | **Required:** `serviceid` (Integer). |
| **Success Response** | **Code:** 200. **Content:** `[ { "result": "success", "action": "Unrescue", "message": "Server Server#1234 Rebooting Noramal." } ]` |
| **Error Response** | **Code:** 200. **Content:** `[ { "result": "error", "action": "Unrescue", "message": "A valid serviceid is required." } ]` |
| **Notes** | **Takes the server out of rescue mode and performs a normal reboot.** |

---

## Instance Configuration

### edit_instance (Edit Instance Server Name)

| Spec | Detail |
| :---- | :---- |
| **Action** | `edit_instance` |
| **URL Params** | None |
| **Data Params** | **Required:** `serviceid` (Integer), `srvrname` (String). |
| **Success Response** | **Code:** 200. **Content:** `[ { "result": "success", "action": "Edit Instance", "message": "Instance server name has been updated" } ]` |
| **Error Response** | **Code:** 200. **Content:** Standard Error Response. |
| **Notes** | This action updates the user-facing name/label (`srvrname`) of the server, not the internal hostname. |

---

### change_password (Change Instance Password)

| Spec | Detail |
| :---- | :---- |
| **Action** | `change_password` |
| **URL Params** | None |
| **Data Params** | **Required:** `serviceid` (Integer), `password` (String). |
| **Success Response** | **Code:** 200. **Content:** `[ { "result": "success", "action": "Change Password", "message": "Server Server#12345 has new password!<br>Reboot required for changes to take effect!" } ]` |
| **Notes** | The server typically requires a **reboot** for the password change to take effect. |

---

### toggle_lock (Toggle Instance Lock)

| Spec | Detail |
| :---- | :---- |
| **Action** | `toggle_lock` |
| **URL Params** | None |
| **Data Params** | **Required:** `serviceid` (Integer). |
| **Success Response** | **Code:** 200. **Content:** `[ { "result": "success", "action": "Toggle Lock", "message": "Service has been locked." } ]` |
| **Error Response** | **Code:** 200. **Content:** `[ { "result": "error", "action": "Toggle Lock", "message": "Service Id is required.", "ERROR": "5529903" } ]` |

**Notes**:

- Locked instances cannot be modified or deleted

- Use to prevent accidental changes to production instances

---

## ISO and Special Installations

### finalize_iso (Finalize Instance Creation from ISO)

| Spec | Detail |
| :---- | :---- |
| **Action** | `finalize_iso` |
| **URL Params** | None |
| **Data Params** | **Required:** `serviceid` (Integer). |
| **Success Response** | **Code:** 200. **Content:** `[ { "result": "success", "action": "Finalize ISO", "message": "Instance Server#183 is being Finalized!" } ]` |
| **Error Response** | **Code:** 200. **Content:** Standard Error Response (e.g., if the instance's `template_uuid` field is not 'cd'). |
| **Notes** | This action must be run on an instance that was initially created using an **ISO**. The instance's template field must be 'cd'. The success message indicates an **asynchronous task** has begun. |

---

## Instance Monitoring

### get_instance_maintenance (Get Maintenance Events)

| Spec | Detail |
| :---- | :---- |
| **Action** | `get_instance_maintenance` |
| **URL Params** | None |
| **Data Params** | **Required:** `serviceid` (Integer). |
| **Success Response** | **Code:** 200. **Content:** `[ { "result": "success", "action": "Get Instance Maintenance", "message": "Snapshot is being taken." } ]` |
| **Notes** | This action retrieves information about **in-progress or scheduled maintenance events** for the specific instance. The message field provides the status or type of event. |

---

### check_serviceid (Validate Service ID)

| Spec | Detail |
| :---- | :---- |
| **Action** | `check_serviceid` |
| **URL Params** | None |
| **Data Params** | **Required:** `serviceid` (Integer). |
| **Success Response** | **Code:** 200. **Content:** `{ "success": true }` |
| **Notes** | This function is a simple boolean check. Automation scripts should check for the presence of `"success": true` in the response body. |

---

## Related Documentation

- [Hostwinds API Overview](/docs/implementation-technical/api/hostwinds/overview) - Main API overview

- [Hostwinds Networking API](/docs/implementation-technical/api/hostwinds/networking) - IP and network management

- [Hostwinds Upgrade & Billing API](/docs/implementation-technical/api/hostwinds/upgrade-billing ) - Resource upgrades and billing

- [Hostwinds Automation Best Practices](/docs/implementation-technical/api/hostwinds/automation-best-practices ) - Error handling and workflows

---

**Keywords**: Hostwinds server management, VPS lifecycle, instance operations, server provisioning, rescue mode, instance maintenance
