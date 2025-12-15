---
title: "Hostwinds Networking and IP Management API"
description: "Detailed API contract for Hostwinds IP address management, networking configuration, and firewall setup"
last_modified_date: "2025-11-19"
level: "3"
persona: "Documentation Users"
---

# Hostwinds Networking and IP Management API

**For Platform Engineers and Network Administrators**: Complete API reference for Hostwinds IP management, DNS configuration, and network operations

**Parent Documentation**: [Hostwinds API Overview](/docs/implementation-technical/api/hostwinds/overview)

---

## IP Address Management

### get_instance_ips (Retrieve Server IPs)

| Spec | Detail |
| :---- | :---- |
| **Action** | `get_instance_ips` |
| **URL Params** | None |
| **Data Params** | **Required:** `serviceid`, **Optional:** `location_id` |
| **Success Response** | **Code:** 200. **Content:** `[ { <instance_object>, <instance_object>, ......]` |

**Notes**:

- Returns all IP addresses associated with an instance

- Includes both primary and secondary IPs

- Use `location_id` to filter by data center location

---

### change_main_ip (Change Main IP of Instance)

| Spec | Detail |
| :---- | :---- |
| **Action** | `change_main_ip` |
| **URL Params** | None |
| **Data Params** | **Required:** `serviceid` (Integer) |
| **Success Response** | **Code:** 200. **Content:** `[ { "result": "success", "action": "Change Main IP", "message": "Main IP address changed successfully!" } ]` |

**Notes**:

- Automatically assigns a new primary IP from available pool

- Old IP may be released or converted to secondary

- May require network configuration regeneration

---

### set_main_ip (Set Specific IP as Primary)

| Spec | Detail |
| :---- | :---- |
| **Action** | `set_main_ip` |
| **URL Params** | None |
| **Data Params** | **Required:** `serviceid`, `ip` (String) |
| **Success Response** | **Code:** 200. **Content:** `[ { "result": "success", "message": "Main IP set" } ]` |

**Notes**:

- Sets a specific IP address as the primary IP

- The IP must already be assigned to the instance

- Use this to promote a secondary IP to primary

---

### delete_ip (Remove Secondary IP)

| Spec | Detail |
| :---- | :---- |
| **Action** | `delete_ip` |
| **URL Params** | None |
| **Data Params** | **Required:** `serviceid`, `ip` |
| **Success Response** | **Code:** 200. **Content:** `[ { "result": "success", "action": "Delete IP", "message": "IP removed" } ]` |

> [!WARNING]
> Cannot delete the primary IP address. Use `set_main_ip` to change the primary IP first if needed.

**Notes**:

- Removes a secondary IP from the instance

- IP is returned to the available pool

- Update DNS records before removing IPs used for email

---

## Network Configuration

### regenerate_networking (Rebuild and Reboot Network)

| Spec | Detail |
| :---- | :---- |
| **Action** | `regenerate_networking` |
| **URL Params** | None |
| **Data Params** | **Required:** `serviceid` (Integer). |
| **Success Response** | **Code:** 200. **Content:** `[ { "result": "success", "action": "Regenerate Networking", "message": "Regenerating Network for Server#12345" } ]` |
| **Error Response** | **Code:** 200. **Content:** Standard Error Response. |
| **Notes** | This action initiates a **network rebuild**, which is followed by a **server reboot**. The success message indicates an **asynchronous task** has started, requiring status polling. |

**Use Cases**:

- Network connectivity issues

- After IP address changes

- DNS or routing problems

- Firewall configuration issues

> [!CAUTION]
> This operation **reboots the server**. Schedule during maintenance windows.

---

## DNS Configuration

### set_rdns (Set Reverse DNS)

| Spec | Detail |
| :---- | :---- |
| **Action** | `set_rdns` |
| **URL Params** | None |
| **Data Params** | **Required:** `serviceid` (Integer), `ip` (String), `rdns` (String). **Deprecated:** `rdata` (String). |
| **Success Response** | **Code:** 200. **Content:** `[ { "result": "success", "action": "Set rDNS", "message": "rDNS record updated." } ]` |
| **Error Response** | **Code:** 200. **Content:** `[ { "result": "error", "action": "Set rDNS", "message": "A valid serviceid is required." } ]` |

**Notes**:

- Critical for email deliverability

- rDNS should match the hostname in SMTP HELO/EHLO

- Format: `mail.example.com` (not `example.com`)

- Changes may take up to 24-48 hours to propagate

**Best Practices**:

- Set rDNS before sending email

- Match rDNS to SPF and DKIM records

- Use consistent naming across all IPs

---

## Firewall Management

### get_instance_firewall_profile (Get Firewall Configuration)

**Firewall Profile Object**:

```json
{
    "id": "string",
    "name": "string",
    "description": "string",
    "security_group_rules": "array"
}


```

**Security Group Rule Object**:

```json
{
  "direction": "string",
  "ethertype": "string",
  "protocol": "string, // or null",
  "port_range_min": "integer, // or null",
  "port_range_max": "integer, // or null",
  "description": "string"
}


```

