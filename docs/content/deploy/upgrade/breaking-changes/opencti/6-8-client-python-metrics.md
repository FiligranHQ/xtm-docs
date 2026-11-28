---
title: Metrics renamed in client-python
description: The Prometheus metrics of OpenCTI connectors renamed in client-python for OpenCTI 6.8, old name to new name, and the two variables that set their prefix.
doc_type: reference
products:
  - opencti
personas: [platform-engineer, developer]
owner: docs-platform
last_reviewed: 2026-10-31
migrated_from:
  - opencti:/deployment/breaking-changes/6.8-client-python-metrics/
---

With OpenCTI 6.8, the Prometheus metrics that connectors expose through the Python client library (client-python) follow the [Prometheus naming practices](https://prometheus.io/docs/practices/naming/). This page lists the renamed metrics, so you can update the dashboards and alerts that use them, for example in Grafana.

::diagram{id=breaking-opencti-6-8-client-python-metrics}

| | Version |
| --- | --- |
| Changed in | OpenCTI 6.8 |

## Renamed metrics

| Old name | New name |
| --- | --- |
| `bundle_send` | `bundles_sent_total` |
| `record_send` | `records_sent_total` |
| `run_count` | `runs_total` |
| `ping_api_count` | `ping_api_total` |
| `ping_api_error` | `ping_api_errors_total` |
| `error_count` | `errors_total` |
| `client_error_count` | `client_errors_total` |

The `state` metric keeps its name.

## Namespace and subsystem

Two new settings add a prefix to the metric names, as Prometheus builds them: `<namespace>_<subsystem>_<name>`. Both are empty by default.

| Environment variable | Configuration key |
| --- | --- |
| `CONNECTOR_METRICS_NAMESPACE` | `connector.metrics_namespace` |
| `CONNECTOR_METRICS_SUBSYSTEM` | `connector.metrics_subsystem` |

:::only{opencti=">=7.261127"}
## Identity of the connector

client-python also exposes an `identity_info` metric, with the same prefix as the others. Its value is always `1`: its labels carry the connector's identity. The other metrics of the connector carry no label of their own: join them with `identity_info` on the labels Prometheus adds to the target, such as `instance`. Join it on the connector ID with the metrics that name the connector, such as those of its RabbitMQ queues.

| Label | Value |
| --- | --- |
| `id` | The connector ID, `CONNECTOR_ID` |
| `name` | The connector name, `CONNECTOR_NAME` |
| `type` | The connector type, `CONNECTOR_TYPE`, such as `EXTERNAL_IMPORT` |
| `scope` | The connector scope, `CONNECTOR_SCOPE`, its values separated by commas |

It adds a metric and renames none: your dashboards keep working.
:::

## What to do

Replace the old names in your dashboards, alerts and recording rules. If you set a namespace or a subsystem, add the prefix they create.

## What's next

- [Observability metrics](/deploy/operate/metrics/), to collect the metrics of the stack.
- [Breaking changes](/deploy/upgrade/breaking-changes/), the other changes per version.
