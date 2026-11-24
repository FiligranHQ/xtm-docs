---
title: Configure OpenCRQ
description: Set the database, file storage, XTM One, first tenant, per-tenant connector, telemetry and HTTP settings of an OpenCRQ deployment, with their variables and what OpenCRQ does when one is wrong.
doc_type: how-to
products:
  - opencrq
personas: [platform-engineer]
owner: docs-opencrq
last_reviewed: 2026-11-20
---

This page sets the configuration an OpenCRQ deployment needs: where its database and file storage are, how it reaches XTM One, which tools each tenant reads, what it reports, and how it listens. Every setting is an environment variable of the `opencrq` service, read when the platform starts. [Install OpenCRQ](/deploy/install/opencrq/) sets the minimum; come back here to connect tenants to OpenCTI and OpenAEV, size the database connections, turn the usage telemetry off, or move a service out of the stack. The settings not covered here, such as the background jobs, are in the [OpenCRQ configuration](/reference/opencrq/configuration/) reference.

::diagram{id=deploy-configure-opencrq}

## How OpenCRQ reads its settings

- **A variable per key.** The key in upper case, with `__` between its levels: `POSTGRES__POOL_MAX` is `postgres.pool_max`. Lists take commas.
- **Checked at start.** OpenCRQ checks the whole configuration against its schema when it starts. A value of the wrong type, a required key left blank, or a block set halfway stops the platform with a message that names the key. Nothing is reloaded while it runs: after a change, recreate the container with `docker compose up -d`.
- **Defaults in the image.** The image ships the defaults of the background jobs, the recomputations and the processes. A variable overrides its key.

## Database

| Variable | Default | What it sets |
| --- | --- | --- |
| `POSTGRES__HOST`, `POSTGRES__PORT` | Required | Where PostgreSQL listens |
| `POSTGRES__DATABASE` | Required | The database OpenCRQ owns. It creates its tables there at each start, through its migrations |
| `POSTGRES__USER`, `POSTGRES__PASSWORD` | Required | The role it connects as, the owner of the database |
| `POSTGRES__POOL_MAX` | `20`, at least `10` | The connections each process may open in its pool |

The migrations create the `unaccent` extension. The owner of the database can create it, but a managed service that allows extensions one by one refuses any it does not list, and OpenCRQ then stops at its first start: on Azure Database for PostgreSQL, add `unaccent` to the `azure.extensions` server parameter first.

Count the connections against PostgreSQL's `max_connections`, shared with the other products of the instance. With the default `split` profile, OpenCRQ runs two processes, each with its own pool of `POSTGRES__POOL_MAX` connections; the worker also keeps 5 connections for the job runtime and one per channel of database notifications it listens to. Plan about 50 connections with the defaults.

Give PostgreSQL at least 1 GB of shared memory (`shm_size: 1gb` in Docker Compose, a memory `emptyDir` on Kubernetes). With the 64 MB a container gets by default, a large query fails with `could not resize shared memory segment`, and the background job that ran it fails with it.

## File storage

OpenCRQ keeps the files people upload, such as the STIX bundles of the **Threats** page, in an S3-compatible bucket, each tenant under its own prefix. The file storage is optional:

| What you set | What OpenCRQ does |
| --- | --- |
| No `FILE_STORAGE__*` variable | Starts without file storage: on the **Threats** page, **Analyze new bundle** is greyed out, with **File storage is not configured** |
| The block, with a wrong endpoint, wrong keys or a bucket that does not exist | Stops at start with a message: `File storage is misconfigured — fix FILE_STORAGE__* before restarting` |
| The block, right, with the storage briefly unreachable | Starts, logs the error, and connects at the first upload |

| Variable | Default | What it sets |
| --- | --- | --- |
| `FILE_STORAGE__ENDPOINT` | Required in the block | The S3 API, such as `http://minio:9000` |
| `FILE_STORAGE__ACCESS_KEY`, `FILE_STORAGE__SECRET_KEY` | Required in the block | The keys OpenCRQ signs in with |
| `FILE_STORAGE__BUCKET` | `opencrq` | The bucket. Create it before the first start: OpenCRQ never creates it in production |
| `FILE_STORAGE__REGION` | `eu-west-3` | The region the S3 client sends. Any value works for MinIO-compatible storage |

## XTM One

