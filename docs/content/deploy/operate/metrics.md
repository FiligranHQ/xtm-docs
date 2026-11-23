---
title: Observability metrics
description: The Prometheus and OpenTelemetry metrics OpenCTI, OpenAEV and OpenCRQ expose for your own monitoring, how to turn them on in the XTM stack and what each one measures.
doc_type: reference
products:
  - opencti
  - openaev
  - opencrq
personas: [platform-engineer]
owner: docs-platform
last_reviewed: 2026-11-20
migrated_from:
  - opencti:/deployment/telemetry/
---

OpenCTI, OpenAEV and OpenCRQ expose observability metrics for your own monitoring system, such as Prometheus or an OpenTelemetry (OTLP) collector. This page says how to turn them on in the XTM stack, where to scrape them and what the metrics measure. They are all off by default.

::diagram{id=deploy-operate-metrics}

:::note[Not usage telemetry]
These metrics stay in your infrastructure. The anonymous usage data the platforms send to Filigran is something else: see [Usage telemetry and privacy](/opencti/configure/telemetry/), and the [telemetry of OpenCRQ](/deploy/configure/opencrq/#telemetry).
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
:::tab[OpenCRQ]
OpenCRQ serves its metrics to Prometheus, pushes them to an OTLP collector, or both: each transport has its own switch. The Prometheus endpoint of a container gathers the metrics of all its processes.

| Environment variable | Default | What it does |
| --- | --- | --- |
| `TELEMETRY__OBSERVABILITY__PROMETHEUS__ENABLED` | `false` | Turns the Prometheus endpoint on, served at `/metrics` |
| `TELEMETRY__OBSERVABILITY__PROMETHEUS__PORT` | `14269` | Port of the endpoint. On the port of the health check, `8444`, the endpoint stays off |
| `TELEMETRY__OBSERVABILITY__PROMETHEUS__HOST` | `0.0.0.0` | Address the endpoint listens on. `127.0.0.1` keeps it inside the container |
| `TELEMETRY__OBSERVABILITY__OTLP__METRICS__ENABLED` | `false` | Turns the push to an OTLP collector on |
| `TELEMETRY__OBSERVABILITY__OTLP__METRICS__EXPORTER` | None | URL of the collector, over gRPC, such as `http://otel-collector:4317`. Required when the push is on |

:::steps
1. Create `docker-compose.override.yml` in the `xtm-docker` folder, or add to it.

   ```yaml
   services:
     opencrq:
       environment:
         - TELEMETRY__OBSERVABILITY__PROMETHEUS__ENABLED=true
   ```

2. Recreate the platform.

   ```bash
   docker compose up -d opencrq
   ```

   :::verify
   ```bash
   docker compose exec opencrq wget -qO- http://localhost:14269/metrics | grep '^# TYPE opencrq_'
   ```
   It lists the `opencrq_` metrics, such as `opencrq_pm_processes`.
   :::
:::

The port is not published on the host, and the endpoint asks for no credentials: keep it on the Docker network of the stack. Its metrics show the version of OpenCRQ and the address and name of its database.

Scrape `opencrq:14269` every 15 seconds or more. Within 5 seconds of a collection (`TELEMETRY__OBSERVABILITY__PROMETHEUS__MIN_INTERVAL_MS`), the endpoint serves the same document again.

```yaml
scrape_configs:
  - job_name: opencrq
    scrape_interval: 15s
    static_configs:
      - targets: ["opencrq:14269"]
```

The names of earlier builds, such as `TELEMETRY__METRICS__PROMETHEUS__ENABLED`, stop the platform at start: see [Telemetry](/deploy/configure/opencrq/#telemetry).
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

## OpenCRQ traces, logs and profiles

OpenCRQ can also send traces and logs to an OTLP collector, and continuous profiles to Pyroscope. Each is off by default and has its own switch. Every process of the container sends its own data.

| Environment variable | Default | What it does |
| --- | --- | --- |
| `TELEMETRY__OBSERVABILITY__SERVICE_NAME` | `opencrq-platform` | Service name of the traces, logs and metrics, and application name in Pyroscope |
| `TELEMETRY__OBSERVABILITY__OTLP__TRACES__ENABLED` | `false` | Sends traces, such as a span for each HTTP request and each PostgreSQL query |
| `TELEMETRY__OBSERVABILITY__OTLP__TRACES__EXPORTER` | None | URL of the collector, over gRPC, such as `http://otel-collector:4317`. Required when traces are on |
| `TELEMETRY__OBSERVABILITY__OTLP__LOGS__ENABLED` | `false` | Sends the logs to the collector too. The container keeps writing them |
| `TELEMETRY__OBSERVABILITY__OTLP__LOGS__EXPORTER` | None | URL of the collector, over gRPC. Required when logs are on |
| `TELEMETRY__OBSERVABILITY__PYROSCOPE__ENABLED` | `false` | Sends memory and CPU profiles to Pyroscope |
| `TELEMETRY__OBSERVABILITY__PYROSCOPE__SERVER_ADDRESS` | None | URL of the Pyroscope server, such as `http://pyroscope:4040`. Required when profiling is on |

A switch on without its URL stops the platform at start. With a switch off, OpenCRQ does not check the other settings of that transport, so an empty URL beside it is fine.

The data of each process carries its name in `process.name`: `http`, `worker` or `pm`, the process manager that supervises the other two. Pyroscope tags each profile with the same name and the process ID. When the OTLP collector does not answer, OpenCRQ logs the error once every 5 minutes, with the number of failures since.

:::example[Send the traces and logs of OpenCRQ to the collector]
Example CERT sends the traces and logs of its OpenCRQ to the OpenTelemetry collector of its monitoring stack, under a name that tells this platform apart:

```yaml
services:
  opencrq:
    environment:
      - TELEMETRY__OBSERVABILITY__SERVICE_NAME=opencrq-example-cert
      - TELEMETRY__OBSERVABILITY__OTLP__TRACES__ENABLED=true
      - TELEMETRY__OBSERVABILITY__OTLP__TRACES__EXPORTER=http://otel-collector.example.com:4317
      - TELEMETRY__OBSERVABILITY__OTLP__LOGS__ENABLED=true
      - TELEMETRY__OBSERVABILITY__OTLP__LOGS__EXPORTER=http://otel-collector.example.com:4317
```

After `docker compose up -d opencrq`, the collector receives the traces and the logs of the `opencrq-example-cert` service, with one `process.name` for each process.
:::

## OpenCRQ metrics

Every series carries the labels `process_name`, `service_name` and `service_version`. With the default `split` profile, `process_name` is `http`, `worker` or `pm`. The OTLP push sends the same metrics under their dotted names, such as `db.client.connection.count`.

To read a container as a whole, add up a counter or a count across its processes, with `sum by (instance)`. Never add up a ratio or a delay, such as `nodejs_eventloop_utilization` or `nodejs_eventloop_delay_p99`: take its `max` or its `avg`.

| Metrics | Type | What they measure |
| --- | --- | --- |
| `opencrq_pm_processes` | Gauge | Processes in each state of their lifecycle, attribute `state`, such as `RUNNING`, `STOPPED` or `ERROR` |
| `opencrq_pm_collection_ok`, `opencrq_pm_collection_failures_total` | Gauge, counter | Whether the endpoint collected the metrics of each process (`supervised_process`), and its failures by `reason`, such as `timeout`. Prometheus endpoint only |
| `db_client_connection_count`, `db_client_connection_limit`, `db_client_connection_pending_requests` | Gauge | The database connections of each process, `used` or `idle` (attribute `db_client_connection_state`), the limit of its pool and the callers waiting for one |
| `db_client_connection_wait_time`, `db_client_connection_use_time`, `db_client_connection_timeouts_total` | Histogram, counter | The time to get a connection and to use it, in seconds, and the requests that timed out |
| `db_client_operation_duration` | Histogram | Duration of the database queries, by `db_operation_name`: `SELECT`, `INSERT`, `UPDATE`, `DELETE` or `_OTHER` |
| `opencrq_db_client_connection_reconnection_attempts_total` | Counter | Attempts to connect to the database again after an error of the pool |
| `opencrq_trpc_server_duration`, `opencrq_trpc_server_calls_total` | Histogram, counter | Duration of the API calls, in seconds, and the calls by procedure (`trpc_procedure`) and error (`error_type`) |
| `opencrq_recompute_asks_total` | Counter | Requests to run a background job again after a change of its data, by `job_kind` and `reason` |
| `opencrq_opencti_mirror_rebuild_actors_total` | Counter | Threats of the OpenCTI stream that a rebuild settled, by `outcome`: `rebuilt`, `retracted` (deleted in OpenCTI) or `failed` |
| `opencrq_events_doorbells_total`, `opencrq_events_doorbell_procedures` | Counter, histogram | The data changes that make the open screens reload, and how many queries each one makes stale |
| `nodejs_eventloop_utilization`, `nodejs_eventloop_delay_max`, `nodejs_eventloop_delay_mean`, `nodejs_eventloop_delay_p99` | Gauge | Load and delay of the event loop of each process |
| `v8js_gc_duration`, `v8js_memory_heap_used` | Histogram, gauge | Garbage collection time, and the heap used, one series for each heap space (`v8js_heap_space_name`): add them up |
| `process_cpu_time_total`, `process_cpu_utilization`, `process_memory_usage` | Counter, gauge | CPU time by `cpu_mode`, `user` or `system`, and memory of each process, in bytes |

A family that counts failures appears only after the first one, such as `db_client_connection_timeouts_total`: write the panels and alerts so that a missing series counts as zero.

The Node.js metrics of OpenCRQ do not take the names of OpenCTI's, such as `process_resident_memory_bytes` for `process_memory_usage`. A dashboard built for OpenCTI needs recording rules that rename them to show OpenCRQ.

### Alerts worth setting

The pool of a process is saturated when all its connections are in use and callers wait for one, across several scrapes:

```promql
(
  db_client_connection_count{db_client_connection_state="used"}
    >= ignoring (db_client_connection_state) db_client_connection_limit
)
and ignoring (db_client_connection_state) (
  db_client_connection_pending_requests > 0
)
```

Raise `POSTGRES__POOL_MAX`, and `max_connections` on PostgreSQL with it: see [Database](/deploy/configure/opencrq/#database).

A threat that fails its rebuild fails again at every run, and the run still succeeds. Alert when `increase(opencrq_opencti_mirror_rebuild_actors_total{outcome="failed"}[1h]) > 0` stays true over several hours. The log line `OpenCTI actor rebuild failed, its rows stay marked` gives the STIX ID of the threat.

### How the endpoint collects the processes

At each scrape, the process manager asks every process for its metrics and merges them into one document. A process that answers too late, or sends too much, is left out of that document and counted in `opencrq_pm_collection_failures_total`.

| Environment variable | Default | What it sets |
| --- | --- | --- |
| `TELEMETRY__OBSERVABILITY__PROMETHEUS__COLLECT_TIMEOUT_MS` | `2500` | How long the endpoint waits for each process, in milliseconds |
| `TELEMETRY__OBSERVABILITY__PROMETHEUS__ROUND_DEADLINE_MS` | `4500` | How long a whole collection may take. Must be above `COLLECT_TIMEOUT_MS`, or OpenCRQ stops at start |
| `TELEMETRY__OBSERVABILITY__PROMETHEUS__MIN_INTERVAL_MS` | `5000` | Within this time after a collection starts, a scrape gets the same document again, failures included |
| `TELEMETRY__OBSERVABILITY__PROMETHEUS__MAX_REPLY_BYTES` | `2097152` | The largest share of one process, in bytes. A larger one is refused, with the reason `oversized` |
| `TELEMETRY__OBSERVABILITY__PROMETHEUS__PUBLIC_PORT` | None | The port the endpoint is published on, when it differs. Written in the logs only |

The endpoint stays off, with an error in the logs, when it cannot start or when its port is the port of the health check: `Metrics scrape port is already the healthcheck port; metrics endpoint disabled`. The rest of the platform keeps running. When Prometheus cannot connect, look for these lines first.

## What's next

- [Check that the platforms are healthy](/deploy/operate/health-probes/), to let an orchestrator or a load balancer check the platforms.
- [OpenCTI configuration](/reference/opencti/configuration/) and [OpenAEV configuration](/reference/openaev/configuration/), every key.
- [Configure OpenCRQ](/deploy/configure/opencrq/#telemetry), for its usage telemetry and the names its telemetry variables had in earlier builds.
- The [OpenCTI metrics reference](/reference/opencti/metrics/), every metric of your version.
- [Troubleshoot the XTM stack](/deploy/operate/troubleshoot/), to find the logs.
