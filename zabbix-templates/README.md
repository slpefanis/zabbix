# Graylog Platform by HTTP

Zabbix 7.4 template for monitoring Graylog 7.x through its REST API.

Provides automatic discovery of application nodes, Data Nodes, inputs and Sidecars, including monitoring dashboards and alerts.

## Requirements

- Zabbix 7.4+
- Graylog 7.x
- HTTPS access from the Zabbix server or proxy to the Graylog API
- Graylog service account with an API token

## Graylog Configuration

### 1. Create a Service Account

In Graylog, navigate to **System → Users and Teams → Users**.

Create a service account (e.g. `zabbix-monitor`) and generate an API token.

### 2. Configure Permissions

Assign the following roles to the service account:

| Role | Purpose |
|---|---|
| `Reader` | Basic Graylog API access |
| `Sidecar Reader` | Read Sidecar inventory, heartbeats and collector health |
| `Zabbix Data Node Monitoring` | Custom role for OpenSearch monitoring |

Create the custom role using the Graylog REST API:

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

Create the role using a Graylog administrator account, then assign it to the monitoring service account.

### 3. Configure Zabbix

Import the template and link it to a Zabbix host representing the Graylog environment.

Configure these macros:

| Macro | Value |
|---|---|
| `{$GRAYLOG.API.URL}` | `https://graylog.example.com/api` |
| `{$GRAYLOG.API.TOKEN}` | Service account API token (Secret text) |
| `{$GRAYLOG.DATANODE.PROXY}` | Hostname of a Graylog Data Node |

The Data Node hostname must be resolvable by Graylog's API proxy, not necessarily by the Zabbix server.

## Verification

After importing, confirm that:

- HTTP master items collect data successfully.
- Discovery rules identify application nodes, Data Nodes, inputs and Sidecars.
- The included dashboards display current metrics.

## Notes

- Uses HTTPS and standard Graylog API endpoints.
- No direct connection to individual Graylog or OpenSearch nodes is required.
- A valid TLS certificate chain is required when certificate verification is enabled.
- Monitoring credentials should have only the permissions listed above.