OpenCRQ does not start without XTM One. It registers with XTM One when it starts and every 5 minutes, which brings back its edition, and XTM One calls it as the person who asks when an AI agent works on its data.

| Variable | What it sets |
| --- | --- |
| `XTM_ONE__URL` | Where OpenCRQ reaches XTM One. An internal address works, such as `http://xtm-one:4000` |
| `XTM_ONE__TOKEN` | The registration token, the value of `PLATFORM_REGISTRATION_TOKEN` on XTM One |
| `XTM_ONE__PLATFORM_URL` | The public URL of OpenCRQ, such as `https://opencrq.example.com`. It names OpenCRQ in the tokens it signs and in its registration |
| `XTM_ONE__ENABLED` | Nothing: XTM One cannot be turned off. Leave it out; any value other than `true` stops the platform |

Keep these rules:

- **Your own registration token.** Set it before the first start: OpenCRQ registers with XTM One with this token, and the registration brings both the sign-in through XTM One and the XTM license. XTM One refuses to register a new platform with its published default token, `xtm-default-registration-token`, with HTTP 401: it keeps only the heartbeat of a platform registered earlier, and withholds the XTM license from it.
- **Public and internal URLs.** OpenCRQ reads XTM One's public identity from `/xtm/auth/metadata` on `XTM_ONE__URL`, once an hour, and uses it for the tokens and for the links users follow to XTM One. When `XTM_ONE__PLATFORM_URL` does not resolve inside Docker, set `OPENCRQ_API_URL` on the XTM One services to the internal address of OpenCRQ, as [Install OpenCRQ](/deploy/install/opencrq/) does.
- **The edition.** The Enterprise Edition of OpenCRQ comes from the XTM license that XTM One returns at registration, which OpenCRQ verifies, or, until it expires, from a Filigran license installed on the platform before 17 November 2026. Each verified answer of XTM One grants it for 7 days, in memory: after a restart, a platform without an installed license is in the Community Edition until the next registration succeeds. The Enterprise Edition brings more than one tenant and the Ask Ariane assistant.
- **When XTM One is down.** OpenCRQ keeps serving and its health check stays healthy. Every user sees a message saying XTM One is not answering, its registration is refused, or its answer cannot be read, and Ask Ariane is unavailable until a registration succeeds.

## Connectors of each tenant

The credentials of a connector are set per tenant, never in the interface: two tenants can read two different OpenCTI platforms. A variable takes the identifier of the tenant, shown as **Identifier** in :menu[Settings > Parameters]: `CONNECTORS__<NAME>__<tenant identifier>__<FIELD>`.

At its first start, on a platform without any tenant, OpenCRQ creates one, so that you can write its connector variables before that start:

| Variable | Default | What it sets |
| --- | --- | --- |
| `INITIAL_TENANT__ID` | `01a043da-ff47-72c2-aaa4-7da56d85c83e` | The identifier of that first tenant, a UUID |
| `INITIAL_TENANT__NAME` | `Default` | Its name, which administrators can change afterwards |

