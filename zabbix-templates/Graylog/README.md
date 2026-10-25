# Graylog Platform by HTTP

Comprehensive monitoring of Graylog, OpenSearch Data Nodes, message ingestion and Graylog Sidecars using **Zabbix 7.4**.

This community-maintained template provides centralized monitoring of Graylog deployments through the REST API, with automatic discovery, actionable alerts, historical ingestion reporting and operational dashboards.

**Latest version:** v1.9.2  
**Tested with:** Zabbix 7.4 and Graylog 7.1.11  
**Status:** Import validated, live data collection operational, individual Sidecar offline alert confirmed

## Features

### Graylog application monitoring

- Automatic discovery of Graylog application nodes
- Incoming and outgoing message throughput
- Per-node journal utilization
- Graylog input discovery and monitoring
- Cluster-wide processing metrics

### OpenSearch and Data Node monitoring

- OpenSearch cluster health
- Data Node discovery and statistics
- Message index storage consumption
- Message index count
- Unhealthy indices
- Message indices without replicas
- Cluster storage and shard visibility

OpenSearch metrics are collected through Graylog's authenticated Data Node API proxy. Direct OpenSearch access is not required.

### Graylog Sidecar monitoring

- Automatic discovery of registered Sidecars
- Sidecar online/offline status
- Last check-in and heartbeat age
- Collector running, stopped and failed states
- Collector error messages
- Sidecar version and IP address
- Consolidated Sidecar diagnostics table
- Individual High-severity alerts for offline Sidecars

The diagnostics table prioritizes offline and failing Sidecars, displaying their status, address, version, last contact, collector state and reported errors.

### Individual Sidecar alerting

**Introduced in v1.9.2**

Each discovered Sidecar can generate its own Zabbix problem when it has been offline for more than five minutes.

| Condition | Behaviour |
|---|---|
| Sidecar offline for over 5 minutes | High-severity problem |
| Multiple Sidecars offline | Separate problems for each Sidecar |
| Sidecar comes back online | Automatic recovery when the trigger condition clears |
| Problem acknowledged | Acknowledgement and comments retained in Zabbix |
| Sidecar removed from Graylog | Discovered monitoring objects follow Zabbix's lost-resource cleanup policy |

Example problem:

`Graylog: Sidecar example-server-01 offline`

This approach supports operational accountability. An offline Sidecar remains a detectable problem even when the reason is known, such as an intentional server decommission.

Operators can acknowledge the alert and document the reason without immediately closing or suppressing the underlying problem.

**Acknowledgement is not resolution.** The problem remains active until its trigger condition clears or the discovered monitoring object is removed.

Individual offline alerts are separate from the template's aggregate collector-health monitoring.

### Ingestion and capacity reporting

Historical ingestion monitoring using Graylog's cumulative traffic counters.

- Incoming and processed bytes per second
- Rolling 1-hour ingestion volume
- Rolling 24-hour ingestion volume
- Rolling 30-day ingestion volume
- Projected 30-day ingestion based on the trailing 24 hours
- Incoming versus outgoing traffic comparisons
- Historical throughput and volume graphs
- Per-application-node traffic collection

These measurements support capacity planning, ingestion trending and future log-management licensing assessments.

Graylog traffic counters are not guaranteed to match another logging platform's billable ingestion volume.

## Requirements

| Component | Requirement |
|---|---|
| Zabbix | Tested on 7.4 |
| Graylog | Tested on 7.1.11 |
| Graylog API | HTTPS access from the Zabbix server or proxy |
| Authentication | Dedicated Graylog API token |
| Data Nodes | Graylog-managed OpenSearch Data Nodes for OpenSearch monitoring |
| Permissions | Reader, Sidecar Reader and custom Data Node monitoring role |

The template is intended for a single logical Zabbix host representing the complete Graylog deployment.

No custom scripts or external exporters are required for API-based monitoring.

Underlying operating systems, Nginx, MongoDB and other infrastructure components should be monitored separately.

## Installation

### 1. Create a monitoring account

In Graylog, navigate to:

**System → Users and Teams → Users**

