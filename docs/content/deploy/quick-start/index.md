---
title: Run the XTM platform locally
description: Start the whole XTM platform, XTM One, OpenCTI, OpenAEV, OpenCRQ and XTM Composer,
  on one machine with Docker Compose.
doc_type: tutorial
products:
  - suite
personas: [platform-engineer, evaluator]
owner: docs-platform
last_reviewed: 2026-10-31
---

This tutorial starts the XTM platform on one machine with the official Docker deployment, [FiligranHQ/xtm-docker](https://github.com/FiligranHQ/xtm-docker), in about fifteen minutes. Two commands configure and start the whole stack: XTM One, at the center of the platform, with OpenCTI, OpenAEV, OpenCRQ, XTM Composer, their default integrations and the shared services. No module is optional. It is meant to try the platform or to develop against it, not for production.

::diagram{id=deploy-quick-start}

:::tip[Let an AI agent do it]
Every step below has a command and a check, so a coding agent such as Claude Code can follow it on its own. See [Deploy with an AI agent](/deploy/quick-start/with-an-ai-agent/).
:::

## What you need

- Docker Engine 20.10 or later with Docker Compose 2.20.2 or later (the stack is split into included files): `docker compose version` answers.
- On a Mac with an Apple chip: Docker Desktop with **Use Rosetta for x86_64/amd64 emulation** on. XTM One is published for Intel processors only and runs through emulation.
- 16 GB of RAM for the whole platform, 32 GB recommended, and 50 GB of free disk.
- Git, to get the deployment. Nothing else: the configuration is generated in a container, and the same commands work on Linux, macOS and Windows.
- Free ports on the machine: 8080 (OpenCTI), 8081 (OpenAEV), 8082 (OpenCRQ), 8090 (XTM One), and 9200, 9000, 5672, 15672 and 6379 for the shared services.

## What the stack runs

Every module starts, with the shared services they use, and each platform answers on its own address.

::stack-builder

## Start the platform

:::steps
1. Get the deployment.

   ```bash
   git clone https://github.com/FiligranHQ/xtm-docker.git
   cd xtm-docker
   ```

2. Create the configuration.

   ```bash
   docker compose -f init.yml run --rm init
   ```

   A temporary container creates `.env` from `.env.sample`: a random value for every password, key and token, the users of the shared services, and mail sent to the local machine. It prints the address and the admin password of each platform. Every secret stays in `.env`, which only you can read; the command never overwrites an existing `.env`.

   :::verify
   ```bash
   docker compose config --quiet
   ```
   It prints nothing: the stack's configuration is complete.
   :::

   On a machine with 16 GB of RAM, give Elasticsearch 2 GB instead of 4: set `ELASTIC_MEMORY_SIZE=2G` in `.env`. If another program uses one of the ports above, change `OPENCTI_PORT` or `XTM_ONE_PORT` in `.env`. The ports of OpenAEV, OpenCRQ and the shared services move in `docker-compose.override.yml`, with `OPENAEV_PORT` and `OPENCRQ_PORT` set to match, as [Network and ports](/deploy/plan/network/#ports-published-on-the-host) shows.

3. Start the platform.

   ```bash
   docker compose up -d --pull always
   ```

   `--pull always` downloads the current images first, even if you have run XTM before: Docker Compose would otherwise start an older image kept under the same `latest` tag. The first start creates the databases: allow a few minutes.

   :::verify
   ```bash
   docker compose ps opencti openaev xtm-one opencrq
   ```
   The four platforms say `(healthy)` in their status. Run it again until they do.
   :::

4. Sign in at the addresses the second step printed, with the admin email and password of each platform. They are in `.env` too: `OPENCTI_ADMIN_PASSWORD`, `OPENAEV_ADMIN_PASSWORD` and `XTM_ONE_ADMIN_PASSWORD`. OpenCRQ has no password: **Sign in with XTM One**, as the admin of XTM One. You land in its first tenant, `Default`, which OpenCRQ creates at its first start and xtm-docker already connects to OpenCTI and OpenAEV.
:::

## Stop and start over

| To | Run | Your data |
| --- | --- | --- |
| Stop the platform | `docker compose stop` | Kept |
| Remove the containers | `docker compose down` | Kept in the volumes |
| Start from scratch | `docker compose down -v` | Deleted, with every volume |

## What's next

- [Deploy with an AI agent](/deploy/quick-start/with-an-ai-agent/), to repeat this without typing.
- [The XTM platform](/start/), what each module does and how they work together.
