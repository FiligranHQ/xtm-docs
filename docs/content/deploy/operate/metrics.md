---
title: Observability metrics
description: The Prometheus and OpenTelemetry metrics OpenCTI and OpenAEV expose for your own monitoring, how to turn them on in the XTM stack and what each one measures.
doc_type: reference
products:
  - opencti
  - openaev
personas: [platform-engineer]
owner: docs-platform
last_reviewed: 2026-10-31
migrated_from:
  - opencti:/deployment/telemetry/
---

OpenCTI and OpenAEV expose observability metrics for your own monitoring system, such as Prometheus or an OpenTelemetry (OTLP) collector. This page says how to turn them on in the XTM stack, where to scrape them and what the metrics measure. Both are off by default.

::diagram{id=deploy-operate-metrics}

:::note[Not usage telemetry]
These metrics stay in your infrastructure. The anonymous usage data the platforms send to Filigran is something else: see [Usage telemetry and privacy](/opencti/configure/telemetry/).
:::

## Turn the metrics on

The platform services of xtm-docker take their settings from `docker-compose.yml` and the files it includes, in `modules/`. To add a setting without editing these files, put it in `docker-compose.override.yml`: see [Change the stack without editing docker-compose.yml](/deploy/install/docker-compose/#change-the-stack-without-editing-docker-compose-yml).

::::tabs
:::tab[OpenCTI]
OpenCTI exports its metrics with OpenTelemetry, to a Prometheus endpoint, an OTLP collector or both.

| Key | Environment variable | Default | What it does |
| --- | --- | --- | --- |
| `app:telemetry:metrics:enabled` | `APP__TELEMETRY__METRICS__ENABLED` | `false` | Turns the metrics on |
| `app:telemetry:metrics:exporter_prometheus` | `APP__TELEMETRY__METRICS__EXPORTER_PROMETHEUS` | `14269` | Port of the Prometheus endpoint, served at `/metrics` |
| `app:telemetry:metrics:exporter_otlp` | `APP__TELEMETRY__METRICS__EXPORTER_OTLP` | empty | URL of an OTLP collector to push the metrics to every second. Empty: no push |

:::steps
1. Create `docker-compose.override.yml` in the `xtm-docker` folder, or add to it.

   ```yaml
   services:
     opencti:
       environment:
         - APP__TELEMETRY__METRICS__ENABLED=true
   ```

2. Recreate the platform.

   ```bash
   docker compose up -d opencti
   ```

   :::verify
   ```bash
   docker compose exec opencti wget -qO- http://localhost:14269/metrics | grep '^# TYPE opencti_'
   ```
   It lists the `opencti_` metrics.
   :::
:::

The port is not published on the host. Scrape `opencti:14269` from a Prometheus on the Docker network of the stack, or publish the port with a `ports` entry in the override file.

The OpenCTI workers have their own Prometheus endpoint: set `WORKER_TELEMETRY_ENABLED=true` on the `worker` service, and each worker serves its metrics on port `14270` (`WORKER_PROMETHEUS_TELEMETRY_PORT`).
:::
:::tab[OpenAEV]
OpenAEV serves its metrics on its API port, at `/actuator/prometheus`. The endpoint answers only a request that carries the metrics key as a bearer token, and with no key set, it refuses every request.

| Key | Environment variable | Default | What it does |
| --- | --- | --- | --- |
| `openaev.metrics.key` | `OPENAEV_METRICS_KEY` | empty | The key Prometheus sends as `Authorization: Bearer <key>`. Empty: the endpoint answers 401 |

:::steps
1. Generate a key and add it to `.env`.

   ```bash
   echo "OPENAEV_METRICS_KEY=$(openssl rand -hex 32)" >> .env
   ```

2. Pass it to the platform in `docker-compose.override.yml`.

   ```yaml
   services:
     openaev:
       environment:
         - OPENAEV_METRICS_KEY=${OPENAEV_METRICS_KEY}
   ```

3. Recreate the platform.

   ```bash
   docker compose up -d openaev
   ```

   :::verify
   ```bash
   set -a; . ./.env; set +a
   curl -fsS -H "Authorization: Bearer $OPENAEV_METRICS_KEY" "http://localhost:8081/actuator/prometheus" | grep '^# TYPE openaev_'
   ```
   It lists the `openaev_` metrics.
   :::
:::

In a Prometheus scrape configuration, give the key with `authorization: {credentials: <key>}`.
:::
::::

## OpenCTI traces and profiles

Beyond metrics, OpenCTI can send traces and continuous profiles, to follow a slow request through the platform. Both are off by default and cost some CPU: turn them on to investigate, or on one node.

### Traces

With tracing on, OpenCTI sends an OpenTelemetry span for each GraphQL operation and for its calls to the databases and to Python, under the service name `opencti-platform`, with W3C trace context propagation. It sends them to an OTLP collector over HTTP, to Zipkin, or to both. Without an exporter URL, it sends nothing.

| Key | Environment variable | Default | What it does |
| --- | --- | --- | --- |
| `app:telemetry:tracing:enabled` | `APP__TELEMETRY__TRACING__ENABLED` | `false` | Turns tracing on |
| `app:telemetry:tracing:exporter_otlp` | `APP__TELEMETRY__TRACING__EXPORTER_OTLP` | empty | Full URL of the OTLP HTTP traces endpoint, such as `http://otel-collector:4318/v1/traces` |
| `app:telemetry:tracing:exporter_zipkin` | `APP__TELEMETRY__TRACING__EXPORTER_ZIPKIN` | empty | URL of the Zipkin endpoint, such as `http://zipkin:9411/api/v2/spans` |

### Profiles

OpenCTI can send continuous profiling data to a Pyroscope server, to see which functions use the CPU and the memory of the platform.

| Key | Environment variable | Default | What it does |
| --- | --- | --- | --- |
| `app:telemetry:pyroscope:enabled` | `APP__TELEMETRY__PYROSCOPE__ENABLED` | `false` | Turns profiling on, at start |
| `app:telemetry:pyroscope:exporter` | `APP__TELEMETRY__PYROSCOPE__EXPORTER` | empty | URL of the Pyroscope server |
| `app:telemetry:pyroscope:identifier` | `APP__TELEMETRY__PYROSCOPE__IDENTIFIER` | `OpenCTI` | Application name of this platform in Pyroscope |

:::example[Trace and profile a slow platform]
Example CERT investigates slow searches on its OpenCTI. It sends traces to the OpenTelemetry collector of its monitoring stack and profiles to its Pyroscope server, under a name that tells this platform apart:

```yaml
services:
  opencti:
    environment:
      - APP__TELEMETRY__TRACING__ENABLED=true
      - APP__TELEMETRY__TRACING__EXPORTER_OTLP=http://otel-collector.example.com:4318/v1/traces
      - APP__TELEMETRY__PYROSCOPE__ENABLED=true
      - APP__TELEMETRY__PYROSCOPE__EXPORTER=http://pyroscope.example.com:4040
      - APP__TELEMETRY__PYROSCOPE__IDENTIFIER=opencti-example-cert
```

After `docker compose up -d opencti`, `docker compose logs opencti | grep Pyroscope` shows `Pyroscope plugin successfully loaded.`, and the traces of the `opencti-platform` service reach the collector.
:::

## OpenCTI metrics

`app:schema_metrics` is not an observability setting: it declares custom metrics on entities, data of the knowledge graph. See [Functional customization](/reference/opencti/configuration/#functional-customization).

The [OpenCTI metrics reference](/reference/opencti/metrics/) lists every metric your version exports, with its kind, unit and attributes. They count GraphQL requests and errors (`opencti_api_requests`, `opencti_api_errors`), time each request (`opencti_api_latency`, in milliseconds), and measure emails sent and ingestion bulks.

:::only{opencti=">=7.261030"}
Recent versions add lock contention and data migration timings, the state of each dependency and the storage used.
:::

Prometheus adds `_total` to the name of a counter and `_bucket`, `_sum` and `_count` to a histogram.

### Attributes of the request metrics

`opencti_api_requests`, `opencti_api_errors` and `opencti_api_latency` carry these attributes:

| Attribute | Value |
| --- | --- |
| `operation` | `query`, `mutation` or `subscription` |
| `name` | Name of the GraphQL operation, `Unspecified` when it has none |
| `status` | `SUCCESS` or `ERROR` |
| `type` | Error type, such as `FUNCTIONAL_ERROR`, `VALIDATION_ERROR` or `DATABASE_ERROR`, on errors only |
| `user_agent` | User agent of the client, `Unspecified` when it sends none |

Authentication and permission errors (`AUTH_REQUIRED`, `AUTH_FAILURE`, `FORBIDDEN_ACCESS`) are not counted as errors: the request goes to `opencti_api_requests` with the status `SUCCESS`.

### Attributes of the email and bulk metrics

| Metric | Attribute | Value |
| --- | --- | --- |
| `opencti_sent_email` | `category` | Why the email was sent: `notification`, `user-notification`, `dissemination`, `password-reset`, `password-change` or `hub-registration` |
| `opencti_sent_email` | `identifier` | ID of what the email comes from: the user (`user-<id>` for `user-notification`), the dissemination list or the triggers of the notification. None for `hub-registration` |
| `opencti_api_direct_bulk`, `opencti_api_side_bulk` | `type` | Entity type of the indexed elements, such as `Report`, `History` or `Activity` |

`opencti_api_direct_bulk` records the size of each bulk request that writes elements to Elasticsearch. `opencti_api_side_bulk` records the size of the bulk request that then updates the entities at both ends of the relationships it wrote. Both count the lines of the request: two for each document.

:::only{opencti=">=7.261030"}
### Health and usage metrics

`opencti_dependency_up` is `1` when a dependency answered the last check and `0` when it failed, for each `dependency`: `elasticsearch`, `storage`, `rabbitmq` or `redis`. The usage metrics give the size of the primary store of Elasticsearch or OpenSearch, the size of the stored files and the consumers on the push queues.

A background monitor refreshes these values, so a scrape never contacts a dependency. Each node checks its dependencies itself, every `app:health_monitoring:dependency_check_interval` milliseconds (default `30000`). The usage metrics describe the whole cluster: one node computes them and shares them through Redis, and every node reads the shared value every `app:health_monitoring:usage_metrics_interval` milliseconds (default `300000`).

The usage metrics need their manager, which is off by default: set `PLATFORM_USAGE_METRICS_MANAGER__ENABLED=true`. A usage metric that could not be collected is left out, not exported as `0`.

`opencti_queue_consumers` has one value for each `connector_type` declared on the push queues, such as `EXTERNAL_IMPORT` or `INTERNAL_ENRICHMENT`, and `UNKNOWN` for the queues without a type. OpenCTI does not add them up: sum the types that matter to you, such as the consumers of the import queues to follow the capacity of your workers.
:::

### Node.js runtime metrics

OpenCTI also exports the standard Node.js metrics of its process: CPU, memory, heap, event loop lag and garbage collection, such as `process_resident_memory_bytes` and `nodejs_eventloop_lag_seconds`.

## OpenAEV metrics

| Metric | Type | What it measures |
| --- | --- | --- |
| `openaev_dependency_up` | Gauge | `1` when the last check of a dependency succeeded, `0` when it failed. Attribute `dependency`: `postgresql`, `rabbitmq`, `s3` or `engine` (Elasticsearch) |
| `openaev_dependency_probe_duration_seconds` | Gauge | Duration of the last check of each dependency |
| `openaev_storage_used_bytes` | Gauge | Size used in each store. Attribute `storage`: `postgresql`, `engine` or `s3` |

OpenAEV checks its dependencies every `openaev.healthcheck.connectivity-probe-interval` (default `PT10S`, ten seconds) and measures the storage every `openaev.healthcheck.storage-probe-interval` (default `PT4H`). The endpoint also serves the standard Spring Boot and Java virtual machine metrics.

## What's next

- [Check that the platforms are healthy](/deploy/operate/health-probes/), to let an orchestrator or a load balancer check the platforms.
- [OpenCTI configuration](/reference/opencti/configuration/) and [OpenAEV configuration](/reference/openaev/configuration/), every key.
- The [OpenCTI metrics reference](/reference/opencti/metrics/), every metric of your version.
- [Troubleshoot the XTM stack](/deploy/operate/troubleshoot/), to find the logs.
