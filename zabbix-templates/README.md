# Graylog Platform Monitoring for Zabbix

A community-maintained Zabbix monitoring template for **Graylog 7.x**, providing centralised monitoring of Graylog application servers, OpenSearch Data Nodes, inputs, Sidecars and overall platform health.

The template uses the **Graylog REST API** to collect metrics from a single endpoint, automatically discovering components without requiring individual monitoring configurations for each node.

## Features

- **Application Monitoring** — Graylog node availability, processing status, message throughput and journal utilisation.
- **OpenSearch Monitoring** — Cluster health, Data Node performance, JVM memory, disk utilisation, shard health and index storage.
- **Input Monitoring** — Automatic discovery and health monitoring of configured Graylog inputs across application nodes.
- **Sidecar Monitoring** — Automatic discovery of Sidecars, heartbeat monitoring, collector status and failure diagnostics.
- **Dashboards** — Platform overview, ingestion and storage metrics, and Sidecar diagnostics.
- **Alerting** — Sensible default triggers designed to highlight meaningful issues without unnecessary alert noise.

## Requirements

- Zabbix 7.4 or later (developed and tested with Zabbix 7.4).
- Graylog 7.x with REST API access.
- A dedicated Graylog service account and API token.
- Appropriate read permissions for Graylog cluster, Sidecar and Data Node monitoring.
- HTTPS connectivity from the Zabbix server or proxy to the Graylog API.

## How It Works

The template is assigned to a single logical Zabbix host representing the Graylog environment.

HTTP agent items retrieve metrics from Graylog's REST API. Dependent items and low-level discovery automatically generate monitoring for individual application nodes, Data Nodes, inputs and Sidecars.

This approach minimises API requests and simplifies ongoing maintenance.

## Installation

1. Import the latest YAML template into Zabbix.
2. Create a logical Zabbix host for the Graylog platform.
3. Link the **Graylog Platform by HTTP** template.
4. Configure the required host macros:

| Macro | Description |
|---|---|
| `{$GRAYLOG.API.URL}` | Graylog REST API base URL |
| `{$GRAYLOG.API.TOKEN}` | Graylog service account API token |
| `{$GRAYLOG.DATANODE.PROXY}` | Data Node hostname used for OpenSearch API proxy requests |

5. Verify HTTP item collection and automatic discovery.
6. Open the included dashboards.

Store API tokens securely using Zabbix secret macros.

## Dashboards

The template includes three dashboards:

- **Platform Overview** — Overall environment health, message throughput, storage and Sidecar status.
- **Ingestion & Storage** — Message processing rates, OpenSearch indices and storage consumption.
- **Sidecar Diagnostics** — Sidecar availability, heartbeat age, collector failures and diagnostic information.

## Design Principles

- One template for the entire Graylog environment.
- Automatic discovery wherever possible.
- Minimal API overhead using dependent items.
- Read-only monitoring access.
- Practical, low-noise alerting.
- No dependency on organisation-specific hostnames or infrastructure.
- Compatibility with existing Zabbix operating-system monitoring templates.

## Limitations

- Some OpenSearch API endpoints may be restricted by Graylog's API proxy.
- Message throughput is measured in messages per second; raw ingestion volume in bytes is not currently collected.
- Sidecar diagnostic views are subject to Zabbix dashboard widget limitations.
- The template does not replace operating-system, MongoDB or Nginx monitoring.

## Contributing

Contributions, bug reports and feature requests are welcome.

Please remove credentials, internal hostnames, IP addresses and other sensitive information before submitting issues or pull requests.

## Disclaimer

This is a community project and is not officially affiliated with or supported by Graylog or Zabbix.

Test templates and triggers in your own environment before deploying to production.
