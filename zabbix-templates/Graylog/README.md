# Graylog Platform by HTTP

## Overview

This template monitors **Graylog 7.x** through its REST API using a single Zabbix host.

Features include:

- Automatic discovery of Graylog application nodes, OpenSearch Data Nodes, inputs and Sidecars.
- Message throughput, journal utilisation and processing health.
- OpenSearch cluster health, index storage and Data Node performance.
- Sidecar heartbeat and collector failure monitoring.
- Three dashboards, including an HTML Sidecar diagnostics table.
- Low-noise alerting using configurable thresholds.

No Zabbix agent or additional scripts are required for API monitoring.

## Requirements

- Zabbix 7.4 or later
- Graylog 7.x
- HTTPS connectivity from the Zabbix server or proxy to the Graylog REST API
- Graylog service account with API token and required permissions

## Tested versions

| Component | Version |
|---|---|
| Zabbix | 7.4 |
| Graylog | 7.1.11 |
| Template | 1.6 |

## Setup

### 1. Create a Graylog service account

Navigate to **System → Users and Teams → Users**.

Create a service account, such as `zabbix-monitor`, and generate an API token.

Assign the following roles:

| Role | Purpose |
|---|---|
| `Reader` | Basic Graylog API access |
| `Sidecar Reader` | Sidecar inventory, heartbeat and collector health |
| `Zabbix Data Node Monitoring` | Custom role for OpenSearch API proxy access |

### 2. Create the custom Data Node role

Use an administrator account to create the required role through the Graylog REST API.

Replace `graylog.example.com` with your Graylog server address.

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

Expected response: `HTTP 201 Created`.

Assign the new role to the monitoring service account.

### 3. Configure Zabbix

1. Import `Graylog-Platform-v1.6.yaml`.
2. Create a host representing your Graylog environment.
3. Link the **Graylog Platform by HTTP** template.
4. Configure the required macros.

## Macros used

| Macro | Description | Example |
|---|---|---|
| `{$GRAYLOG.API.URL}` | Graylog REST API base URL | `https://graylog.example.com/api` |
| `{$GRAYLOG.API.TOKEN}` | Service account API token | Secret text |
| `{$GRAYLOG.DATANODE.PROXY}` | Data Node hostname for the OpenSearch proxy | `graylog-datanode.example.com` |

The Data Node hostname must identify a Data Node known to Graylog.

Store the API token as a **Secret text** macro.

## Dashboards

The template includes three dashboards:

| Dashboard | Description |
|---|---|
| Platform Overview | Application health, message throughput, storage and Sidecar summary |
| Ingestion & Storage | Message processing rates, index counts, storage and replica visibility |
| Sidecar Diagnostics | HTML inventory with all discovered Sidecars, heartbeat ages, collector status and errors |

The Sidecar table highlights offline and failing collectors, with problematic devices listed first.

## Notes

- Monitoring uses the central Graylog API without requiring direct connections to individual nodes.
- Data collection uses HTTP agent items, dependent items and low-level discovery.
- OpenSearch proxy operations are subject to Graylog's security restrictions.
- Raw ingestion volume in GB/day is not currently measured.
- Operating-system, MongoDB and load-balancer monitoring require separate Zabbix templates.
- Validate alert thresholds and notification behaviour for your environment.

## Feedback

Issues, suggestions and contributions are welcome through GitHub Issues.
