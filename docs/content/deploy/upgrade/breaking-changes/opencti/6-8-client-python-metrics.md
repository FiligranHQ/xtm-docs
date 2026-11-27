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

## What to do

Replace the old names in your dashboards, alerts and recording rules. If you set a namespace or a subsystem, add the prefix they create.

## What's next

- [Observability metrics](/deploy/operate/metrics/), to collect the metrics of the stack.
- [Breaking changes](/deploy/upgrade/breaking-changes/), the other changes per version.
