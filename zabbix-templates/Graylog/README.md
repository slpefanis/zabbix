# Graylog Platform by HTTP

## Overview

This template monitors Graylog 7.x using the Graylog REST API.

It provides automatic discovery and monitoring of Graylog application nodes, OpenSearch Data Nodes, inputs and Sidecars.

Metrics are collected centrally using Zabbix HTTP agent items, with dependent items for efficient data collection.

Includes platform health, ingestion, storage and Sidecar diagnostic dashboards.

## Requirements

- Zabbix 7.4 or higher
- Graylog 7.x
- HTTPS connectivity to the Graylog REST API
- Graylog service account with API token
- Appropriate Graylog API permissions

No Zabbix agent or additional scripts are required for API monitoring.

## Tested versions

This template has been tested with:

- Zabbix 7.4
- Graylog 7.1.11

## Configuration

Import the template into Zabbix and link it to a host representing the Graylog environment.

Configure the required macros as described below.

## Setup

### Graylog service account

1. Navigate to **System → Users and Teams → Users**.
2. Create a service account, for example `zabbix-monitor`.
3. Generate an API token for the account.

### Required roles

Assign the following roles:

| Role | Description |
|---|---|
| Reader | Basic Graylog API access |
| Sidecar Reader | Sidecar inventory and collector monitoring |
| Zabbix Data Node Monitoring | Custom role for OpenSearch API access |

### Custom Data Node role

Create a custom role through the Graylog REST API.

**Endpoint:** `POST /api/roles`

**Request body:**

```json
{
  "name": "Zabbix Data Node Monitoring",
  "description": "Read-only OpenSearch monitoring through the Graylog API",
  "permissions": [
    "datanode:opensearchproxy"
  ],
  "read_only": false
}
```


### Create the Data Node Monitoring Role

Run the following command using a Graylog administrator account.

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

**Note:** Replace `graylog.example.com` with your Graylog server address. The command prompts for the administrator password.

**Expected result:** `HTTP 201 Created`.


Authenticate with a Graylog administrator account when creating the role.

Assign the new role to the monitoring service account.

### Zabbix configuration

1. Import the template.
2. Create a host representing the Graylog platform.
3. Link the **Graylog Platform by HTTP** template.
4. Configure the macros.
5. Verify data collection and discovery.

## Macros used

| Name | Description | Default |
|---|---|---|
| `{$GRAYLOG.API.URL}` | Graylog REST API base URL | `https://graylog.example.com/api` |
| `{$GRAYLOG.API.TOKEN}` | Graylog service account API token | None |
| `{$GRAYLOG.DATANODE.PROXY}` | Data Node hostname used for OpenSearch API proxy requests | `graylog-datanode.example.com` |

Store the API token as a Zabbix **Secret text** macro.

## Discovery

The template automatically discovers:

- Graylog application nodes
- OpenSearch Data Nodes
- Graylog input instances
- Graylog Sidecars

## Dashboards

- **Platform Overview** — Cluster health, throughput and overall status.
- **Ingestion & Storage** — Message processing, index health and storage.
- **Sidecar Diagnostics** — Sidecar availability, heartbeat age and collector failures.

## Notes

- Monitoring is performed through a single Graylog API endpoint.
- OpenSearch monitoring uses Graylog's Data Node API proxy.
- Some OpenSearch API endpoints are restricted by Graylog.
- Valid HTTPS certificate verification is recommended.
- Operating-system, MongoDB and load-balancer monitoring require separate templates.

## Feedback

Please report issues or suggest improvements through GitHub Issues.
