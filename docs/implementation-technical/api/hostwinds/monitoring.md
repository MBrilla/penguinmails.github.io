---
title: "Hostwinds Monitoring and Diagnostics API"
description: "Detailed API contract for Hostwinds server monitoring, logs, and diagnostic tools"
last_modified_date: "2025-11-19"
level: "3"
persona: "Documentation Users"
---

# Hostwinds Monitoring and Diagnostics API

**For DevOps Engineers and SRE Teams**: Complete API reference for Hostwinds server monitoring, log access, and diagnostic operations

**Parent Documentation**: [Hostwinds API Overview](/docs/implementation-technical/api/hostwinds/overview)

---

## Serial Console and Logs

### get_serial / get_vnc (Get Server Logs)

**Data Array Structure**:

```json
[
  "0": "string", // Status
  "1": "string", // Timestamp
  "2": "string", // Method
  "3": "string", // Action/Path
  "4": "string"  // Error Message
]


```

**Log Container Object**:

```json
{
  "draw": "integer",
  "recordsTotal": "integer",
  "recordsFiltered": "integer",
  "data": ["data_array", "data_array..."]
}


```

| Spec | Detail |
| :---- | :---- |
| **Action** | `get_serial` |
| **URL Params** | None |
| **Data Params** | **Required:** `serviceid` (Integer), **DataTables Parameters** (e.g., draw, start, length, columns). |
| **Notes on Data Params** | **The API expects DataTables request data** to handle pagination/filtering based on the 5 requested columns: **Status, Timestamp, Method, Action, and Error.** |
| **Success Response** | **Code:** 200. **Content:** `{"success": {<log_entry_object>}}` |

**Use Cases**:

- Debugging boot failures

- Investigating system crashes

- Monitoring system messages

- Troubleshooting network issues

**DataTables Integration**:
This endpoint is designed to work with jQuery DataTables for pagination and filtering. Example parameters:

- `draw` - Draw counter for DataTables

- `start` - Starting record index

- `length` - Number of records to return

- `columns` - Column definitions for filtering

---

## Performance Monitoring

### get_server_charts (Get Server Performance Data)

| Spec | Detail |
| :---- | :---- |
| **Action** | `get_server_charts` |
| **URL Params** | None |
| **Data Params** | **Required:** `serviceid`, `period` (String) |
| **Success Response** | No response is listed in the documentation |

**Period Options** (typical):

- `1h` - Last hour

- `24h` - Last 24 hours

- `7d` - Last 7 days

- `30d` - Last 30 days

**Expected Metrics** (based on typical VPS monitoring):

- CPU utilization

- Memory usage

- Disk I/O

- Network bandwidth

- Disk space usage

> [!NOTE]
> Response format not documented in official API. Test in development environment to determine structure.

---

## Service Validation

### check_serviceid (Validate Service ID)

| Spec | Detail |
| :---- | :---- |
| **Action** | `check_serviceid` |
| **URL Params** | None |
| **Data Params** | **Required:** `serviceid` (Integer). |
| **Success Response** | **Code:** 200. **Content:** `{ "success": true }` |
| **Notes** | This function is a simple boolean check. Automation scripts should check for the presence of `"success": true` in the response body. |

**Use Cases**:

- Validate service ID before operations

- Check if instance still exists

- Verify API access to specific instance

- Pre-flight checks in automation scripts

---

## SMTP Monitoring

### get_smtp_filters (Get SMTP Filter Configuration)

| Spec | Detail |
| :---- | :---- |
| **Action** | `get_smtp_filters` |
| **URL Params** | None |
| **Data Params** | **Required:** None. **Optional:** `serviceid` (Integer). |
| **Success Response** | **Code:** 200. **Content:** `{ success: {spam: [ { type: string,created: string, invoice_id: integer} ]}}` |

**Use Cases**:

- Monitor spam filtering status

- Audit SMTP security configuration

- Track filter deployment dates

- Verify billing for premium filters

---

## Maintenance and Events

### get_instance_maintenance (Get Maintenance Events)

| Spec | Detail |
| :---- | :---- |
| **Action** | `get_instance_maintenance` |
| **URL Params** | None |
| **Data Params** | **Required:** `serviceid` (Integer). |
| **Success Response** | **Code:** 200. **Content:** `[ { "result": "success", "action": "Get Instance Maintenance", "message": "Snapshot is being taken." } ]` |
| **Notes** | This action retrieves information about **in-progress or scheduled maintenance events** for the specific instance. The message field provides the status or type of event. |

**Event Types** (typical):

- Snapshots in progress

- Scheduled maintenance windows

- System updates

