# Graylog Platform by HTTP

Comprehensive monitoring of Graylog, OpenSearch Data Nodes, message ingestion, and Graylog Sidecars using the **Zabbix HTTP agent**.

This template provides centralized monitoring of a Graylog deployment through its REST API, with automatic discovery, operational alerts, historical ingestion reporting, and integrated dashboards.

Designed for environments running multiple Graylog application nodes and Graylog-managed OpenSearch Data Nodes.

**No custom scripts, external exporters, or Zabbix agents are required for API-based monitoring.**

## Features

### Graylog application monitoring

- Automatic discovery of Graylog application nodes.
- Incoming and outgoing message throughput.
- Per-node journal utilization and processing health.
- Automatic discovery of configured Graylog inputs.
- Per-input operational visibility.
- Cluster-wide aggregation of application metrics.

### OpenSearch and Data Node monitoring

- OpenSearch cluster health.
- Data Node discovery and operational statistics.
- Index storage consumption.
- Total message index count.
- Indices reporting unhealthy states.
- Message indices without replicas.
- Shard and storage visibility.

OpenSearch metrics are collected through Graylog's authenticated Data Node API proxy. Direct OpenSearch access is not required.

### Graylog Sidecar monitoring

- Automatic discovery of registered Sidecars.
- Sidecar heartbeat age and online/offline status.
- Collector status and failures.
- Sidecar version and host information.
- Consolidated collector health monitoring.
- Detailed HTML diagnostics table.

The Sidecar diagnostics table displays discovered Sidecars with their status, IP address, version, last contact time, collectors, and reported errors.

Problematic Sidecars appear first, allowing operators to identify failures without searching through individual hosts.

### Ingestion and capacity reporting

Historical ingestion monitoring using Graylog's cumulative traffic counters.

- Incoming and processed traffic in bytes per second.
- Rolling 1-hour ingestion volume.
- Rolling 24-hour ingestion volume.
- Rolling 30-day ingestion volume.
- Estimated 30-day ingestion based on the trailing 24-hour rate.
- Incoming and outgoing traffic comparisons.
- Historical throughput and volume graphs.
- Per-application-node traffic collection.

This provides a useful foundation for capacity planning, identifying ingestion trends, and estimating future log management licensing requirements.

**Important:** These metrics measure Graylog-reported traffic. They should not automatically be treated as the billable ingestion volume of another logging platform.

## Requirements

| Component | Requirement |
|---|---|
| Zabbix | 7.4 or later |
| Graylog | 7.x |
| Graylog API | HTTPS access from the Zabbix server or proxy |
| Authentication | Graylog service account with API token |
| Data Nodes | Graylog-managed OpenSearch Data Nodes for OpenSearch monitoring |
| Permissions | Reader, Sidecar Reader and custom Data Node monitoring role |

The Zabbix server or proxy must be able to reach the Graylog REST API endpoint.

A valid TLS certificate chain is recommended.

## Tested environment

| Component | Version |
|---|---|
| Zabbix | 7.4 |
| Graylog | 7.1.11 |
| OpenSearch | Graylog-managed Data Nodes |
| Template | 1.9.1 |

The core monitoring functionality has been tested against the versions above. Dashboard changes in v1.9.1 should be validated after import.

Compatibility with other versions of Graylog and Zabbix has not been fully established.

## Installation

### 1. Create a Graylog monitoring account

In the Graylog web interface, navigate to:

**System → Users and Teams → Users**

Create a dedicated monitoring account, for example:

`zabbix-monitor`

Generate an API token for this account.

Assign the following roles:

| Role | Purpose |
|---|---|
| `Reader` | Access to Graylog monitoring information |
| `Sidecar Reader` | Sidecar inventory and collector information |
| `Zabbix Data Node Monitoring` | OpenSearch monitoring through the Data Node proxy |

The third role is custom and must be created as described below.

### 2. Create the Data Node monitoring role

Graylog exposes selected OpenSearch operations through its authenticated Data Node API proxy.

