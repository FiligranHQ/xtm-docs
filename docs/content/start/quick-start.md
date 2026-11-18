---
title: Try OpenCTI with Docker
description: Run OpenCTI on your machine with the official Docker deployment, sign in, and browse MITRE ATT&CK and reference data in about fifteen minutes.
doc_type: tutorial
products:
  - opencti
  - xtm-one
personas: [evaluator, platform-engineer]
owner: docs-platform
last_reviewed: 2026-11-01
sidebar_position: 1
---

In about fifteen minutes, most of it spent downloading images, you run OpenCTI on your own machine and sign in as its administrator. You then browse real data: the MITRE ATT&CK techniques and threat groups, and the countries, regions and sectors that OpenCTI uses as reference. You use the official Docker deployment of the XTM platform, [FiligranHQ/xtm-docker](https://github.com/FiligranHQ/xtm-docker), which starts the whole stack; this tutorial looks at OpenCTI and its default connectors. This setup is meant for trying the platform, not for production: to deploy for real, start from [Deploy the platforms](/deploy/).

XTM One, the AI layer at the center of the platform, OpenAEV, OpenCRQ and XTM Composer start with OpenCTI: every module of the stack runs. To try each of them, follow [Run the XTM platform locally](/deploy/quick-start/) instead.

::diagram{id=start-quick-start}

:::note
The deployment starts the latest release of each product, whatever version of this documentation you read. Long-term support (LTS) versions need a license and their own image tags: see [Release lifecycle](/deploy/upgrade/lifecycle/).
:::

## What you need

- Docker Engine 20.10 or later with Docker Compose 2.20.2 or later (the stack is split into included files): `docker compose version` answers.
- On a Mac with an Apple chip, Docker Desktop with **Use Rosetta for x86_64/amd64 emulation** on. XTM One is published for Intel processors only and runs through emulation.
- 16 GB of RAM and 50 GB of free disk.
- Git. Nothing else: the configuration is generated in a container, and the same commands work on Linux, macOS and Windows.
- These ports free on the machine: 8080 (OpenCTI), 8081 (OpenAEV), 8082 (OpenCRQ), 8090 (XTM One), and 9200, 9000, 5672, 15672 and 6379 for the shared services.

## Step 1: Get the deployment

```bash
git clone https://github.com/FiligranHQ/xtm-docker.git
cd xtm-docker
```

The folder holds `docker-compose.yml`, which describes every service of the platform, and `.env.sample`, the settings to copy.

## Step 2: Create the configuration

```bash
docker compose -f init.yml run --rm init
```

A temporary container copies `.env.sample` to `.env` and gives every password, key and token a random value: OpenCTI does not start with the sample values. Mail goes to the local machine, so no mail server is needed. The command prints the address and the administrator password of each platform, and never overwrites an existing `.env`.

Keep `.env` to yourself: it holds the administrator password, and anyone who can read it can take over the platform. The command makes it readable by you alone.

## Step 3: Fit the stack to your machine

Give Elasticsearch 2 GB of memory instead of 4, which suits a 16 GB machine: in `.env`, set `ELASTIC_MEMORY_SIZE=2G`.

The deployment starts OpenCTI with its worker and ten connectors: file imports and exports, document analysis, and the two that bring data in, **OpenCTI Datasets** and the MITRE ATT&CK connector. XTM One, OpenAEV with its collectors and injectors, XTM Composer and the shared services start with it.

:::verify
```bash
docker compose config --services
```
The list holds `opencti`, `worker`, `connector-mitre`, `connector-opencti` and eight other `connector-` services, `xtm-one` with its worker, `openaev`, `opencrq` with its two init jobs, `xtm-composer`, and the shared services: 32 services in all.
:::

## Step 4: Start the stack

```bash
docker compose up -d --pull always
```

`--pull always` downloads the current images first, even if you have used xtm-docker before: Docker Compose would otherwise start an older image kept under the same `latest` tag. The first start creates the databases and indices: allow a few minutes.

:::verify
```bash
docker compose ps opencti openaev xtm-one opencrq
```
The four platforms say `(healthy)` in their status. Run it again until they do.
:::

If a container restarts in a loop, read its logs with `docker compose logs opencti`. Elasticsearch on Linux may need a larger memory map: `sudo sysctl -w vm.max_map_count=262144`.

## Step 5: Sign in

Open `http://localhost:8080` in your browser and sign in with the email and password that step 2 printed for OpenCTI. They are in `.env` too, as `OPENCTI_ADMIN_EMAIL` and `OPENCTI_ADMIN_PASSWORD`. The home dashboard opens.

The administrator is created from `.env` and reset to its values at every start: a password you change in the interface lasts until the next restart. See [Sign in the first time and keep an emergency account](/deploy/configure/first-access/).

## Step 6: Watch the data arrive

As soon as OpenCTI is up, the two data connectors send their bundles, and the three workers write them into the knowledge base. MITRE ATT&CK is the larger one: allow about ten minutes for it to land.

1. Open :menu[Integrations] and stay on the **Deployed** tab. **OpenCTI Datasets** and the MITRE ATT&CK connector show their queued messages. The number goes down as the workers process them.

   ::screenshot{id=opencti-ingest-connectors-deployed alt="The Deployed tab of Integrations: the platform figures at the top, with queued bundles and connected workers, then each connector with its queued messages, its last activity and its status."}

2. Open :menu[Techniques > Attack patterns]. The ATT&CK techniques fill the list, such as **Phishing** and its sub-techniques.
3. Open :menu[Threats > Intrusion sets]. The groups that ATT&CK tracks are there, each with the techniques it uses on its **Knowledge** tab, under **Attack patterns**.
4. Open :menu[Locations > Countries] and :menu[Entities > Sectors]. The reference data that reports and threats point to is in place.

:::example[Check ATT&CK coverage before a hotel-sector project]
The architect of Example CERT, which serves hotel chains, wants to see how OpenCTI models an attack technique before proposing the platform. On a 16 GB laptop, `.env` reads:

```bash
ELASTIC_MEMORY_SIZE=2G
```

Ten minutes after the start, the architect types `Phishing` in **Search the platform...** at the top and opens the attack pattern **Phishing** (T1566). Its **Overview** gives its sub-techniques and the courses of action that mitigate it, and its **Knowledge** tab lists the intrusion sets that use it, all imported from ATT&CK.
:::

## Stop and start over

| To | Run | Your data |
| --- | --- | --- |
| Stop the stack | `docker compose stop` | Kept |
| Remove the containers | `docker compose down` | Kept in the volumes |
| Start from scratch | `docker compose down -v` | Deleted, with every volume |

## What you learned

- OpenCTI is a platform, workers that write data, and connectors that bring it in: [How OpenCTI works](/opencti/concepts/architecture/).
- What you browsed is a knowledge graph of STIX objects and relationships: [Data model](/opencti/concepts/data-model/).
- Connectors are how data reaches OpenCTI, and the **Deployed** tab is where you follow them: [Import with connectors](/opencti/use/ingest/connectors/).

To go further, explore the knowledge you imported in [Explore knowledge](/opencti/use/explore/), try OpenAEV and XTM One with [Run the XTM platform locally](/deploy/quick-start/), or plan a real deployment in [Deploy the platforms](/deploy/).
