---
title: Configure OpenCRQ
description: Finish the setup of OpenCRQ after the install and sign in to its first tenant, then set its database, storage, XTM One, connector, telemetry and HTTP settings.
doc_type: how-to
products:
  - opencrq
personas: [platform-engineer]
owner: docs-opencrq
last_reviewed: 2026-11-20
---

This page sets the configuration an OpenCRQ deployment needs: where its database and file storage are, how it reaches XTM One, which tools each tenant reads, what it reports, and how it listens. Every setting is an environment variable of the `opencrq` service, read when the platform starts. OpenCRQ starts with the rest of the XTM stack, as [Install with Docker Compose](/deploy/install/docker-compose/) describes: [Finish the setup after the install](#finish-the-setup-after-the-install) gives it its public URL, then XTM One's administrator signs in to its first tenant. Come back here to connect tenants to OpenCTI and OpenAEV, size the database connections, turn the usage telemetry off, or move a service out of the stack. The settings not covered here, such as the background jobs, are in the [OpenCRQ configuration](/reference/opencrq/configuration/) reference.

::diagram{id=deploy-configure-opencrq}

## What xtm-docker runs for OpenCRQ

::diagram{id=deploy-install-opencrq}

`modules/opencrq.yml` adds three services to the stack:

| Service | What it does | When |
| --- | --- | --- |
| `opencrq-db-init` | Creates the role of OpenCRQ, `OPENCRQ_POSTGRES_USER`, not a superuser, and the `opencrq` database it owns, in the shared `pgsql` instance. OpenCRQ connects as that role | At each start; it sets the role's password from `.env` again, and changes nothing else once they exist |
| `opencrq-bucket-init` | Creates the OpenCRQ bucket in the shared S3 storage, with the AWS command line interface: in production, OpenCRQ never creates it | At each start; it changes nothing once the bucket exists |
| `opencrq` | OpenCRQ itself, the image `filigran/opencrq`, public on Docker Hub. It listens on port 3002, published on port 8082 of the host, and answers its health check on port 8444 inside the container | Once both jobs succeeded and XTM One is healthy |

The stack also gives the shared PostgreSQL 1 GB of shared memory, which the large queries of OpenCRQ need, and lets XTM One reach OpenCRQ inside the stack (`OPENCRQ_API_URL=http://opencrq:3002`).

| `.env` setting | Default | What it sets |
| --- | --- | --- |
| `OPENCRQ_EXTERNAL_SCHEME`, `OPENCRQ_HOST`, `OPENCRQ_PORT` | `http`, `localhost`, `8082` | The public URL of OpenCRQ. It is also its identity towards XTM One: the tokens OpenCRQ signs name it, and the tokens XTM One sends must be addressed to it |
| `OPENCRQ_POSTGRES_USER`, `OPENCRQ_POSTGRES_PASSWORD` | `opencrq`, drawn by `init-env.sh` | The PostgreSQL role of OpenCRQ, which must differ from `POSTGRES_USER` and `XTM_ONE_POSTGRES_USER`. Set the role once: changed after the first start, the `opencrq` database and its tables stay owned by the old role. Without the password, `opencrq-db-init` fails and OpenCRQ does not start |
| `OPENCRQ_ADMIN_TOKEN` | A UUID version 4, drawn by `init-env.sh` | The admin token of the OpenCRQ API (`APP__ADMIN__TOKEN`): see [Sign in the first time and keep an emergency account](/deploy/configure/first-access/#opencrq). Another value stops OpenCRQ at start |
| `OPENCRQ_S3_BUCKET` | `opencrq` | The bucket of OpenCRQ in the shared S3 storage |
| `OPENCRQ_TENANT_ID` | Empty, read as `01a043da-ff47-72c2-aaa4-7da56d85c83e` | The identifier of the tenant the OpenCTI and OpenAEV connectors of the `opencrq` service feed |

OpenCRQ registers with XTM One with `PLATFORM_REGISTRATION_TOKEN`, as OpenCTI and OpenAEV do. Its users sign in through XTM One: OpenCRQ keeps no password, and has no sign-in form of its own. OpenCRQ also reaches the internet: `telemetry.ocrq.filigran.io` for the anonymous usage telemetry, on by default; `raw.githubusercontent.com` for the default threats, once a tenant loads them; and the CVE sources (NIST, FIRST and CISA), once a tenant administrator turns them on. See [Network and ports](/deploy/plan/network/#outbound-connections).

At its first start, OpenCRQ creates its first tenant, `Default`, with the identifier `01a043da-ff47-72c2-aaa4-7da56d85c83e`. The connections to OpenCTI and OpenAEV are set per tenant, by the tenant's identifier, and xtm-docker sets them for the tenant of `OPENCRQ_TENANT_ID`, that same identifier by default: the first tenant is connected from that start, as [Connect the first tenant](#connect-the-first-tenant) shows.

OpenCRQ adds two processes to the host: the HTTP server, with a heap of 512 MB, and the worker that runs the background jobs, with a heap of 4,096 MB. Plan about 5 GB for OpenCRQ on top of what the rest of the stack needs, and about 50 more PostgreSQL connections at peak: see [Requirements](/deploy/plan/requirements/) and [Sizing](/deploy/plan/sizing/).

## Finish the setup after the install

OpenCRQ starts with the rest of the stack. These steps give it its public URL and check that it registered with XTM One. You need:

- The stack installed and running as [Install with Docker Compose](/deploy/install/docker-compose/) describes, with `PLATFORM_REGISTRATION_TOKEN` set to a value of your own: see [Connect the modules to each other](/deploy/configure/connect-modules/#register-the-platforms-with-xtm-one). Run the commands of this page from its folder.
- The host name users will type, such as `opencrq.example.com`, served over HTTPS by your reverse proxy: see [Publish the platforms over HTTPS](/deploy/install/reverse-proxy/). XTM One sends users back to OpenCRQ at that URL after they sign in, and accepts it only over HTTPS, or on `localhost` for a trial. The session cookie of OpenCRQ is `Secure`: over plain HTTP, browsers keep it on `localhost` only.
- The credentials of XTM One's administrator, the account of `XTM_ONE_ADMIN_EMAIL` in `.env`: until you add members to the first tenant, it is the only account that can open OpenCRQ.

:::steps
1. Check the public URL of OpenCRQ in `.env`, which [Configure the stack](/deploy/install/docker-compose/#configure-the-stack) sets with the other platforms.

   ```bash
   grep -E '^OPENCRQ_(EXTERNAL_SCHEME|HOST|PORT)=' .env
   ```

   :::verify
   `OPENCRQ_EXTERNAL_SCHEME=https`, `OPENCRQ_HOST=opencrq.example.com` and `OPENCRQ_PORT=443`. `OPENCRQ_PORT` is the port of the public URL, 443 behind the reverse proxy: the host still publishes OpenCRQ on port 8082. After a change, run `docker compose up -d`.
   :::

2. Check that OpenCRQ runs. Its first start creates its tables and loads the reference catalogues: allow a minute or two. The stack runs `filigran/opencrq:latest`, the latest release of OpenCRQ, like the other modules, unless you [pinned the image versions](/deploy/install/docker-compose/#pin-the-image-versions).

   ```bash
   docker compose ps opencrq-db-init opencrq-bucket-init opencrq --all --format '{{.Service}} {{.State}} {{.Health}}'
   docker compose exec opencrq wget -qO- http://localhost:8444/health
   ```

   :::verify
   Both jobs show `exited`, and `opencrq` shows `running healthy`. Then `{"healthy":true,...}`, with the state of each process: each line `childrenHealthStates` lists says `RUNNING`.
   :::

3. Check that OpenCRQ registered with XTM One.

   ```bash
   docker compose logs opencrq | grep "Registered with XTM One"
   ```

   The line names the version of XTM One, and `eeEnabled` says whether the XTM license covers OpenCRQ. OpenCRQ registers when it starts, then every 5 minutes. A `XTM One registration failed` line with HTTP 401 means that XTM One refused the token. Either OpenCRQ and XTM One hold different values, as after a `docker compose restart` that kept the old ones, or the token is the published default `xtm-default-registration-token`, which cannot register a new platform. Set a value of your own in `PLATFORM_REGISTRATION_TOKEN` and run `docker compose up -d`.

4. Put the reverse proxy in front of port 8082 of the host, for `opencrq.example.com`. Sign-in goes through this URL: XTM One sends the browser back to `https://opencrq.example.com/api/auth/xtm/callback`, and a sign-in started at another address first moves the browser there.

   :::verify
   ```bash
   docker compose logs opencrq | grep "Sign-in with XTM One stays off"
   ```
   The command prints nothing. A line means the public URL of OpenCRQ is neither HTTPS nor `localhost`: XTM One would refuse it, so nobody can sign in. Fix `OPENCRQ_EXTERNAL_SCHEME` and run `docker compose up -d` again.
   :::
:::

## Sign in to OpenCRQ

OpenCRQ has no first-administrator form. XTM One's administrator is OpenCRQ's super administrator: every right on the platform and in every tenant, given again at each sign-in, without being a member of any group. They sign in to the first tenant and give it the name of the organization OpenCRQ prices risk for.

:::steps
1. Open `https://opencrq.example.com` in a browser and click **Sign in with XTM One**.
2. Sign in to XTM One with the account of `XTM_ONE_ADMIN_EMAIL`. If XTM One already has a session in this browser, it sends you back at once.
3. OpenCRQ opens in the `Default` tenant, on its **Get started** wizard the first time.
4. Open :menu[Settings > Security], choose **Platform** at the top of the menu, and open **Tenants**.
5. Choose **Rename** in the actions menu of the `Default` card, enter the **Name** of the organization, and click **Save**.
:::

::screenshot{id=opencrq-install-first-tenant alt="The Tenants page of the platform shows the card of the tenant OpenCRQ created at its first start, with its number of members."}

To name the tenant from the first start instead, set `INITIAL_TENANT__NAME` on the `opencrq` service before that start: see [Connectors of each tenant](#connectors-of-each-tenant).

The tenant starts with an **Admin** group and a **Connectors** group, and with no member: you reach it as super administrator, tagged **Super Administrator** in its **Users** list, without belonging to it. Add its people from :menu[Settings > Security > Users] with **Add members**: see [Give people access with users, groups and roles](/opencrq/administer/users-and-roles/). Anyone else who signs in before then reads **You don't have access to OpenCRQ yet**. A Community Edition platform serves this one tenant; more require the Enterprise Edition: see [Run several tenants in one platform](/opencrq/administer/tenants/).

When the sign-in does not work:

| OpenCRQ shows | What it means |
| --- | --- |
| **XTM One is unavailable**, with **Try again** | XTM One does not answer OpenCRQ, or has given it no sign-in client yet. There is no other way in. Check that XTM One is healthy, that `PLATFORM_REGISTRATION_TOKEN` is not the published default (XTM One then creates no sign-in client), and the reverse proxy step of [Finish the setup after the install](#finish-the-setup-after-the-install). Then click **Try again** |
| **You are not signed in**, with the reason | XTM One refused or interrupted the sign-in, or it was cancelled. Click **Sign in with XTM One** again |
| **You don't have access to OpenCRQ yet** | You signed in with another XTM One account than its administrator. Click **Sign out**, sign out of XTM One too, and sign in again with the account of `XTM_ONE_ADMIN_EMAIL` |

## Connect the first tenant

xtm-docker connects the tenant of `OPENCRQ_TENANT_ID` to the stack's OpenCTI and OpenAEV, with their admin tokens: by default the first tenant, `01a043da-ff47-72c2-aaa4-7da56d85c83e`. You have nothing to set. Both connectors come up enabled at the first start: OpenCRQ creates their service accounts, `OpenCTI (deployment)` and `OpenAEV (deployment)`, and creates in OpenCTI the Live Stream `OpenCRQ threat feed`, which carries the threats and their relationships (`targets`, `uses`, `attributed-to`). The OpenCTI token needs **Manage data sharing** or **Bypass all capabilities** for that, which the admin token holds: see [Link OpenCRQ to OpenCTI and OpenAEV](/deploy/configure/connect-modules/#link-opencrq-to-opencti-and-openaev). The tenant's **Integrations** shows both connectors enabled.

A tenant's identifier is in its :menu[Settings > Parameters], under **Identifier**. If you set `INITIAL_TENANT__ID` on the `opencrq` service, set `OPENCRQ_TENANT_ID` in `.env` to the same value: OpenCRQ skips a connection that names a tenant it does not hold. To connect another tenant, created in the Enterprise Edition, set the variables [Connectors of each tenant](#connectors-of-each-tenant) lists for its identifier, then run `docker compose up -d`.

The links OpenCRQ shows to OpenCTI objects open at the URL of the tenant's connection, `http://opencti:8080` in xtm-docker, which only the containers of the stack reach. The product switcher of OpenCRQ opens the public URLs of OpenCTI and OpenAEV, which xtm-docker gives it as `XTM__OPENCTI_URL` and `XTM__OPENAEV_URL`.

:::example[Bring OpenCRQ into service in Example CERT's stack]
Example CERT runs the XTM stack for its member hotel chains, with OpenCTI at `https://opencti.example.com`, XTM One at `https://xtm-one.example.com` and OpenCRQ at `https://opencrq.example.com`, for Grand Meridian Hotels and the other members. Once the stack runs, its platform engineer checks OpenCRQ:

```bash
grep -E '^OPENCRQ_(EXTERNAL_SCHEME|HOST|PORT)=' .env
docker compose logs opencrq | grep -E "Registered with XTM One|Sign-in with XTM One stays off"
```

`.env` holds the three values of the public URL, and the log of `opencrq` shows `Registered with XTM One`, and no `Sign-in with XTM One stays off` line. Once the reverse proxy publishes `opencrq.example.com`, the CERT's administrator, `admin@example.com` in XTM One (`XTM_ONE_ADMIN_EMAIL`), clicks **Sign in with XTM One** and lands in the `Default` tenant. They rename it `Grand Meridian Hotels` on the **Tenants** page. Its :menu[Settings > Parameters] shows the **Identifier** `01a043da-ff47-72c2-aaa4-7da56d85c83e`, the one `OPENCRQ_TENANT_ID` keeps when left empty. Its **Integrations** already show the OpenCTI and OpenAEV connectors enabled, and the OpenCTI one reads the Live Stream `OpenCRQ threat feed` that OpenCRQ created with the admin token. The administrator then adds Grand Meridian's risk manager with **Add members**.
:::

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

Count the connections against PostgreSQL's `max_connections`, shared with the other modules of the instance. With the default `split` profile, OpenCRQ runs two processes, each with its own pool of `POSTGRES__POOL_MAX` connections; the worker also keeps 5 connections for the job runtime and one per channel of database notifications it listens to. Plan about 50 connections with the defaults.

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
- **Public and internal URLs.** OpenCRQ reads XTM One's public identity from `/xtm/auth/metadata` on `XTM_ONE__URL`, once an hour, and uses it for the tokens and for the links users follow to XTM One. When `XTM_ONE__PLATFORM_URL` does not resolve inside Docker, set `OPENCRQ_API_URL` on the XTM One services to the internal address of OpenCRQ, as xtm-docker does ([What xtm-docker runs for OpenCRQ](#what-xtm-docker-runs-for-opencrq)).
- **The edition.** The Enterprise Edition of OpenCRQ comes from the XTM license that XTM One returns at registration, which OpenCRQ verifies, or, until it expires, from a Filigran license installed on the platform before 17 November 2026. Each verified answer of XTM One grants it for 7 days, in memory: after a restart, a platform without an installed license is in the Community Edition until the next registration succeeds. The Enterprise Edition brings more than one tenant and the Ask Ariane assistant.
- **When XTM One is down.** OpenCRQ keeps serving and its health check stays healthy. Every user sees a message saying XTM One is not answering, its registration is refused, or its answer cannot be read, and Ask Ariane is unavailable until a registration succeeds.

## Connectors of each tenant

The credentials of a connector are set per tenant, never in the interface: two tenants can read two different OpenCTI instances. A variable takes the identifier of the tenant, shown as **Identifier** in :menu[Settings > Parameters]: `CONNECTORS__<NAME>__<tenant identifier>__<FIELD>`.

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

## Where the data is

| Data | Where |
| --- | --- |
| Risk scopes, assets, scenarios, prices, settings | The `opencrq` database of `pgsql`, in the `pgsqlshareddata` volume |
| STIX bundles uploaded on the **Threats** page | The `opencrq` bucket of `minio`, in the `s3data` volume |

Back them up with the rest of the stack: see [Back up and restore](/deploy/operate/backup/). OpenCRQ keeps nothing else: the container can be recreated at any time.

## What's next

- [OpenCRQ configuration](/reference/opencrq/configuration/), every key with its variable, type and default, including the background jobs.
- [Observability metrics](/deploy/operate/metrics/), to scrape the metrics of OpenCRQ and send its traces, logs and profiles to your tools.
- [Background processes](/reference/opencrq/processes/), what the worker runs and when.
- [Connect the modules to each other](/deploy/configure/connect-modules/), for the registration token XTM One shares with every platform.
