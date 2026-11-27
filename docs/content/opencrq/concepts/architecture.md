---
title: "How OpenCRQ works: processes, ports, storage and connections"
description: What runs in the OpenCRQ container, how its processes share PostgreSQL and S3 storage, which ports it listens on and which services it calls.
doc_type: explanation
products:
  - opencrq
personas: [platform-engineer, admin]
owner: docs-opencrq
last_reviewed: 2026-11-26
no_screenshot: "This page explains the processes of a container and the network flows between services, which no screen of OpenCRQ shows: the diagram draws them."
---

OpenCRQ runs as one container. Inside it, a process manager starts two Node.js processes: the HTTP process serves the interface and the API, and the worker runs the background jobs and the connectors. Both keep everything in PostgreSQL, and the files people upload in S3 storage. OpenCRQ calls XTM One, which it cannot start without, and the sources each tenant connects. This page explains what each part does, how the processes talk to each other, which ports they listen on and what leaves the container. With it, you can plan the network of a deployment, watch its health and find which part to look at when something stops. How OpenCRQ fits in the XTM stack, next to XTM One, OpenCTI and OpenAEV, is in [The XTM stack](/deploy/plan/architecture/). Its settings are in [Configure OpenCRQ](/deploy/configure/opencrq/).

## What OpenCRQ is made of

::diagram{id=opencrq-architecture-components}

| Part | What it is | Keeps data |
| --- | --- | --- |
| Process manager | The program the image starts, `node index-pm.mjs`. It starts the processes of a profile, watches them, and serves the health check and the Prometheus endpoint | No |
| HTTP process, `http` | Serves the web interface, the API and the sign-in through XTM One, registers with XTM One, and stores uploaded files | No |
| Worker process, `worker` | Runs the jobs: connector syncs, recomputes after a change, the daily recompute. It also holds the live stream of each tenant's OpenCTI | No |
| PostgreSQL | Every record of every tenant, and the job queue the two processes share, in the database `opencrq` in xtm-docker | Yes |
| S3 storage (RustFS in xtm-docker) | The files people upload, such as the STIX bundles of the **Threats** page, under a prefix per tenant, in the bucket `opencrq` by default. Optional | Yes |
| XTM One | Another module of the XTM platform, required: OpenCRQ registers with it, gets its edition from it and signs its users in through it | Its own |

The processes keep nothing on disk: the container can be recreated at any time, and you [back up](/deploy/operate/backup/) PostgreSQL and the bucket. OpenCRQ uses no Elasticsearch, RabbitMQ or Redis.

## The process manager and its profiles

The process manager reads `PROCESS_MANAGER__PROFILE` and starts each process of that profile as a separate Node.js process, with its own memory limit, the heap size of the `--max-old-space-size` option:

| Profile | Process | What it runs | Heap |
| --- | --- | --- | --- |
| `split`, the default | `http` | The interface, the API, the sign-in, the registration with XTM One, the uploads | 512 MB |
| | `worker` | The jobs, the connectors, the recomputes, the OpenCTI live streams | 4,096 MB |
| `monolith` | `main` | All of the above in one process | 2,048 MB |