`INITIAL_TENANT__NAME` counts only while the platform holds no tenant: once one exists, changing it renames nothing. `INITIAL_TENANT__ID` is read at every start too: it names the tenant of the `APP__ADMIN__TOKEN` account and of the accounts `/api/xtm/peer-tokens` creates, as [Sign in the first time](/deploy/configure/first-access/#opencrq) describes. Changing it later creates no tenant. If the creation fails, OpenCRQ logs `Creating the initial tenant failed, booting without it` and starts without it. Every other tenant gets its identifier when an administrator creates it: see [Run several tenants in one platform](/opencrq/administer/tenants/).

| Connector | Variables, after `CONNECTORS__` and the tenant identifier | Without them |
| --- | --- | --- |
| CISA KEV, FIRST EPSS | None | No variable needed. Off until a tenant administrator binds a service account in **Integrations** |
| NIST NVD | `NIST__…__API_KEY` | Off until a tenant administrator binds a service account in **Integrations**. Without a key, it runs at NVD's lower rate limit |
| OpenCTI | `OPENCTI__…__URL`, `OPENCTI__…__TOKEN` | Inactive for the tenant |
| OpenAEV | `OPENAEV__…__URL`, `OPENAEV__…__TOKEN` | Inactive for the tenant |
| Microsoft Defender, through Microsoft Graph | `MICROSOFT_GRAPH__…__AZURE_TENANT_ID`, `__CLIENT_ID`, `__CLIENT_SECRET` | Inactive for the tenant |
| Drata | `DRATA__…__API_KEY`, and `__URL` (`https://public-api.drata.com` by default; `https://public-api.eu.drata.com` for an EU account), `__WORKSPACE_ID` (`1` by default) | Inactive for the tenant |

The OpenCTI and OpenAEV connectors come up enabled once their variables are set. At each start, OpenCRQ creates the Live Stream `OpenCRQ threat feed` in OpenCTI, or puts its filter back, and binds each of the two connectors that nobody configured to a service account. A binding a tenant administrator made is kept. See [Link OpenCRQ to OpenCTI and OpenAEV](/deploy/configure/connect-modules/#link-opencrq-to-opencti-and-openaev). Another connector with its variables set still waits for a tenant administrator to give it a service account in **Integrations**: see [Let integrations sign in with service accounts](/opencrq/administer/service-accounts/). The [connectors reference](/reference/opencrq/connectors/) says what each one brings and when it runs, and the connector pages say what each tool must grant: [Connect OpenCTI with a Live Stream](/opencrq/use/sources/opencti/), [Import OpenAEV results](/opencrq/use/sources/openaev/).

## Telemetry

The `TELEMETRY__*` variables form two groups. `TELEMETRY__USAGE__*` sets the anonymous usage telemetry OpenCRQ sends to Filigran. `TELEMETRY__OBSERVABILITY__*` sets the observability metrics, traces, logs and profiles it sends to your own tools: see [Observability metrics](/deploy/operate/metrics/).

OpenCRQ accepts no other `TELEMETRY__*` variable. An unknown one stops the platform at start with `Unrecognized key` and the name of the key under `telemetry`, rather than leaving a transport off without a word. The names of earlier builds count as unknown, so rename them:

| Name in earlier builds | Name now |
| --- | --- |
| `TELEMETRY__TAGS` | `TELEMETRY__USAGE__TAGS` |
| `TELEMETRY__APPLICATION_NAME` | `TELEMETRY__OBSERVABILITY__SERVICE_NAME` |
| `TELEMETRY__TRACES__ENABLED`, `TELEMETRY__TRACES__EXPORTER_OTLP` | `TELEMETRY__OBSERVABILITY__OTLP__TRACES__ENABLED`, `TELEMETRY__OBSERVABILITY__OTLP__TRACES__EXPORTER` |
| `TELEMETRY__LOGS__ENABLED`, `TELEMETRY__LOGS__EXPORTER_OTLP` | `TELEMETRY__OBSERVABILITY__OTLP__LOGS__ENABLED`, `TELEMETRY__OBSERVABILITY__OTLP__LOGS__EXPORTER` |
| `TELEMETRY__METRICS__OTLP__*` | `TELEMETRY__OBSERVABILITY__OTLP__METRICS__*` |
| `TELEMETRY__METRICS__PROMETHEUS__*` | `TELEMETRY__OBSERVABILITY__PROMETHEUS__*` |
| `TELEMETRY__PYROSCOPE__*` | `TELEMETRY__OBSERVABILITY__PYROSCOPE__*` |

| Variable | Default | What it sets |
| --- | --- | --- |
| `TELEMETRY__USAGE__ENABLED` | `true` | `false` turns the usage telemetry off: OpenCRQ never contacts Filigran's collector, and logs `Anonymous usage telemetry is disabled by TELEMETRY__USAGE__ENABLED=false, nothing is sent` |
| `TELEMETRY__USAGE__TAGS` | Empty | Tags of the deployment, separated by commas, such as `saas,eu-west`, sent with the usage data. OpenCRQ trims, lowercases, deduplicates and sorts them |

The usage telemetry sends counts over the whole platform, such as the number of tenants, users, risk scenarios and simulations run, and whether it runs in the Enterprise Edition: no names and no content. The image sends them every 6 hours to `https://telemetry.ocrq.filigran.io`. When the collector does not answer at start, as on a platform without internet access, OpenCRQ logs `Telemetry endpoint is not reachable, anonymous usage telemetry is disabled` and sends nothing until its next start.

## HTTP and processes

| Variable | Default | What it sets |
| --- | --- | --- |
| `HTTP_SERVER__HOST`, `HTTP_SERVER__PORT` | None | Where the interface and the API listen in the container, such as `0.0.0.0` and `3002`. Required for the interface and the API: without them, OpenCRQ starts with no HTTP server |
| `HEALTHCHECK_HTTP_SERVER__PORT` | Required | The port of the health check, such as `8444`. It answers `200` with `{"healthy":true,...}` while every process runs, and `503` otherwise |
| `HTTP_SERVER__PUBLIC_PORT`, `HEALTHCHECK_HTTP_SERVER__PUBLIC_PORT` | None | The ports published on the host, for the logs only |
| `HTTP_SERVER__MAX_RESPONSE_ITEMS` | `20000` | The most items one API call returns. Above it, the call is refused with `PAYLOAD_TOO_LARGE` |
| `PROCESS_MANAGER__PROFILE` | `split` | `split` runs the HTTP server and the background worker as two processes. `monolith` runs one process, for development only |
| `PROCESS_MANAGER__HEARTBEAT_TIMEOUT` | `60000` | The milliseconds without a heartbeat from a process after which the health check answers unhealthy |

Each process of a profile takes its own settings under `PROCESS_MANAGER__<profile>__<process>__`. The worker of the `split` profile has a heap of 4,096 MB (`PROCESS_MANAGER__SPLIT__WORKER__MEMORY`) and the HTTP process 512 MB (`PROCESS_MANAGER__SPLIT__HTTP__MEMORY`): raise the worker's for a large estate, and the container's memory limit with it.

:::warning
Leave `PLATFORM_SEED__ENABLED` and `PLATFORM__RESET_ON_VERSION_CHANGE` unset in production. The first loads demonstration data; the second deletes the whole database when the version of OpenCRQ changes.
:::

:::example[Connect Grand Meridian's tenant to OpenCTI and OpenAEV]
Example CERT runs OpenCRQ for Grand Meridian Hotels, whose tenant identifier is `01a043da-ff47-72c2-aaa4-7da56d85c83e`: OpenCRQ created that tenant at its first start. Its platform engineer connects the tenant to the CERT's OpenCTI, to read the threats against the hospitality sector, to its OpenAEV, to read the simulation results of the hotels, and to the NVD with an API key. The tokens live in `.env`, as `GRAND_MERIDIAN_OPENCTI_TOKEN`, `GRAND_MERIDIAN_OPENAEV_TOKEN` and `NVD_API_KEY`. In `docker-compose.override.yml`, under the `opencrq` service, these variables replace the ones xtm-docker sets for the same tenant with the admin tokens:

```yaml
    environment:
      - CONNECTORS__OPENCTI__01a043da-ff47-72c2-aaa4-7da56d85c83e__URL=http://opencti:8080
      - CONNECTORS__OPENCTI__01a043da-ff47-72c2-aaa4-7da56d85c83e__TOKEN=${GRAND_MERIDIAN_OPENCTI_TOKEN}
      - CONNECTORS__OPENAEV__01a043da-ff47-72c2-aaa4-7da56d85c83e__URL=http://openaev:8080
      - CONNECTORS__OPENAEV__01a043da-ff47-72c2-aaa4-7da56d85c83e__TOKEN=${GRAND_MERIDIAN_OPENAEV_TOKEN}
      - CONNECTORS__NIST__01a043da-ff47-72c2-aaa4-7da56d85c83e__API_KEY=${NVD_API_KEY}
```

The user of `GRAND_MERIDIAN_OPENCTI_TOKEN` holds **Manage data sharing** in OpenCTI. After `docker compose up -d`, the OpenCTI and OpenAEV connectors of the tenant come up enabled in **Integrations**, OpenCTI holds the Live Stream `OpenCRQ threat feed`, the threats of Velvet Harbor and Ember Tide stream in, and the hotels' simulation results follow every hour. The NIST NVD connector runs once a tenant administrator binds a service account to it in **Integrations**.
:::

## What's next

- [OpenCRQ configuration](/reference/opencrq/configuration/), every key with its variable, type and default, including the background jobs.
- [Observability metrics](/deploy/operate/metrics/), to scrape the metrics of OpenCRQ and send its traces, logs and profiles to your tools.
- [Background processes](/reference/opencrq/processes/), what the worker runs and when.
- [Connect the modules to each other](/deploy/configure/connect-modules/), for the registration token XTM One shares with every platform.