A custom role is required to grant access to this endpoint.

Using a Graylog administrator account, execute:

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

Replace `graylog.example.com` with your Graylog hostname.

The `-u admin` option prompts for the administrator's password.

A successful creation returns HTTP `201 Created`.

Assign the new role to the monitoring service account.

**Security note:** The `datanode:opensearchproxy` permission provides access to Graylog's allowlisted OpenSearch proxy operations. Review its scope before assigning it and use a dedicated monitoring account.

### 3. Import the template

In Zabbix:

1. Navigate to **Data collection → Templates**.
2. Select **Import**.
3. Import the latest `Graylog-Platform-v*.yaml` file.
4. Create a host representing the complete Graylog platform.
5. Link the **Graylog Platform by HTTP** template.
6. Configure the required macros.

The template is intended to be linked to **one logical Zabbix host per Graylog deployment**, rather than a separate template instance for every Graylog application node.

Application nodes, Data Nodes, inputs, and Sidecars are discovered automatically.

### 4. Configure macros

| Macro | Description | Example |
|---|---|---|
| `{$GRAYLOG.API.URL}` | Graylog REST API base URL | `https://graylog.example.com/api` |
| `{$GRAYLOG.API.TOKEN}` | Monitoring account API token | Secret text |
| `{$GRAYLOG.DATANODE.PROXY}` | Data Node hostname used by the OpenSearch proxy | `graylog-datanode.example.com` |

The Data Node hostname must match a Data Node registered with Graylog.

Store the API token as a **Secret text** macro.

Do not commit API tokens or other credentials to source control.

### 5. Verify data collection

After linking the template and configuring its macros:

1. Open **Monitoring → Latest data** for the logical Graylog host.
2. Confirm that Graylog application metrics are being collected.
3. Check discovery of application nodes and Data Nodes.
4. Confirm that OpenSearch cluster health is available.
5. Verify Sidecar discovery and status reporting.
6. Check that cumulative traffic counters are increasing.
7. Open the template dashboards.

Newly imported templates require time to populate historical graphs and rolling ingestion totals.

## Dashboards

The template includes four dashboard pages.

| Dashboard | Purpose |
|---|---|
| **Platform overview** | High-level Graylog operational status |
| **Ingestion and storage** | Message throughput, index storage and index health |
| **Sidecar diagnostics** | Detailed Sidecar and collector troubleshooting |
| **Ingestion & capacity** | Historical traffic volume, trends and capacity planning |

### Platform overview

A high-level view of the Graylog environment, including cluster processing, operational health and key platform metrics.

Designed to help answer:

- Is Graylog operating normally?
- Are application nodes processing messages?
- Are there emerging processing or storage problems?

### Ingestion and storage

Provides operational metrics for message processing and OpenSearch storage, including:

- Incoming and outgoing message rates.
- Message index storage consumption.
- Message index count.
- Unhealthy indices.
- Indices without replicas.

This is intended as an operational summary rather than a replacement for Graylog's own index management interface.

### Sidecar diagnostics

A consolidated HTML table showing Sidecar information.

Includes:

- Sidecar name and address.
- Online/offline status.
- Sidecar version.
- Heartbeat information.
- Collector status.
- Errors reported by collectors.

Offline and failing collectors are prioritized for investigation.

### Ingestion & capacity

A dedicated page for understanding how much traffic Graylog receives and processes.

Includes:

- Rolling 24-hour incoming volume.
- Rolling 24-hour processed volume.
- Rolling 30-day incoming volume.
- Projected 30-day incoming volume.
- Incoming versus processed traffic rates.
- Incoming versus outgoing message rates.
- Historical rolling-volume graphs.

The projection is an estimate derived from the preceding 24 hours, not an observed 30-day total.

## Understanding ingestion volume

The template uses the following Graylog metrics:

```text
org.graylog2.traffic.input
org.graylog2.traffic.output
```

These are cumulative counters collected from Graylog application nodes.

Zabbix calculates traffic changes over time to derive throughput and historical volume.

