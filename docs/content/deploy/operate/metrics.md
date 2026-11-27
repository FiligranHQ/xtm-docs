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
These metrics stay in your infrastructure. The anonymous usage data the platforms send to Filigran is something else: see [Usage telemetry and privacy](/opencti/configure/telemetry/), and [the usage telemetry of OpenCRQ](/opencrq/administer/telemetry/).
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
| `TELEMETRY__OBSERVABILITY__PROMETHEUS__PORT` | `14269` | Port of the endpoint. On the port of the health check, `HEALTHCHECK_HTTP_SERVER__PORT` (`8444` in xtm-docker), the endpoint stays off |
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

The port is not published on the host, and the endpoint asks for no credentials: keep it on the Docker network of the stack. Anyone who reaches it reads the version of OpenCRQ, the address, port and name of its database, the processes and how many of each run, the SQLSTATE codes and the kinds of SQL operation of its queries, the name of every API procedure (`trpc_procedure`) and the kinds of background job (`job_kind`). No table name and no SQL text.

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
      - APP__APP_LOGS__LOGS_LEVEL=info
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

## Find OpenCTI's settings in OpenCRQ

OpenCRQ's observability settings do the work of OpenCTI's under other names, with three differences on purpose:

- One root, `TELEMETRY__`: `TELEMETRY__OBSERVABILITY__*` for your monitoring and `TELEMETRY__USAGE__*` for the usage telemetry ([Telemetry](/deploy/configure/opencrq/#telemetry)), where OpenCTI has `APP__TELEMETRY__*` and `TELEMETRY_MANAGER__*`.
- One `ENABLED` switch for each transport of the metrics, the Prometheus endpoint and the OTLP push. OpenCTI has one switch for both, and turns each on by its port or its URL.
- A log export to the OTLP collector, which OpenCTI does not have.

| OpenCRQ | OpenCTI | What differs |
| --- | --- | --- |
| `TELEMETRY__OBSERVABILITY__PROMETHEUS__ENABLED` | `APP__TELEMETRY__METRICS__ENABLED` | OpenCTI's switch turns both transports on |
| `TELEMETRY__OBSERVABILITY__PROMETHEUS__PORT` | `APP__TELEMETRY__METRICS__EXPORTER_PROMETHEUS` | Nothing: same default, `14269` |
| `TELEMETRY__OBSERVABILITY__PROMETHEUS__HOST` | `APP__TELEMETRY__METRICS__EXPORTER_PROMETHEUS_HOST`, from OpenCTI 7.261119 | Nothing: same default, `0.0.0.0` |
| `TELEMETRY__OBSERVABILITY__PROMETHEUS__COLLECT_TIMEOUT_MS`, `TELEMETRY__OBSERVABILITY__PROMETHEUS__ROUND_DEADLINE_MS`, `TELEMETRY__OBSERVABILITY__PROMETHEUS__MIN_INTERVAL_MS`, `TELEMETRY__OBSERVABILITY__PROMETHEUS__MAX_REPLY_BYTES`, `TELEMETRY__OBSERVABILITY__PROMETHEUS__PUBLIC_PORT` | None | OpenCTI serves the metrics of its own process, with nothing to collect from others |
| `TELEMETRY__OBSERVABILITY__OTLP__METRICS__ENABLED` | `APP__TELEMETRY__METRICS__ENABLED` | OpenCTI's switch turns both transports on |
| `TELEMETRY__OBSERVABILITY__OTLP__METRICS__EXPORTER` | `APP__TELEMETRY__METRICS__EXPORTER_OTLP` | OpenCRQ pushes over gRPC every 60 seconds, OpenCTI over HTTP every second |
| `TELEMETRY__OBSERVABILITY__OTLP__TRACES__ENABLED` | `APP__TELEMETRY__TRACING__ENABLED` | Nothing |
| `TELEMETRY__OBSERVABILITY__OTLP__TRACES__EXPORTER` | `APP__TELEMETRY__TRACING__EXPORTER_OTLP` | OpenCRQ sends over gRPC, to an address such as `http://otel-collector:4317`; OpenCTI over HTTP, to the full URL ending in `/v1/traces`. OpenCRQ has no Zipkin export |
| `TELEMETRY__OBSERVABILITY__OTLP__LOGS__ENABLED`, `TELEMETRY__OBSERVABILITY__OTLP__LOGS__EXPORTER` | None | OpenCTI does not export its logs |
| `TELEMETRY__OBSERVABILITY__PYROSCOPE__ENABLED` | `APP__TELEMETRY__PYROSCOPE__ENABLED` | Nothing |
| `TELEMETRY__OBSERVABILITY__PYROSCOPE__SERVER_ADDRESS` | `APP__TELEMETRY__PYROSCOPE__EXPORTER` | Nothing |
| `TELEMETRY__OBSERVABILITY__SERVICE_NAME` | `APP__TELEMETRY__PYROSCOPE__IDENTIFIER`, for Pyroscope | OpenCRQ's name also goes on its traces, logs and metrics. OpenCTI's traces always carry `opencti-platform` |

## OpenCRQ metrics

Every series carries the labels `process_name`, `service_name` and `service_version`. With the default `split` profile, `process_name` is `http`, `worker` or `pm`, the process manager, which sends the supervision metrics and the Node.js metrics of its own process. With the `monolith` profile, it is `main` or `pm`, and `main` sends what `http` and `worker` send.

The endpoint names a family after its OpenTelemetry name, with `_` for each dot and `_total` after a counter; it never adds the unit, which the table gives. The OTLP push sends the dotted names, such as `db.client.connection.count`, every 60 seconds, with one more attribute, `service.instance.id`: the name and the process ID of the process, such as `worker-412`, new at every restart. The scraped series never carry it.

To read a container as a whole, add up a counter or a count across its processes, with `sum by (instance)`, and keep `instance` in the grouping when Prometheus scrapes several containers, or their series are added together. Never add up a ratio or a delay, such as `nodejs_eventloop_utilization` or `nodejs_eventloop_delay_mean`: take its `max` or its `avg`. A percentile, such as `nodejs_eventloop_delay_p99`, cannot be combined across processes: take the worst one, with `max`.

| Metrics | Type | Processes | What they measure |
| --- | --- | --- | --- |
| `opencrq_pm_processes` | Gauge | `pm` | Supervised processes in each state of their lifecycle, attribute `state`: `FORKING`, `BOOTING`, `STARTING`, `RUNNING`, `STOPPING`, `STOPPED`, `EXITED` or `ERROR` |
| `opencrq_pm_collection_ok`, `opencrq_pm_collection_duration`, `opencrq_pm_collection_failures_total` | Gauge, histogram, counter | `pm` | For each process the endpoint collects (`supervised_process`, `pm` included): whether its last collection succeeded (`1`) or not (`0`), how long it took, in seconds, and its failures by `reason`: `timeout`, `error`, `not_connected`, `not_running`, `oversized`, `invalid`, `empty` or `send_error`. Only while the Prometheus endpoint runs: see [How the endpoint collects the processes](#how-the-endpoint-collects-the-processes) |
| `db_client_connection_count`, `db_client_connection_limit`, `db_client_connection_pending_requests` | Gauge | `http`, `worker` | The connections of each database pool, `used` or `idle` (attribute `db_client_connection_state`), the limit of the pool and the callers waiting for one |
| `db_client_connection_wait_time`, `db_client_connection_use_time`, `db_client_connection_timeouts_total` | Histogram, counter | `http`, `worker` | The time to get a connection and the time it was held, in seconds, and the requests that timed out |
| `opencrq_db_client_connection_reconnection_attempts_total` | Counter | `http`, `worker` | Attempts to connect to the database again after an error of the pool |
| `db_client_operation_duration` | Histogram | `http`, `worker` | Duration of the database queries, in seconds, by `db_operation_name`: `SELECT`, `INSERT`, `UPDATE`, `DELETE` or `_OTHER`. Also by `server_address`, `server_port`, `db_namespace` (the database), `db_system_name` and, when the PostgreSQL instrumentation reports it, `error_type`: a SQLSTATE of the classes `08`, `22`, `23`, `25`, `28`, `3D`, `40`, `42`, `53`, `55`, `57`, `58` or `XX`, any other as `_OTHER` |
| `opencrq_trpc_server_duration` | Histogram | `http` | Duration of the API calls, in seconds, by `trpc_type` (`query` or `mutation`) and `error_type` (a tRPC error code, such as `NOT_FOUND`, or `_OTHER`). No procedure, on purpose: it is on the counter below. Subscriptions are not measured |
| `opencrq_trpc_server_calls_total` | Counter | `http` | The API calls, by procedure (`trpc_procedure`), `trpc_type` and `error_type` |
| `opencrq_recompute_asks_total` | Counter | `http`, `worker` | Requests to run a background job again, by `job_kind`, `path`, `already_existed` (`true` when the request joined a job already queued) and `reason`. The `path` is `dispatcher` (the dispatcher of background jobs), `followup` (asked by the platform after its own work), `manual` (asked from the interface) or `sweep` (a backfill). On the `dispatcher` path only, `reason` says why: `change` (the job's data changed), `reconnect` (catching up after a reconnection), `daily` (the daily run) or `followup` (its data changed during its last run). `manual` asks and some `followup` asks come from `http` |
| `opencrq_recompute_unattributed_reads_total` | Counter | `worker` | Runs of a background job whose reads could not be tied to tables, so that any change of data asks for it again, by `job_kind`. An engineering signal |
| `opencrq_recompute_feeding_cycles_total` | Counter | `worker` | Kinds of background job whose reads and writes feed each other, by `job_kind` and `cycle` (the kinds joined by ` -> `). They still run, and the log says `Recompute jobs feed each other: their reads and writes form a cycle`. An engineering signal |
| `opencrq_opencti_mirror_rebuild_actors_total` | Counter | `worker` | Threats of the OpenCTI stream that a rebuild settled, by `outcome`: `rebuilt`, `retracted` (deleted in OpenCTI) or `failed` |
| `opencrq_events_doorbells_total`, `opencrq_events_doorbell_procedures` | Counter, histogram | `http` | The data changes that make the open screens reload, by `resolution` (`named`, the queries it makes stale are known, or `all`), and how many queries each `named` one makes stale |
| `opencrq_events_query_index_explains_total`, `opencrq_events_query_index_procedures` | Counter, gauge | `http` | The `EXPLAIN` statements run to learn which tables each query reads, by `outcome` (`ok` or `refused`), and the queries the index knows, by `state`: `observed`, `declared`, `widened` or `without_reads`. An engineering signal |
| `nodejs_eventloop_utilization`, `nodejs_eventloop_delay_max`, `nodejs_eventloop_delay_mean`, `nodejs_eventloop_delay_p99` | Gauge | Every process | Load of the event loop, a ratio, and its delay, in seconds |
| `nodejs_eventloop_time_total` | Counter | Every process | Time the event loop spent `active` or `idle` (attribute `nodejs_eventloop_state`), in seconds |
| `v8js_gc_duration`, `v8js_memory_heap_used` | Histogram, gauge | Every process | Garbage collection time, in seconds, every kind of collection in one series, and the heap used, in bytes, one series for each heap space (`v8js_heap_space_name`): add them up |
| `process_cpu_time_total`, `process_cpu_utilization`, `process_memory_usage` | Counter, gauge | Every process | CPU time, in seconds, and share of the CPU, by `cpu_mode` (`user` or `system`), and memory of each process, in bytes |

The series of a pool carry its name, `db_client_connection_pool_name`: `host:port/database`, such as `pgsql:5432/opencrq` in xtm-docker. Each of `http` and `worker` has two pools: the main one, of `POSTGRES__POOL_MAX` connections, and the one of the background jobs' leases, `pgsql:5432/opencrq#control`, of 5 connections.

A family appears only once it has something to report: a family that counts failures after the first one, such as `db_client_connection_timeouts_total`, and the wait and use times of a pool after its first connection. Write the panels and alerts so that a missing series counts as zero. `or vector(0)` gives a series without labels: it fills a total, not a series by process or by instance.

### Histogram buckets

A panel or a `histogram_quantile` on `le` reads these bounds. A quantile never goes above the highest one.

| Histogram | Bounds |
| --- | --- |
| `db_client_operation_duration`, `db_client_connection_wait_time` | 0.005, 0.025, 0.1, 0.5, 2, 10 and 60 seconds |
| `db_client_connection_use_time` | The same, then 300 and 1800 seconds |
| `opencrq_trpc_server_duration` | 0.05, 0.25, 1, 5, 30 and 120 seconds |
| `v8js_gc_duration` | 0.001, 0.01, 0.1, 1, 5 and 10 seconds |
| `opencrq_pm_collection_duration` | 0.05, 0.25, 1, 2.5 and 4.5 seconds |
| `opencrq_events_doorbell_procedures` | 0, 1, 2, 5, 10, 20, 50 and 100 queries |

### Left out on purpose

OpenCRQ drops these families before either transport, so do not look for them:

- The event loop delays other than the maximum, the mean and the p99: `nodejs.eventloop.delay.min`, `nodejs.eventloop.delay.stddev`, `nodejs.eventloop.delay.p50` and `nodejs.eventloop.delay.p90`.
- `v8js.memory.heap.space.*`, three families on the heap spaces `v8js_memory_heap_used` already reports, and `v8js.resource.active`, whose series would grow with what the process holds.
- The HTTP and system metrics of the instrumentations: `http.server.*`, `http.client.*` and `system.*`.
- The pool metrics of the PostgreSQL instrumentation, `db.client.connection.*`, including `db.client.connection.max`, the former name of the limit: OpenCRQ measures its pools itself, under the names of the table.

`v8js.memory.heap.limit` is never sent. OpenCRQ ships no dashboard and no alert rule.

### Show OpenCRQ on OpenCTI's runtime dashboards

OpenCTI exports its Node.js metrics with the package `opentelemetry-node-metrics` 3.0.0, under other names than OpenCRQ's. These recording rules write OpenCRQ's under OpenCTI's names, so that a panel built on OpenCTI's Node.js metrics shows OpenCRQ unchanged, as long as it selects no `job` of its own: the recorded series keep `job="opencrq"`. OpenCTI has no pool, query or supervision metrics, so only the Node.js metrics need a rule.

| OpenCRQ | OpenCTI | Rule |
| --- | --- | --- |
| `nodejs_eventloop_delay_max`, `nodejs_eventloop_delay_mean`, `nodejs_eventloop_delay_p99` | `nodejs_eventloop_lag_max_seconds`, `nodejs_eventloop_lag_mean_seconds`, `nodejs_eventloop_lag_p99_seconds` | Rename. The p99 is the worst process's, not a percentile of the container |
| `v8js_gc_duration` | `nodejs_gc_duration_seconds` | Rename. No `kind` label: the four kinds of collection are one series. OpenCTI's bounds are 0.001, 0.01, 0.1, 1, 2 and 5 seconds, so a panel on `le="2"` stays empty |
| `v8js_memory_heap_used` | `nodejs_heap_space_size_used_bytes` | Rename of the metric and of its label, `space`, whose values lose their `_space` suffix |
| `v8js_memory_heap_used`, added up over the spaces | `nodejs_heap_size_used_bytes` | Close, not equal: OpenCTI's is the heap used that Node.js reports for the whole process |
| `process_memory_usage` | `process_resident_memory_bytes` | Rename |
| `process_cpu_time_total`, `user`, `system` or both | `process_cpu_user_seconds_total`, `process_cpu_system_seconds_total`, `process_cpu_seconds_total` | Rename, and a sum for the total |
| `nodejs_eventloop_utilization` | None: a ratio, where OpenCTI measures a lag | Point the panel at `nodejs_eventloop_lag_mean_seconds` |
| `process_cpu_utilization` | None | Point the panel at `rate(process_cpu_seconds_total[5m])` |
| `nodejs_eventloop_time_total` | None | Query it under its own name |

Every rule selects `job="opencrq"`. OpenCTI serves `process_cpu_seconds_total`, `process_cpu_user_seconds_total`, `process_cpu_system_seconds_total` and `process_resident_memory_bytes` too, and without the selector the series of both modules would be added together. Change it when your job has another name; never remove it.

```yaml
groups:
  - name: opencrq-opencti-runtime
    rules:
      - record: nodejs_eventloop_lag_max_seconds
        expr: max without (process_name) (nodejs_eventloop_delay_max{job="opencrq"})
      - record: nodejs_eventloop_lag_mean_seconds
        expr: avg without (process_name) (nodejs_eventloop_delay_mean{job="opencrq"})
      - record: nodejs_eventloop_lag_p99_seconds
        expr: max without (process_name) (nodejs_eventloop_delay_p99{job="opencrq"})
      - record: process_resident_memory_bytes
        expr: sum without (process_name) (process_memory_usage{job="opencrq"})
      - record: nodejs_heap_size_used_bytes
        expr: sum without (process_name, v8js_heap_space_name) (v8js_memory_heap_used{job="opencrq"})
      - record: nodejs_heap_space_size_used_bytes
        expr: sum without (process_name, v8js_heap_space_name) (label_replace(v8js_memory_heap_used{job="opencrq"}, "space", "$1", "v8js_heap_space_name", "(.*)_space"))
      - record: process_cpu_user_seconds_total
        expr: sum without (process_name, cpu_mode) (process_cpu_time_total{job="opencrq", cpu_mode="user"})
      - record: process_cpu_system_seconds_total
        expr: sum without (process_name, cpu_mode) (process_cpu_time_total{job="opencrq", cpu_mode="system"})
      - record: process_cpu_seconds_total
        expr: sum without (process_name, cpu_mode) (process_cpu_time_total{job="opencrq"})
      - record: nodejs_gc_duration_seconds_bucket
        expr: sum without (process_name) (v8js_gc_duration_bucket{job="opencrq"})
      - record: nodejs_gc_duration_seconds_sum
        expr: sum without (process_name) (v8js_gc_duration_sum{job="opencrq"})
      - record: nodejs_gc_duration_seconds_count
        expr: sum without (process_name) (v8js_gc_duration_count{job="opencrq"})
```

The mean is exact only when every process measured as many delays. The heap space rule gives a `space` label to the spaces whose name ends in `_space`, which V8 gives all of them today. A panel on another runtime metric of OpenCTI, such as `nodejs_eventloop_lag_seconds`, stays empty.

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

On the main pool, raise `POSTGRES__POOL_MAX`, and `max_connections` on PostgreSQL with it: see [Database](/deploy/configure/opencrq/#database). The alert can also fire on the `#control` pool, at its limit of 5, which `POSTGRES__POOL_MAX` does not change.

A threat that fails its rebuild fails again at every run, and the run still succeeds. Alert when `increase(opencrq_opencti_mirror_rebuild_actors_total{outcome="failed"}[1h]) > 0` stays true over several hours. The log line `OpenCTI actor rebuild failed, its rows stay marked` gives the STIX ID of the threat.

### How the endpoint collects the processes

At each scrape, the process manager asks every process for its metrics and merges them into one document. A process that answers too late, or sends too much, is left out of that document and counted in `opencrq_pm_collection_failures_total`. The scrape is not broken: it waits at most `COLLECT_TIMEOUT_MS` for each process and `ROUND_DEADLINE_MS` in all, then gets the metrics of the processes that answered in time.

| Environment variable | Default | What it sets |
| --- | --- | --- |
| `TELEMETRY__OBSERVABILITY__PROMETHEUS__COLLECT_TIMEOUT_MS` | `2500` | How long the endpoint waits for each process, in milliseconds |
| `TELEMETRY__OBSERVABILITY__PROMETHEUS__ROUND_DEADLINE_MS` | `4500` | How long a whole collection may take. Must be above `COLLECT_TIMEOUT_MS`, or OpenCRQ stops at start |
| `TELEMETRY__OBSERVABILITY__PROMETHEUS__MIN_INTERVAL_MS` | `5000` | Within this time after a collection starts, a scrape gets the same document again, failures included |
| `TELEMETRY__OBSERVABILITY__PROMETHEUS__MAX_REPLY_BYTES` | `2097152` | The largest share of one process, in bytes. A larger one is refused, with the reason `oversized` |
| `TELEMETRY__OBSERVABILITY__PROMETHEUS__PUBLIC_PORT` | None | The port the endpoint is published on, when it differs. Written in the logs only |

The process manager records the outcome of a collection after it has taken its own metrics, so the collection families show each outcome one scrape late, and are missing from the first scrape after a start. They exist only while the Prometheus endpoint runs; with the OTLP push on too, the process manager pushes them as well.

The endpoint stays off, with an error in the logs, when it cannot start, when its port is the port of the health check (`Metrics scrape port is already the healthcheck port; metrics endpoint disabled`), or when a process of the profile takes the process manager's own name (`Process profile names a child after the process manager's own reserved name; metrics endpoint disabled`). The rest of the platform keeps running. When Prometheus cannot connect, look for these lines first.

## What's next

- [Check that the platforms are healthy](/deploy/operate/health-probes/), to let an orchestrator or a load balancer check the platforms.
- [OpenCTI configuration](/reference/opencti/configuration/) and [OpenAEV configuration](/reference/openaev/configuration/), every key.
- [Configure OpenCRQ](/deploy/configure/opencrq/#telemetry), for its usage telemetry and the names its telemetry variables had in earlier builds.
- The [OpenCTI metrics reference](/reference/opencti/metrics/), every metric of your version.
- [Troubleshoot the XTM stack](/deploy/operate/troubleshoot/), to find the logs.