xtm-docker runs the `split` profile and changes no heap. Keep `monolith` for development, as [HTTP and processes](/deploy/configure/opencrq/#http-and-processes) says. The process manager is a third Node.js process, which supervises and does no work of its own: give the container more memory than the two heaps together.

The HTTP process serves the interface from the folder in `FRONT_PATH`, which the image sets to `/app/public`, where it holds the built interface. Leave it: unset, the server looks for `../front/dist`, the folder of a development checkout.

The processes depend on each other. When one exits, or fails to start, the process manager stops the other one and exits with an error. It gives each process 30 seconds to stop cleanly, then kills it. The container stops with it: xtm-docker restarts it, with `restart: always` on the `opencrq` service. A `docker compose stop` ends the processes the same way.

Both processes connect to PostgreSQL when they start, and take turns behind a lock in the database. The first one applies the migrations and loads the reference data. The second one finds that work done for this start, and does not load the data again.

### How the two processes work together

The HTTP process and the worker never call each other. They meet in PostgreSQL:

- **Work goes through the job queue.** When someone clicks **Sync now** or uploads a STIX bundle, the HTTP process adds a job to a table. PostgreSQL notifies the worker, which claims the job and runs it, up to 4 at a time by default (`JOBS__MAX_CONCURRENT`). A worker that misses a notification checks the queue every 30 seconds anyway (`JOBS__POLLING_FALLBACK_MS`, in milliseconds).
- **Changes come back as notifications.** A write to a tenant's data makes PostgreSQL send a notification. The worker reads it to book the recomputes that depend on the changed data. The HTTP process reads it to tell the open browsers of that tenant to refresh what they show: it groups the changes people make within 100 milliseconds into one refresh (`EVENTS__DOORBELL_WINDOW_MS`), and the changes jobs write within 30 seconds (`EVENTS__JOB_DOORBELL_WINDOW_MS`), so that a long import does not refresh every open page for the whole of its run. Each time the worker starts listening to the notifications, at its start or after it lost them, it may have missed changes: it books every recompute of every tenant. A second catch-up waits until 10 minutes have passed since the previous one (`RECOMPUTE__RECONNECT_MIN_INTERVAL_MS`).
- **The worker reports it is alive.** It writes a heartbeat in the database every 10 seconds (`JOBS__HEARTBEAT_INTERVAL_MS`), whether it has work or not. When no worker has written one for 30 seconds (`JOBS__HEARTBEAT_STALE_AFTER_MS`, which OpenCRQ refuses at start unless it is longer than the interval), the **Integrations** page of every tenant shows **No worker is answering. Queued runs will not advance.**

Only three things go through the process manager: each process's health, the edition XTM One grants (the HTTP process learns it and the process manager passes it to the worker), and the metrics of each process for the Prometheus endpoint.

What the worker runs, and when, is listed in [Background processes](/reference/opencrq/processes/).

### Give one process its own settings

Every process reads the settings of the whole container, then the ones set for it alone, which win: the same variable with `PROCESS_MANAGER__<profile>__<process>__` in front, the profile and the process in upper case, such as `PROCESS_MANAGER__SPLIT__WORKER__POSTGRES__POOL_MAX`. Use them when one process needs what the other does not, such as a larger pool of database connections for the worker, or its own PostgreSQL account.

| Variable | What it sets for that process |
| --- | --- |
| `PROCESS_MANAGER__<profile>__<process>__SCRIPT` | The file the process runs: `index-http.mjs` and `index-worker.mjs` in `split`, `index.mjs` in `monolith`. The image sets it: leave it |
| `PROCESS_MANAGER__<profile>__<process>__POSTGRES__HOST` | The PostgreSQL host it connects to |
| `PROCESS_MANAGER__<profile>__<process>__POSTGRES__PORT` | The port of that host |
| `PROCESS_MANAGER__<profile>__<process>__POSTGRES__DATABASE` | The database |
| `PROCESS_MANAGER__<profile>__<process>__POSTGRES__USER` | The PostgreSQL user it signs in as |
| `PROCESS_MANAGER__<profile>__<process>__POSTGRES__PASSWORD` | That user's password. Pass it through a secret store |
| `PROCESS_MANAGER__<profile>__<process>__POSTGRES__POOL_MAX` | The most connections its pool opens, 10 at least, 20 by default |
| `PROCESS_MANAGER__<profile>__<process>__HTTP_SERVER__HOST` | The interface it listens on |
| `PROCESS_MANAGER__<profile>__<process>__HTTP_SERVER__PORT` | The port it listens on |
| `PROCESS_MANAGER__<profile>__<process>__HTTP_SERVER__PUBLIC_PORT` | The port the host publishes, written in its startup log only |
| `PROCESS_MANAGER__<profile>__<process>__HTTP_SERVER__MAX_RESPONSE_ITEMS` | The most items one API call returns, 20,000 by default |
| `PROCESS_MANAGER__<profile>__<process>__FRONT_PATH` | The folder of the built interface it serves |
| `PROCESS_MANAGER__<profile>__<process>__JOBS__ENABLED` | Whether it runs jobs. Required as soon as one of its `JOBS__` keys is set |
| `PROCESS_MANAGER__<profile>__<process>__JOBS__MAX_CONCURRENT` | How many jobs it runs at once, 4 by default |
| `PROCESS_MANAGER__<profile>__<process>__JOBS__POLLING_FALLBACK_MS` | How often it checks the queue without a notification |
| `PROCESS_MANAGER__<profile>__<process>__JOBS__HEARTBEAT_INTERVAL_MS` | How often it writes its heartbeat |
| `PROCESS_MANAGER__<profile>__<process>__JOBS__HEARTBEAT_STALE_AFTER_MS` | How old a heartbeat may be before it reads as no worker |
| `PROCESS_MANAGER__<profile>__<process>__JOBS__LOG_RETENTION_RUNS` | How many runs of a kind keep their log lines |
| `PROCESS_MANAGER__<profile>__<process>__JOBS__LOG_RETENTION_LINES` | How many lines of the OpenCTI stream it keeps |
| `PROCESS_MANAGER__<profile>__<process>__JOBS__RUN_RETENTION_DAYS` | How many days a finished run is kept |

The `JOBS__` keys of a process replace the whole `jobs` block for it: a key it leaves out takes its default, not the value of the variable without the prefix. Set them all under the prefix, as the worker's `PROCESS_MANAGER__SPLIT__WORKER__JOBS__ENABLED=true` with `PROCESS_MANAGER__SPLIT__WORKER__JOBS__MAX_CONCURRENT=8`.

## The health check

The process manager answers the health check on port 8444 (`HEALTHCHECK_HTTP_SERVER__PORT`), apart from the API. Each process sends it a heartbeat every 5 seconds with its state: `FORKING`, `BOOTING`, `STARTING`, `RUNNING`, `STOPPING`, `STOPPED`, `EXITED` or `ERROR`.

The health check answers `200` with `{"healthy":true,...}` when every process is `RUNNING` and has sent a heartbeat in the last 60 seconds (`PROCESS_MANAGER__HEARTBEAT_TIMEOUT`, in milliseconds). Otherwise it answers `503`, with `"healthy":false`. Either way, `childrenHealthStates` lists each process with its state and its last heartbeat.

It reports the processes, not the services behind them. A process that cannot reach PostgreSQL at start tries 30 times, a second apart, then exits: the process manager stops the container. XTM One out of reach leaves OpenCRQ healthy: users see a message instead, as [XTM One](/deploy/configure/opencrq/#xtm-one) describes.

xtm-docker calls `http://localhost:8444/health` every 10 seconds, and allows 120 seconds for the first start. A Kubernetes liveness or readiness probe can call the same address.

## The ports it listens on

| Port | Served by | What for | Exposure |
| --- | --- | --- | --- |
| 3002 (`HTTP_SERVER__PORT`) | The HTTP process | The interface, the API, the sign-in through XTM One, and the calls XTM One makes to OpenCRQ | Published by xtm-docker on port 8082 of the host, behind your reverse proxy |
| 8444 (`HEALTHCHECK_HTTP_SERVER__PORT`) | The process manager | The health check | Inside the stack; not published by xtm-docker |
| 14269 | The process manager | The Prometheus endpoint, `/metrics`, only when it is turned on. Its port is a setting of [Observability metrics](/deploy/operate/metrics/) | Never public: the endpoint asks for no credentials |

Every port, address and token of the stack is in [Network and ports](/deploy/plan/network/).

## What OpenCRQ calls

Each call leaves from one process. The worker makes every connector call, so a firewall rule for a connector applies to the whole container.

| Destination | Host | From | When |
| --- | --- | --- | --- |
| XTM One | `XTM_ONE__URL`, such as `http://xtm-one:4000` | HTTP process | Always: it registers at start and every 5 minutes, signs users in, and relays the questions of Ask Ariane |
| The default threat set of OpenCRQ | `raw.githubusercontent.com`, the file `default-threats-v1.json` of OpenCRQ's dataset repository | Worker | For every tenant, as soon as the worker finds one that has not loaded it, then every day at 06:00 UTC. `CONNECTORS__DEFAULT_THREATS__DATASET_URL` points it to a mirror, or, empty, turns it off |
| CISA KEV | `www.cisa.gov` | Worker | Once a tenant administrator binds a service account to it in **Integrations**, then every day at 06:00 UTC |
| FIRST EPSS | `api.first.org` | Worker | The same |
| NIST NVD | `services.nvd.nist.gov`, and `cveawg.mitre.org` for the vendor's remediation of each CVE | Worker | The same, with or without an API key |
| Microsoft Defender | `login.microsoftonline.com` and `graph.microsoft.com` | Worker | For a tenant whose credentials are set, once a service account is bound; every 6 hours |
| Drata | `public-api.drata.com` by default, or the URL set for the tenant | Worker | The same; every day at 06:00 UTC |
| OpenAEV | The URL set for the tenant, such as `http://openaev:8080` | Worker | For a tenant whose URL and token are set, with a service account bound, which OpenCRQ binds itself at start when nobody did; every hour |
| OpenCTI | The URL set for the tenant, such as `http://opencti:8080` | Worker | The same, but not on a schedule: the worker keeps a Live Stream open and reads it as server-sent events |
| Filigran | `telemetry.ocrq.filigran.io` | Both processes | The anonymous usage telemetry, on by default. See [Check or stop the usage telemetry sent to Filigran](/opencrq/administer/telemetry/) |

The CVE sources, the default threat set and Filigran's collector are on the internet; the other destinations are where you run them. At each start, the worker also creates the Live Stream `OpenCRQ threat feed` in the OpenCTI of each connected tenant, or puts its filter back: see [Link OpenCRQ to OpenCTI and OpenAEV](/deploy/configure/connect-modules/#link-opencrq-to-opencti-and-openaev). A tenant administrator can change each schedule: the [connectors reference](/reference/opencrq/connectors/) says what each connector brings. The credentials of each tenant are in [Connectors of each tenant](/deploy/configure/opencrq/#connectors-of-each-tenant).

## Observability

OpenCRQ sends nothing to your monitoring by default, and xtm-docker turns nothing on. Each transport has its own switch, and none depends on another:

- **OpenTelemetry (OTLP).** Every process, the process manager included, pushes its own traces, metrics and logs to your collector over gRPC. Each signal carries the name of its process in `process.name`: `pm`, `http` or `worker`.
- **Prometheus.** The process manager serves `/metrics` on port 14269. On each scrape, it asks every process for its metrics and merges them into one document, with the process manager's own metrics on the processes it watches. A collector is not needed for this path.
- **Pyroscope.** Every process sends its CPU and memory profiles to your Pyroscope server, tagged with its process name.

The settings, the metrics and the alerts worth setting are in [Observability metrics](/deploy/operate/metrics/). The usage telemetry OpenCRQ sends to Filigran is separate, and on by default: see [Check or stop the usage telemetry sent to Filigran](/opencrq/administer/telemetry/).

:::example[Open the firewall for Grand Meridian's OpenCRQ]
Example CERT runs the XTM stack for its member hotel chains, with OpenCRQ at `https://opencrq.example.com` for Grand Meridian Hotels. Its platform engineer checks the processes after the first start:

```bash
docker compose exec opencrq wget -qO- http://localhost:8444/health
```

```json
{"healthy":true,"childrenHealthStates":[{"processName":"http","status":"RUNNING","lastHeartbeat":"2026-01-14T09:30:05.112Z"},{"processName":"worker","status":"RUNNING","lastHeartbeat":"2026-01-14T09:30:04.987Z"}]}
```

The CERT's egress firewall lets the `opencrq` container reach `raw.githubusercontent.com` for the default threat set. Grand Meridian's tenant administrator turns on the CVE sources, so the engineer also allows `www.cisa.gov`, `api.first.org`, `services.nvd.nist.gov` and `cveawg.mitre.org`. OpenCTI and OpenAEV run in the same stack and need no rule. Grand Meridian does not use Microsoft Defender or Drata: their hosts stay closed. The CERT turns the usage telemetry off, so `telemetry.ocrq.filigran.io` stays closed too. From the next morning, the three CVE sources run every day at 06:00 UTC, and the default threats are checked for a new file at the same time.
:::

## Find which part to look at

| What you see | Likely cause | What to do |
| --- | --- | --- |
| The container restarts again and again | A process exits at start: PostgreSQL out of reach, a setting refused, the S3 bucket missing | Read `docker compose logs opencrq`: the first error names the process and the cause |
| The health check answers `503` while the container runs | A process is still starting, or stopped sending heartbeats | Read `childrenHealthStates`: the process that is not `RUNNING`, or whose heartbeat is old |
| **No worker is answering. Queued runs will not advance.** | The worker is not running, or cannot write to the database | Check the `worker` line of the health check, then the logs |
| A connector's runs fail at once | Its host is blocked, or its credentials are refused | The run's log in **Integrations**, and the outbound rules of the table above |
| Nobody can sign in, but the health check says healthy | XTM One does not answer the HTTP process | [When the sign-in does not work](/deploy/configure/opencrq/#sign-in-to-opencrq) |

## What's next

- [The XTM stack](/deploy/plan/architecture/): OpenCRQ next to XTM One, OpenCTI and OpenAEV, and the services they share.
- [Configure OpenCRQ](/deploy/configure/opencrq/): the settings of the database, the file storage, XTM One, the connectors and the processes.
- [Network and ports](/deploy/plan/network/): every port and every outbound connection of the stack.
- [Observability metrics](/deploy/operate/metrics/): turn on the Prometheus endpoint, OTLP and Pyroscope.
- [Background processes](/reference/opencrq/processes/): every job the worker runs, and when.
- [How OpenCRQ quantifies risk](/opencrq/concepts/how-risk-is-quantified/): what those jobs compute.