### Measurement considerations

**Cumulative counters:** Counters are process-relative and can reset after Graylog application restarts.

**Counter resets:** Negative differences are discarded to prevent false negative ingestion values.

**Collection gaps:** If an application node is unavailable or monitoring stops, ingestion volume may be undercounted.

**Historical data:** Rolling totals only include data collected since the monitoring items began recording history.

**Units:** Zabbix may automatically scale byte values into KiB, MiB and GiB. Be mindful of the difference between decimal GB and binary GiB.

**Licensing:** Graylog input traffic, processed traffic, indexed storage, and another vendor's billable ingestion are not necessarily equivalent measurements.

For accurate long-term reporting, allow sufficient collection history to accumulate and periodically compare Zabbix totals with Graylog's native traffic reporting.

## Discovery and collection architecture

The template uses native Zabbix functionality:

- HTTP agent items for REST API collection.
- Dependent items for extracting metrics.
- Low-level discovery for dynamically managed components.
- Calculated items for traffic aggregation and reporting.
- JavaScript preprocessing where required.
- Native dashboard widgets and an HTML diagnostics summary.

The architecture reduces unnecessary API requests by reusing collected JSON responses for dependent metrics.

Monitoring is performed centrally through the Graylog API endpoint.

## Alerting philosophy

The template is designed to provide useful operational warnings without excessive alert noise.

The primary objective is to detect platform issues requiring intervention, rather than generating individual alerts for every transient condition.

Discovery and consolidated monitoring are used wherever practical.

Notification actions, alert thresholds and recovery behavior should be reviewed and tested before production notification rules are enabled.

## Limitations

- API monitoring does not replace operating-system or infrastructure monitoring.
- MongoDB, Nginx, disk hardware and underlying operating systems should be monitored with appropriate separate Zabbix templates.
- OpenSearch API requests are limited by Graylog's Data Node proxy permissions and allowed operations.
- Historical ingestion cannot be reconstructed from Zabbix counters before collection began.
- Graylog traffic volume is not guaranteed to match another platform's licensing calculations.
- The template does not perform index configuration or replica management.
- Individual OpenSearch index discovery is intentionally excluded to avoid unnecessary monitoring noise.

## Troubleshooting

### No Graylog metrics

Check the API URL, token, network connectivity and TLS certificate validation.

### OpenSearch metrics unavailable

Confirm that the selected Data Node exists and that the monitoring account has the `datanode:opensearchproxy` permission.

Graylog's proxy restricts which OpenSearch API operations can be accessed.

### Sidecar discovery missing

Confirm that the monitoring account has `Sidecar Reader` permissions.

### Ingestion totals appear too low

New counters require time to accumulate history.

Rolling 24-hour and 30-day totals will initially contain only the available collection period.

Also check for application restarts or monitoring gaps.

### Projection reports no data

Verify that the source traffic metrics and rolling 24-hour aggregation are supported and collecting successfully.

Do not treat a projection as valid until its source calculations have sufficient history.

### Template import fails

Confirm the Zabbix version and use the latest corrected template release.

Zabbix validates dashboard layout fields and widget definitions strictly. Template YAML syntax validation alone does not guarantee successful import.

## Version history

| Version | Changes |
|---|---|
| 1.6 | Stable platform monitoring and HTML Sidecar diagnostics |
| 1.7 | Cumulative ingestion counters and rolling-volume reporting |
| 1.8 | Ingestion & Capacity dashboard with historical graphs |
| 1.9 | Dashboard layout improvements |
| 1.9.1 | Dashboard widget coordinate format correction |

## Contributions

Issues, feature requests and improvements are welcome through GitHub Issues.

When reporting a problem, include:

- Zabbix version.
- Graylog version.
- Template version.
- Relevant error message.
- Whether the problem occurs during import, discovery or data collection.

Please remove credentials, tokens and environment-specific information before sharing logs or exports.

---

**Community-maintained integration.** Not an official Zabbix or Graylog template.