- Hardware migrations

- Network maintenance

**Integration Points**:

- Alert systems for maintenance windows

- Automated backup verification

- Capacity planning for migrations

- Incident response coordination

---

## Monitoring Integration with PenguinMails

### Recommended Monitoring Strategy

**1. Health Checks**:

```javascript
// Every 5 minutes
async function healthCheck(serviceid) {
  const isValid = await checkServiceId(serviceid);
  if (!isValid) {
    alert('Instance no longer accessible');
    return;
  }

  const instance = await getInstance(serviceid);
  if (instance.status !== 'active') {
    alert(`Instance status: ${instance.status}`);
  }
}


```

**2. Performance Monitoring**:

```javascript
// Every hour
async function collectMetrics(serviceid) {
  const charts = await getServerCharts(serviceid, '1h');
  await storeMetrics(serviceid, charts);

  // Alert on thresholds
  if (charts.cpu > 80) {
    alert('High CPU usage');
  }
}


```

**3. Maintenance Tracking**:

```javascript
// Every 15 minutes
async function checkMaintenance(serviceid) {
  const maintenance = await getInstanceMaintenance(serviceid);
  if (maintenance.message) {
    await logMaintenanceEvent(serviceid, maintenance);
  }
}


```

**4. Log Analysis**:

```javascript
// On-demand or scheduled
async function analyzeLogs(serviceid) {
  const logs = await getSerial(serviceid, {
    draw: 1,
    start: 0,
    length: 100
  });

  // Parse for errors
  const errors = logs.data.filter(entry =>
    entry[0] === 'error' || entry[4]
  );

  if (errors.length > 0) {
    await alertErrors(serviceid, errors);
  }
}


```

---

## Alerting and Incident Response

### Critical Alerts

**Immediate Response Required**:

- Instance status changes to `error` or `suspended`

- Service ID validation fails

- Unexpected maintenance events

- High error rate in serial logs

**Warning Alerts**:

- CPU/Memory above 80% for >15 minutes

- Disk space above 85%

- Network bandwidth approaching limits

- Scheduled maintenance in next 24 hours

### Integration with Monitoring Tools

**Prometheus/Grafana (Future/2026 Spike)**:

```yaml


# Example Prometheus scrape config

scrape_configs:


  - job_name: 'hostwinds_instances'
    scrape_interval: 5m
    static_configs:


      - targets: ['monitoring-api:9090']
    metrics_path: '/metrics/hostwinds'


```

**PagerDuty/Opsgenie**:

- Alert on instance status changes

- Escalate on repeated health check failures

- Notify on scheduled maintenance

- Page on critical errors in logs

---

## Best Practices

### Monitoring Frequency

- **Health checks**: Every 5 minutes

- **Performance metrics**: Every 15-60 minutes

- **Log analysis**: Hourly or on-demand

- **Maintenance checks**: Every 15 minutes

### Data Retention

- **Real-time metrics**: 24 hours

- **Hourly aggregates**: 30 days

- **Daily aggregates**: 1 year

- **Critical logs**: Indefinite

### Alert Thresholds

- **CPU**: Warning at 80%, Critical at 95%

- **Memory**: Warning at 85%, Critical at 95%

- **Disk**: Warning at 80%, Critical at 90%

- **Network**: Warning at 80% of limit

### Performance Optimization

- Cache `check_serviceid` results (5 minutes)

- Batch log queries when possible

- Use appropriate time windows for charts

- Implement exponential backoff for retries

---

## Troubleshooting Guide

### Common Issues

**Serial Console Not Responding**:

1. Verify service ID is valid

2. Check instance status

3. Try VNC console as alternative

4. Contact Hostwinds support if persistent

**Missing Performance Data**:

1. Verify `period` parameter is valid

2. Check if instance was recently created

3. Ensure instance is running

4. Try different time periods

**Maintenance Events Not Showing**:

1. Verify no maintenance is scheduled

2. Check if instance is in maintenance mode

3. Poll more frequently during known maintenance windows

---

## Related Documentation

- [Hostwinds API Overview](/docs/implementation-technical/api/hostwinds/overview) - Main API overview

- [Hostwinds Server Management API](/docs/implementation-technical/api/hostwinds/server-management) - Instance operations

- [Hostwinds Automation Best Practices](/docs/implementation-technical/api/hostwinds/automation-best-practices) - Error handling and workflows

- [Architecture Overview](/docs/implementation-technical/architecture-system/architecture-overview) - System monitoring integration

---

**Keywords**: Hostwinds monitoring, server diagnostics, performance metrics, log analysis, health checks, maintenance tracking