| Spec | Detail |
| :---- | :---- |
| **Action** | `get_instance_firewall_profile` |
| **URL Params** | None |
| **Data Params** | **Required:** `serviceid` (Integer). |
| **Success Response** | **Code:** 200. **Content:** `{success: <firewall_profile_object>}` |
| **Notes** | **This action retrieves the detailed network security rules (ingress/egress, protocols, etc.) applied to the instance.** |

---

### set_instance_firewall (Set Firewall Profile)

| Spec | Detail |
| :---- | :---- |
| **Action** | `set_instance_firewall` |
| **URL Params** | None |
| **Data Params** | **Required:** `serviceid` (Integer), `name` (String, name of profile from `get_firewall_profiles`). |
| **Success Response** | **Code:** 200. **Content:** `[ { "result": "success", "action": "Set Instance Firewall Profile", "message": "Instances firewall profile has been updated." } ]` |
| **Error Response** | **Code:** 200. **Content:** `[ { "result": "error", "action": "Set Instance Firewall Profile", "message": "A valid serviceid is required." } ]` |

**Notes**:

- Profile name must exist in available firewall profiles

- Use `get_firewall_profiles` to list available profiles

- Changes apply immediately but may require connection reset

**Common Profiles**:

- `default` - Standard web/SSH access

- `smtp` - Email server (ports 25, 465, 587, 993, 995)

- `web` - HTTP/HTTPS only

- `custom` - User-defined rules

---

## SMTP-Specific Configuration

### get_smtp_filters (Get SMTP Filters)

| Spec | Detail |
| :---- | :---- |
| **Action** | `get_smtp_filters` |
| **URL Params** | None |
| **Data Params** | **Required:** None. **Optional:** `serviceid` (Integer). |
| **Success Response** | **Code:** 200. **Content:** `{ success: {spam: [ { type: string,created: string, invoice_id: integer} ]}}` |

**Notes**:

- Returns spam filtering configuration

- Shows active filters and their creation dates

- May include billing information for premium filters

---

## IP Cleanup and Billing

### create_ip_cleanup_fee (Create Invoice for IP Cleanup Fee)

| Spec | Detail |
| :---- | :---- |
| **Action** | `create_ip_cleanup_fee` |
| **URL Params** | None |
| **Data Params** | **Required:** `serviceid` (Integer). |
| **Success Response** | **Code:** 200. **Content:** `[ { "result": "success", "action": "Create IP Cleanup Fee", "message": "A valid serviceid is required." } ]` |
| **Notes** | This action initiates the **billing process** by creating an invoice for the fee. The specific message "A valid serviceid is required." appears to be a generic placeholder or copy-paste error in the API documentation, but the result: "success" indicates the API call to create the fee was accepted. |

**Use Cases**:

- IP reputation cleanup services

- Spam list removal requests

- IP warming services

---

## Integration with PenguinMails

### Database Schema Integration

The networking API integrates with PenguinMails database schema:

- **`smtp_ip_addresses` table**:

  - `ip_address` - Populated from `get_instance_ips`

  - `rdns_hostname` - Set via `set_rdns`

  - `approximate_cost` - $4.99/month baseline (configurable)

- **`vps_instances` table**:

  - `main_ip` - Tracked from instance object

  - `private_ip` - Internal networking

  - `ips` - Array of all assigned IPs

### Automation Workflows

**IP Allocation Workflow**:

1. Allocate new IP via Hostwinds portal/API

2. Call `get_instance_ips` to retrieve new IP

3. Call `set_rdns` to configure reverse DNS

4. Update `smtp_ip_addresses` table

5. Configure firewall via `set_instance_firewall`

6. Begin IP warmup process

**IP Removal Workflow**:

1. Verify IP is not primary via `get_instance_ips`

2. Update DNS records to remove IP

3. Call `delete_ip` to release IP

4. Update `smtp_ip_addresses` table status

5. Reconcile billing

---

## Best Practices

### IP Management

- Always verify IP assignments before deletion

- Set rDNS immediately after IP allocation

- Use consistent naming conventions for rDNS

- Document IP-to-tenant mappings

### Network Configuration

- Test firewall rules before applying to production

- Schedule network regeneration during maintenance windows

- Monitor connectivity after network changes

- Keep backup of working firewall configurations

### DNS Configuration

- Allow 24-48 hours for rDNS propagation

- Verify rDNS matches SMTP hostname

- Test email deliverability after rDNS changes

- Use DNS monitoring tools to verify changes

---

## Related Documentation

- [Hostwinds API Overview](/docs/implementation-technical/api/hostwinds/overview) - Main API overview

- [Hostwinds Server Management API](/docs/implementation-technical/api/hostwinds/server-management) - Instance operations

- [Hostwinds Automation Best Practices](/docs/implementation-technical/api/hostwinds/automation-best-practices) - Error handling and workflows

- [Infrastructure Overview](/docs/business/implementation/infrastructure-overview) - IP cost modeling

---

**Keywords**: Hostwinds networking, IP management, reverse DNS, firewall configuration, SMTP IPs, network automation
