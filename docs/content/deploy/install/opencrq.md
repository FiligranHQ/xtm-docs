---
title: Install OpenCRQ
description: Start OpenCRQ in the XTM stack deployed with Docker Compose, which sets up its database, bucket and registration with XTM One, then sign in through XTM One and create its first tenant.
doc_type: how-to
products:
  - opencrq
personas: [platform-engineer]
owner: docs-opencrq
last_reviewed: 2026-11-20
---

This page starts OpenCRQ, the cyber risk quantification module of the XTM platform, in an XTM stack installed with [FiligranHQ/xtm-docker](https://github.com/FiligranHQ/xtm-docker). xtm-docker ships OpenCRQ with the rest of the stack: its database in the shared PostgreSQL, its bucket in the shared S3 storage, its registration with XTM One and its connectors to OpenCTI and OpenAEV are set up for you. You give it its public URL. At the end, OpenCRQ answers at that URL, you have signed in through XTM One as its super administrator, and you have created its first tenant.

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

OpenCRQ registers with XTM One with `PLATFORM_REGISTRATION_TOKEN`, as OpenCTI and OpenAEV do. Its users sign in through XTM One: OpenCRQ keeps no password, and has no sign-in form of its own. OpenCRQ also reaches the internet for the vulnerability sources it reads by default (NIST, FIRST and CISA).

The connections to OpenCTI and OpenAEV are set per tenant, by the tenant's identifier. OpenCRQ creates no tenant on its own: you create the first one once you have signed in, and it gets a new identifier. Connect it to OpenCTI and OpenAEV afterwards, as [Connect the first tenant](#connect-the-first-tenant) shows.

OpenCRQ adds two processes to the host: the HTTP server, with a heap of 512 MB, and the worker that runs the background jobs, with a heap of 4,096 MB. Plan about 5 GB for OpenCRQ on top of the stack, and about 50 more PostgreSQL connections at peak.

## Before you start

- The stack installed as [Install with Docker Compose](/deploy/install/docker-compose/) describes, with `PLATFORM_REGISTRATION_TOKEN` set to a value of your own: see [Connect the modules to each other](/deploy/configure/connect-modules/#register-the-platforms-with-xtm-one). Run the commands of this page from its folder.
- The host name users will type, such as `opencrq.example.com`, served over HTTPS by your reverse proxy: see [Publish the platforms over HTTPS](/deploy/install/reverse-proxy/). XTM One sends users back to OpenCRQ at that URL after they sign in, and accepts it only over HTTPS, or on `localhost` for a trial. The session cookie of OpenCRQ is `Secure`: over plain HTTP, browsers keep it on `localhost` only.
- The credentials of XTM One's administrator, the account of `XTM_ONE_ADMIN_EMAIL` in `.env`: it is the only account that can open OpenCRQ before any tenant exists.

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

4. Put the reverse proxy in front of port 8082 of the host, for `opencrq.example.com`. Sign-in goes through this URL: XTM One sends the browser back to `https://opencrq.example.com/api/auth/xtm/callback`, and a sign-in started at another address first moves the browser there.

   :::verify
   ```bash
   docker compose logs opencrq | grep "Sign-in with XTM One stays off"
   ```
   The command prints nothing. A line means the public URL of OpenCRQ is neither HTTPS nor `localhost`: XTM One would refuse it, so nobody can sign in. Fix `OPENCRQ_EXTERNAL_SCHEME` and run `docker compose up -d` again.
   :::
:::

## Sign in and create the first tenant

OpenCRQ has no first-administrator form. XTM One's administrator is OpenCRQ's super administrator: every right on the platform and in every tenant, given again at each sign-in, without being a member of any group. They create the first tenant.

:::steps
1. Open `https://opencrq.example.com` in a browser and click **Sign in with XTM One**.
2. Sign in to XTM One with the account of `XTM_ONE_ADMIN_EMAIL`. If XTM One already has a session in this browser, it sends you back at once.
3. OpenCRQ opens on the **Tenants** page of the platform, `/platform/security/tenants`. Click **Create**.
4. In the **Create a new tenant** panel, enter the **Name** of the organization OpenCRQ prices risk for, and click **Submit**.
5. Open the new tenant from its card.
:::

::screenshot{id=opencrq-install-first-tenant alt="The Tenants page of the platform, opened by XTM One's administrator right after the first sign-in, shows the first tenant's card with its number of members."}

The tenant starts with an **Admin** group and a **Connectors** group, and with no member: you reach it as super administrator, tagged **Super Administrator** in its **Users** list, without belonging to it. Add its people from :menu[Settings > Security > Users] with **Add members**: see [Give people access with users, groups and roles](/opencrq/administer/users-and-roles/). Anyone else who signs in before then reads **You don't have access to OpenCRQ yet**. A Community Edition platform serves this one tenant; more require the Enterprise Edition: see [Run several tenants in one platform](/opencrq/administer/tenants/).

When the sign-in does not work:

| OpenCRQ shows | What it means |
| --- | --- |
| **XTM One is unavailable**, with **Try again** | XTM One does not answer OpenCRQ, or has given it no sign-in client yet. There is no other way in. Check that XTM One is healthy, that `PLATFORM_REGISTRATION_TOKEN` is not the published default (XTM One then creates no sign-in client), and the reverse proxy step of [Start OpenCRQ](#start-opencrq). Then click **Try again** |
| **You are not signed in**, with the reason | XTM One refused or interrupted the sign-in, or it was cancelled. Click **Sign in with XTM One** again |
| **You don't have access to OpenCRQ yet** | You signed in with another XTM One account than its administrator. Click **Sign out**, sign out of XTM One too, and sign in again with the account of `XTM_ONE_ADMIN_EMAIL` |

## Connect the first tenant

The connectors to OpenCTI and OpenAEV take the identifier of the tenant in their variables. Read it in the tenant's :menu[Settings > Parameters], under **Identifier**, and set the URL and the admin token of each platform for it in the `environment` of the `opencrq` service, then run `docker compose up -d`. [Configure OpenCRQ](/deploy/configure/opencrq/#connectors-of-each-tenant) lists the variables of every connector.

Both connectors then come up enabled: at that start, OpenCRQ creates their service accounts, `OpenCTI (deployment)` and `OpenAEV (deployment)`, and creates in OpenCTI the Live Stream `OpenCRQ threat feed`, which carries the threats and their relationships (`targets`, `uses`, `attributed-to`). The OpenCTI token needs **Manage data sharing** or **Bypass all capabilities** for that: see [Link OpenCRQ to OpenCTI and OpenAEV](/deploy/configure/connect-modules/#link-opencrq-to-opencti-and-openaev).

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

`SECRETS_DIR` is the folder the team's secret store syncs. The log of `opencrq` shows `Registered with XTM One`, and no `Sign-in with XTM One stays off` line. Once the reverse proxy publishes `opencrq.example.com`, the CERT's administrator, `admin@example.com` in XTM One (`XTM_ONE_ADMIN_EMAIL`), clicks **Sign in with XTM One** and creates the tenant **Grand Meridian Hotels**. Its :menu[Settings > Parameters] shows the **Identifier** `019a8d21-4b7e-7f30-8c15-3e9a6b2d4f71`. The platform engineer connects it to the stack's OpenCTI and OpenAEV, in `docker-compose.override.yml`, under the `opencrq` service:

```yaml
    environment:
      - CONNECTORS__OPENCTI__019a8d21-4b7e-7f30-8c15-3e9a6b2d4f71__URL=http://opencti:8080
      - CONNECTORS__OPENCTI__019a8d21-4b7e-7f30-8c15-3e9a6b2d4f71__TOKEN=${OPENCTI_ADMIN_TOKEN}
      - CONNECTORS__OPENAEV__019a8d21-4b7e-7f30-8c15-3e9a6b2d4f71__URL=http://openaev:8080
      - CONNECTORS__OPENAEV__019a8d21-4b7e-7f30-8c15-3e9a6b2d4f71__TOKEN=${OPENAEV_ADMIN_TOKEN}
```

After `docker compose up -d`, both connectors come up enabled in the tenant's **Integrations**, and the OpenCTI one reads the Live Stream `OpenCRQ threat feed` that OpenCRQ created with the admin token. The administrator then adds Grand Meridian's risk manager with **Add members**.
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