Create a dedicated monitoring account, such as:

`zabbix-monitor`

Generate an API token and assign these roles:

| Role | Purpose |
|---|---|
| Reader | Graylog operational information |
| Sidecar Reader | Sidecar inventory and collector monitoring |
| Zabbix Data Node Monitoring | Access to the OpenSearch monitoring proxy |

The Data Node monitoring role is custom and must be created separately.

### 2. Create the Data Node monitoring role

Use an authorized Graylog administrator account:

```bash
curl -sS -i --max-time 15 \
  -u admin \
  -X POST \
  -H 'Content-Type: application/json' \
  -H 'X-Requested-By: Zabbix' \
  -d '{
    "name": "Zabbix Data Node Monitoring",
    "description": "Allows Zabbix to query Data Node OpenSearch metrics through the Graylog API",
    "permissions": [
      "datanode:opensearchproxy"
    ],
    "read_only": false
  }' \
  'https://graylog.example.com/api/roles'
```

Replace the example hostname with your Graylog endpoint.

The administrator password is requested interactively.

Assign the new role to the monitoring account.

**Security:** The proxy uses Graylog's allowed OpenSearch operations. Review access before assigning this permission, and avoid using administrator credentials for routine monitoring.

### 3. Import the template

1. Open **Data collection → Templates** in Zabbix.
2. Select **Import**.
3. Import `Graylog-Platform-v1.9.2.yaml`.
4. Create a logical host for the Graylog platform.
5. Link the **Graylog Platform by HTTP** template.
6. Configure the macros below.

### 4. Configure macros

| Macro | Description | Example |
|---|---|---|
| `{$GRAYLOG.API.URL}` | Graylog REST API URL | `https://graylog.example.com/api` |
| `{$GRAYLOG.API.TOKEN}` | Monitoring account API token | Secret text |
| `{$GRAYLOG.DATANODE.PROXY}` | Registered Graylog Data Node hostname | `graylog-datanode.example.com` |

Store the API token as a **Secret text** macro.

Do not commit tokens or other credentials to source control.

### 5. Verify collection

After linking the template:

1. Open **Monitoring → Latest data** for the Graylog platform host.
2. Confirm that application metrics are collecting.
3. Verify application node, Data Node and Sidecar discovery.
4. Check OpenSearch cluster health.
5. Confirm cumulative traffic counters are increasing.
6. Open the dashboard pages.
7. Review individual offline Sidecar problems under **Monitoring → Problems**.

Historical ingestion reporting requires collection time before rolling periods are fully populated.

## Dashboards

Four dashboard pages are included.

| Dashboard | Purpose |
|---|---|
| **Platform overview** | Overall Graylog operational health |
| **Ingestion and storage** | Processing throughput, index storage and index health |
| **Sidecar diagnostics** | Sidecar inventory, status and collector errors |
| **Ingestion & capacity** | Historical traffic volumes, trends and projections |

### Platform overview

Provides a high-level view of the Graylog deployment, focusing on operational health and important processing metrics.

### Ingestion and storage

Displays message throughput, index storage, unhealthy indices and replica coverage.

This is an operational summary rather than a replacement for Graylog's native index management interface.

### Sidecar diagnostics

A consolidated HTML diagnostics table containing:

- Sidecar name
- Status
- IP address
- Version
- Last seen
- Collector states
- Problem or error information

Offline and failing Sidecars appear first, making them easy to identify.

### Ingestion & capacity

Displays:

- Rolling 24-hour incoming volume
- Rolling 24-hour processed volume
- Rolling 30-day observed incoming volume
- Projected 30-day incoming volume
- Incoming and processed traffic rates
- Incoming and outgoing message throughput
- Historical volume graphs

The projected 30-day figure is an estimate derived from the trailing 24-hour period, not an actual monthly total.

## Understanding ingestion measurements

The template collects these Graylog counters:

```text
org.graylog2.traffic.input
org.graylog2.traffic.output
```

They are cumulative application-node counters.

Zabbix calculates differences between observations to derive throughput and historical volume.

### Important considerations

