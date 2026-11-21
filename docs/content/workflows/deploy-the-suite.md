---
title: Deploy the XTM platform
description: Run OpenCTI, OpenAEV, OpenCRQ and XTM One side by side behind your reverse proxy, registered with each other around XTM One, sharing one license, and watched.
doc_type: journey
journey: deploy-the-suite
products:
  - opencti
  - openaev
  - opencrq
  - xtm-one
personas: [platform-engineer, admin]
owner: docs-suite
last_reviewed: 2026-11-09
sidebar_position: 63
---

At the end of this journey, OpenCTI, OpenAEV, OpenCRQ and XTM One run on one host with the official Docker deployment, [FiligranHQ/xtm-docker](https://github.com/FiligranHQ/xtm-docker), behind your reverse proxy over HTTPS. Each platform has registered with XTM One with the same secret, XTM One works with their data as the person who asks, one Filigran license gives the platform its Enterprise Edition, and you know which endpoints and logs tell you it is healthy. A platform engineer drives it; an administrator takes over each platform once it answers. The steps below are the pages to follow, in order.

::diagram{id=journey-deploy-the-suite}

## Who takes part

- **Platform engineers** plan the stack, prepare the host, install the modules, connect them, set the configuration of XTM One, monitor it and upgrade the stack (steps 1 to 9).
- **Administrators** sign in first with the admin accounts of `.env`, add the language model and the license in XTM One (step 4), create the owner account of OpenCRQ (step 5), then hand access to the teams.

## Why it spans several products

The products are built to run as one stack, with XTM One at its center:

- **Shared services.** OpenCTI, OpenAEV, XTM One and OpenCRQ keep their data in one set of backing services: Elasticsearch, RabbitMQ, Redis, S3 storage (RustFS) and PostgreSQL with pgvector. Each module keeps to its own index prefix, bucket and database, so one instance of each service is enough.
- **Registration.** OpenCTI, OpenAEV and OpenCRQ register with XTM One with one shared secret, `PLATFORM_REGISTRATION_TOKEN`, then register again every 5 minutes as a heartbeat. Once registered, their users get **Ask Ariane**, and XTM One agents work with their data.
- **Calls as a person.** XTM One calls each platform as the person who asks, and the platform applies that person's rights. The tokens of these calls name the public URL of their issuer, so the public URLs you set are also the identities of the products.
- **People matched by email.** XTM One finds a person in each platform by email: the administrator's email must be the same in the four products.
- **One license.** A Filigran license installed in XTM One travels back to each platform in its registration answer, and decides the edition of OpenCRQ. A platform with an Enterprise Edition license of its own also gives XTM One its Enterprise Edition.

Installed apart, each product needs its own services and its own license, OpenCRQ does not start without XTM One, and no assistant reads across the products. What each part is and which service it uses is in [The XTM stack: modules and shared services](/deploy/plan/architecture/); how a request carries a person's identity is in [XTM One in OpenCTI](/xtm-one/use/in-opencti/).

## What you need

- A Linux host, `linux/amd64`, with Docker Engine 20.10 or later and Docker Compose 2.24.4 or later, 16 GB of RAM and 50 GB of free disk at least, 32 GB of RAM for production, and about 5 GB more for OpenCRQ. See [Requirements](/deploy/plan/requirements/) and [Sizing](/deploy/plan/sizing/).
- A host name per platform, and a reverse proxy that serves them over HTTPS: see [Publish the platforms over HTTPS](/deploy/install/reverse-proxy/).
- A Filigran license certificate for the Enterprise Edition, and the API key of a language model provider or the address of a model on your network.
- A mail relay for OpenCTI, OpenAEV and XTM One.
- On the host, a shell account that runs `sudo` and Docker. In the products, the admin accounts that `.env` creates are the first administrators: no role is needed before them.

## How it unfolds

The story: Example CERT runs the XTM platform for the hotel chains it serves. Its platform engineer installs the stack on one host, behind Nginx on the same host, with OpenCTI at `https://opencti.example.com`, OpenAEV at `https://openaev.example.com`, XTM One at `https://xtm-one.example.com` and OpenCRQ at `https://opencrq.example.com`. The CERT's administrator, `admin@example.com`, takes over each platform.

### Plan the stack

xtm-docker starts the whole stack, and no module is optional ([step 1](/deploy/plan/architecture/)). XTM One runs as `xtm-one`, the interface and its API, and `xtm-one-worker`, the background work. OpenCTI and OpenAEV run with their default integrations, and XTM Composer manages the integrations of both. Elasticsearch, RabbitMQ, Redis, S3 storage and PostgreSQL serve them all. OpenCRQ is not in xtm-docker yet: it joins the same PostgreSQL and S3 storage later, through the override file.

### Prepare the host

The engineer checks the versions of Docker and Docker Compose, the processor and the memory, and raises `vm.max_map_count` for Elasticsearch ([step 2](/deploy/plan/requirements/)). xtm-docker already sets what the shared services need, such as `max_connections=200` on PostgreSQL for OpenAEV and XTM One together.

### Install the stack on one host

In a clone of xtm-docker, `.env` gets fresh secrets for every password, key and token, and a copy goes to the team's secret store ([step 3](/deploy/install/docker-compose/)). `OPENCTI_ADMIN_EMAIL`, `OPENAEV_ADMIN_EMAIL` and `XTM_ONE_ADMIN_EMAIL` all read `admin@example.com`. Each platform's public URL is built from its `*_EXTERNAL_SCHEME`, `*_HOST` and `*_PORT`, here `https`, its host name and `443`. `docker-compose.override.yml` publishes OpenCTI on `127.0.0.1:8080`, OpenAEV on `127.0.0.1:8081` and XTM One on `127.0.0.1:8090` for the proxy, keeps Elasticsearch and Redis on the host itself, and pins the versions: OpenCTI `7.261030.1`, OpenAEV `3.261029.0`, XTM One `1.261110.2`.

### Make XTM One usable

Behind the proxy, XTM One must know which address the proxy reaches it from: `FORWARDED_ALLOW_IPS=172.18.0.1`, the gateway of the stack's network, under `xtm-one` in the override file ([step 4](/deploy/install/xtm-one/)). The proxy keeps requests open for 300 seconds, since an agent's answer can take minutes. `/api/health/ready` answers `"status":"ok"` once the start is complete. The administrator signs in, adds the language model in **Administration** > **Settings** > **AI and models** > **Models**, or the engineer sets it as `LLM_PROVIDER_ANTHROPIC_API_KEY`, and the license goes in **Administration** > **Settings** > **Platform**, with **Install License** on the **Enterprise License** card.

### Add OpenCRQ

The override file gets three services: `opencrq-db-init`, which creates the `opencrq` role and database in the shared PostgreSQL, `opencrq-bucket`, which creates its bucket, and `opencrq`, published on `127.0.0.1:8082` ([step 5](/deploy/install/opencrq/)). `XTM_ONE__TOKEN` takes `PLATFORM_REGISTRATION_TOKEN`, and `OPENCRQ_API_URL=http://opencrq:3002` on both XTM One containers lets XTM One call it inside the stack. The log reads `Registered with XTM One`. The administrator creates the owner account of OpenCRQ with `admin@example.com` before the proxy publishes it.

### Connect the platforms around XTM One

`PLATFORM_REGISTRATION_TOKEN` holds one random value, read by OpenCTI, OpenAEV, OpenCRQ and both XTM One containers ([step 6](/deploy/configure/connect-modules/)). After `docker compose up -d`, the platforms register within minutes, and **My Profile** > **MCP Endpoint** in XTM One shows each platform as `Connected`. OpenAEV also registers in OpenCTI as the connector `OpenAEV Coverage - <tenant>`, which answers security coverage requests. The clocks stay synchronized with NTP: XTM One refuses a token from a clock more than 60 seconds ahead.

### Finish the configuration of XTM One

Neither xtm-docker nor the product's Compose file passes XTM One a mail server: until `SMTP_HOSTNAME` is set on both containers, XTM One sends no invitation and writes the link of a password reset in its log ([step 7](/deploy/configure/xtm-one/)). The same page sets the security headers of its pages and the size of its processes. `XTM_ONE_SECRET_KEY` never changes once XTM One has started: it encrypts the credentials XTM One stores.

### Watch it

The engineer monitors `/api/health/ready` as well as `/api/health`: the healthcheck of xtm-docker asks `/api/health`, which answers as soon as the server listens ([step 8](/deploy/operate/xtm-one/)). Prometheus metrics, one JSON line per request in the logs, and the **Security audit** section of **Administration** > **Settings** > **Access and security** complete the picture: the audit flags a published registration token or a proxy XTM One does not trust.

### Upgrade

Each upgrade starts with a snapshot of Elasticsearch and a copy of the other volumes, then `git pull`, the new pinned versions, `docker compose pull` and `docker compose up -d` ([step 9](/deploy/upgrade/)). OpenCTI and OpenAEV migrate their data as they start. The [breaking changes](/deploy/upgrade/breaking-changes/) say what to do before and after, and the [compatibility matrix](/deploy/upgrade/compatibility/) which versions work together.

## The steps

Each step opens its page; the first one starts the journey, and every page of it then shows where you are.

::journey-steps

## Where to go next

- [Sign in the first time and keep an emergency account](/deploy/configure/first-access/), before you turn on single sign-on.
- [Configure OpenCRQ](/deploy/configure/opencrq/), to connect each tenant to OpenCTI and OpenAEV.
- [Back up and restore](/deploy/operate/backup/), to keep the volumes and `.env` together.
- [Deploy integrations across the platform with XTM Composer](/workflows/composer-rollout/), to run the integrations of OpenCTI and OpenAEV from their interface.
