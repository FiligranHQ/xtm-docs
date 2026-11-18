---
title: Install OpenCRQ
description: Start OpenCRQ with the XTM stack deployed with Docker Compose, where its database, its bucket and its registration with XTM One are set up for you, and create its first administrator.
doc_type: how-to
products:
  - opencrq
personas: [platform-engineer]
owner: docs-opencrq
last_reviewed: 2026-11-16
---

This page starts OpenCRQ, the cyber risk quantification module of the XTM platform, in an XTM stack installed with [FiligranHQ/xtm-docker](https://github.com/FiligranHQ/xtm-docker). xtm-docker ships OpenCRQ with the rest of the stack: its database in the shared PostgreSQL, its bucket in the shared S3 storage, its registration with XTM One and its connectors to OpenCTI and OpenAEV are set up for you. You give it its public URL. At the end, OpenCRQ answers at that URL and you have created its first administrator.

::diagram{id=deploy-install-opencrq}

## What xtm-docker runs for OpenCRQ

`modules/opencrq.yml` adds three services to the stack:

| Service | What it does | When |
| --- | --- | --- |
| `opencrq-db-init` | Creates the `opencrq` database in the shared `pgsql` instance. OpenCRQ connects to it as `POSTGRES_USER`, like OpenAEV | At each start; it changes nothing once the database exists |
| `opencrq-bucket-init` | Creates the OpenCRQ bucket in the shared S3 storage, with the AWS command line interface: in production, OpenCRQ never creates it | At each start; it changes nothing once the bucket exists |
| `opencrq` | OpenCRQ itself, the image `filigran/opencrq`, public on Docker Hub. It listens on port 3002, published on port 8082 of the host, and answers its health check on port 8444 inside the container | Once both jobs succeeded and XTM One is healthy |

The stack also gives the shared PostgreSQL 1 GB of shared memory, which the large queries of OpenCRQ need, and lets XTM One reach OpenCRQ inside the stack (`OPENCRQ_API_URL=http://opencrq:3002`).

| `.env` setting | Default | What it sets |
| --- | --- | --- |
| `OPENCRQ_EXTERNAL_SCHEME`, `OPENCRQ_HOST`, `OPENCRQ_PORT` | `http`, `localhost`, `8082` | The public URL of OpenCRQ. It is also its identity towards XTM One: the tokens OpenCRQ signs name it, and the tokens XTM One sends must be addressed to it |
| `OPENCRQ_S3_BUCKET` | `opencrq` | The bucket of OpenCRQ in the shared S3 storage |

OpenCRQ registers with XTM One with `PLATFORM_REGISTRATION_TOKEN`, as OpenCTI and OpenAEV do. Its OpenCTI and OpenAEV connectors are set for the **Default** tenant with the admin tokens of both platforms (`OPENCTI_ADMIN_TOKEN`, `OPENAEV_ADMIN_TOKEN`). OpenCRQ also reaches the internet for the vulnerability sources it reads by default (NIST, FIRST and CISA).

OpenCRQ adds two processes to the host: the HTTP server, with a heap of 512 MB, and the worker that runs the background jobs, with a heap of 4,096 MB. Plan about 5 GB for OpenCRQ on top of the stack, and about 50 more PostgreSQL connections at peak.

## Before you start

- The stack installed as [Install with Docker Compose](/deploy/install/docker-compose/) describes, with `PLATFORM_REGISTRATION_TOKEN` set to a value of your own: see [Connect the modules to each other](/deploy/configure/connect-modules/#register-the-platforms-with-xtm-one). Run the commands of this page from its folder.
- The host name users will type, such as `opencrq.example.com`, served over HTTPS by your reverse proxy: see [Publish the platforms over HTTPS](/deploy/install/reverse-proxy/). The session cookie of OpenCRQ is `Secure`: over plain HTTP, browsers keep it on `localhost` only.

## Start OpenCRQ

:::steps
1. Set the public URL of OpenCRQ in `.env`.

   ```bash
   OPENCRQ_EXTERNAL_SCHEME=https
   OPENCRQ_HOST=opencrq.example.com
   OPENCRQ_PORT=443
   ```

   `OPENCRQ_PORT` is the port of the public URL, 443 behind the reverse proxy: the host still publishes OpenCRQ on port 8082. The stack runs `filigran/opencrq:latest`, the latest release of OpenCRQ, like the other modules.

2. Download the images and start the stack. The first start of OpenCRQ creates its tables and loads the reference catalogues: allow a minute or two.

   ```bash
   docker compose pull
   docker compose up -d
   ```

   :::verify
   ```bash
   docker compose ps opencrq-db-init opencrq-bucket-init opencrq --all --format '{{.Service}} {{.State}} {{.Health}}'
   docker compose exec opencrq wget -qO- http://localhost:8444/health
   ```
   Both jobs show `exited`, and `opencrq` shows `running healthy`. Then `{"healthy":true,...}`, with the state of each process: each line `childrenHealthStates` lists says `RUNNING`.
   :::

3. Check that OpenCRQ registered with XTM One.

   ```bash
   docker compose logs opencrq | grep "Registered with XTM One"
   ```

   The line names the version of XTM One, and `eeEnabled` says whether the XTM license covers OpenCRQ. OpenCRQ registers when it starts, then every 5 minutes. A `XTM One registration failed` line with HTTP 401 or 403 means that the registration token was changed in `.env` without recreating `opencrq` and the XTM One containers: run `docker compose up -d` again.

4. Put the reverse proxy in front of port 8082 of the host, for `opencrq.example.com`, once you have created the first administrator below.
:::

## Create the first administrator

The first person to open OpenCRQ creates its owner account. Until that account exists, anyone who reaches port 8082 of the host can create it: create it right after the first start, before the reverse proxy publishes OpenCRQ, through an SSH tunnel to port 8082 if you work from another machine.

:::steps
1. Open OpenCRQ in a browser. It asks you to **Please create the owner account to secure the system and complete the setup.**
2. Enter the **Email** of the administrator, a **Password** of 8 characters or more, and **Repeat Password**. Use the admin email of XTM One, `XTM_ONE_ADMIN_EMAIL` in `.env`: XTM One finds a person in each product by their email, and the assistant and the MCP server work for that account only if both match.
3. Click **Create Admin Account & Launch**.
:::

You are signed in, with every platform capability, in a tenant named **Default**. Its identifier, `01a043da-ff47-72c2-aaa4-7da56d85c83e`, is the same on every installation: it is the tenant the OpenCTI and OpenAEV connectors of xtm-docker are set for. Each connector still waits for a service account in **Integrations**: see [Let integrations sign in with service accounts](/opencrq/administer/service-accounts/). For OpenCTI, pick a live stream whose filter lets through the threats and their relationships (`targets`, `uses`, `attributed-to`): a stream filtered on threats alone delivers bare threats. Other administrators sign in with an invitation: see [Give people access with users, groups and roles](/opencrq/administer/users-and-roles/).

:::example[Start OpenCRQ in Example CERT's stack]
Example CERT runs the XTM stack for its member hotel chains, with OpenCTI at `https://opencti.example.com` and XTM One at `https://xtm-one.example.com`. Its platform engineer starts OpenCRQ at `https://opencrq.example.com` for Grand Meridian Hotels and the other members:

```bash
cat >> .env <<EOF
OPENCRQ_EXTERNAL_SCHEME=https
OPENCRQ_HOST=opencrq.example.com
OPENCRQ_PORT=443
EOF
cp .env "$SECRETS_DIR/xtm-stack.env"
docker compose pull && docker compose up -d
```

`SECRETS_DIR` is the folder the team's secret store syncs. The log of `opencrq` shows `Registered with XTM One`. The CERT's administrator, `admin@example.com` in OpenCTI, OpenAEV and XTM One alike, then creates the owner account of OpenCRQ with the same email, through an SSH tunnel to port 8082, before the reverse proxy publishes `opencrq.example.com`.
:::

## Where the data is

| Data | Where |
| --- | --- |
| Risk scopes, assets, scenarios, prices, settings | The `opencrq` database of `pgsql`, in the `pgsqlshareddata` volume |
| STIX bundles uploaded on the **Threats** page | The `opencrq` bucket of `minio`, in the `s3data` volume |

Back them up with the rest of the stack: see [Back up and restore](/deploy/operate/backup/). OpenCRQ keeps nothing else: the container can be recreated at any time.

## What's next

- [Configure OpenCRQ](/deploy/configure/opencrq/), to connect other tenants to OpenCTI, OpenAEV and the other sources.
- [How OpenCRQ quantifies risk](/opencrq/concepts/how-risk-is-quantified/), to hand over to the risk managers.
- [OpenCRQ configuration](/reference/opencrq/configuration/), every setting with its variable.