- **Counter resets:** Application restarts can reset cumulative counters. Negative differences are discarded.
- **Collection gaps:** Missed observations may cause ingestion volume to be undercounted.
- **Historical availability:** Totals cannot reconstruct traffic from before collection began.
- **Units:** Distinguish decimal GB from binary GiB when comparing reports.
- **Licensing:** Incoming traffic, processed traffic and indexed storage are different measurements.
- **Projections:** Estimated 30-day ingestion is not equivalent to observed 30-day consumption.

For long-term accuracy, compare Zabbix's accumulated traffic history against Graylog's native ingestion reporting over equivalent time windows.

## Alerting and accountability

The monitoring philosophy is to generate actionable problems without excessive alert noise.

Individual Sidecar offline events receive their own High-severity problem, allowing separate investigation and acknowledgement.

Collector failures can still be reported through aggregate health monitoring to avoid generating excessive alerts for multiple failures across the environment.

### Recommended operational workflow

1. Zabbix detects an offline Sidecar.
2. A High-severity problem is created for the affected Sidecar.
3. An operator investigates and acknowledges the problem.
4. The acknowledgement comment records the cause or accepted operational condition.
5. The problem remains visible until recovery or inventory cleanup.
6. Zabbix retains the problem and acknowledgement history according to its configuration.

Example acknowledgement comment:

`Server decommissioned. Awaiting removal from Graylog Sidecar inventory.`

This avoids silently excluding legitimate outages simply because they may have an acceptable explanation.

Automatic recovery, lost-resource cleanup and notification actions should be validated against each organization's Zabbix configuration.

## Troubleshooting

### No Graylog metrics

Check the API URL, token, connectivity, TLS certificate validation and assigned permissions.

### OpenSearch metrics unavailable

Confirm that the configured Data Node exists in Graylog and that the monitoring account has the `datanode:opensearchproxy` permission.

### Sidecars missing

Confirm that the monitoring account has Sidecar Reader access and that Sidecar discovery is working.

### Offline Sidecar not generating a problem

Check that:

- The Sidecar is discovered.
- Its heartbeat item is collecting data.
- The offline trigger prototype has been instantiated.
- The Sidecar has exceeded the configured offline threshold.
- The trigger and discovered monitoring objects are enabled.

### Decommissioned Sidecar remains visible

A Sidecar that is no longer reporting may remain in Graylog's inventory.

Acknowledging its Zabbix problem does not remove the Sidecar.

Remove obsolete Sidecar registrations through the appropriate Graylog administration process when cleanup is authorized.

### Ingestion totals appear too low

New items require time to accumulate history. Review collection gaps and application restarts before investigating discrepancies.

### Projection displays no data

Confirm that the underlying traffic counters and rolling 24-hour calculated items are collecting and supported by the available history.

## Limitations

- Does not replace operating-system or infrastructure monitoring.
- Does not manage OpenSearch configuration or replicas.
- Does not discover every individual OpenSearch index.
- OpenSearch visibility is limited to operations permitted through Graylog's Data Node API proxy.
- Historical ingestion cannot be reconstructed before Zabbix collection begins.
- Traffic measurements are not guaranteed to match external vendor licensing calculations.
- Notification actions, acknowledgements and recovery handling depend on Zabbix configuration.
- Long-term ingestion accuracy is still being evaluated.

## Version history

| Version | Changes |
|---|---|
| **1.9.2** | Individual High-severity offline Sidecar alerting with per-device problem identification |
| 1.9.1 | Corrected dashboard widget coordinate field types; import validated |
| 1.9 | Dashboard layout and presentation improvements |
| 1.8 | Ingestion & Capacity dashboard with historical graphs |
| 1.7 | Traffic byte counters and rolling-volume reporting |
| 1.6 | Stable platform monitoring and HTML Sidecar diagnostics |

## Contributions

Issues, feature requests and improvements are welcome through GitHub Issues.

When reporting a problem, include the Zabbix version, Graylog version, template version and relevant error messages.

Remove credentials, API tokens and environment-specific information before sharing logs or exports.

---

**Community-maintained template. Not an official Zabbix or Graylog integration.**
