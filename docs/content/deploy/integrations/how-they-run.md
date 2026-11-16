---
title: How integrations run
description: How connectors, collectors, injectors and executors run next to OpenCTI and OpenAEV, how they reach their platform, and which ones xtm-docker always starts.
doc_type: explanation
products:
  - opencti
  - openaev
  - composer
personas: [platform-engineer, admin]
owner: docs-platform
last_reviewed: 2026-10-31
migrated_from:
  - opencti:/deployment/connectors/
---

An integration is a small program that connects OpenCTI or OpenAEV to another system: it imports or exports data, enriches it, or runs actions. Most integrations run as their own container next to the platform. This page explains how they run, how they reach their platform, and which ones the XTM stack starts by default, so you can choose how to run yours.

## Four kinds of integration

| Kind | Platform | What it does | How it is built |
| --- | --- | --- | --- |
| Connector | OpenCTI | Imports, enriches, streams or exports threat intelligence | [Connectors](/develop/integration-model/connectors/) |
| Collector | OpenAEV | Reads detection and prevention results, or reference data, from a security tool or a source | [Collectors](/develop/integration-model/collectors/) |
| Injector | OpenAEV | Runs injects: sends emails, runs scans, calls other tools | [Injectors](/develop/integration-model/injectors/) |
| Executor | OpenAEV | Deploys and runs OpenAEV agents on endpoints | [Executors](/develop/integration-model/executors/) |

The catalog of available integrations is the [XTM Hub integrations library](https://hub.filigran.io/cybersecurity-solutions/open-cti-integrations). Each platform also shows its own catalog: :menu[Integrations], **Available** tab (:menu[Data > Ingestion > Connector catalog] before 7.260811) in OpenCTI, **Integrations** > **Catalog** in OpenAEV.

### Built-in and external

Some integrations live inside the platform and need no deployment:

- OpenAEV embeds some injectors, such as email, SMS and media pressure. You configure them in the platform configuration.
- OpenAEV runs its own agent natively, and you configure the executors of endpoint security products, such as Tanium or CrowdStrike Falcon, in **Integrations** > **Executors** of OpenAEV.

Everything else is external: a program, usually in Python, published as a Docker image (`opencti/connector-*`, `openaev/collector-*`, `openaev/injector-*`), that you run yourself or let XTM Composer run for you.

## How an integration reaches its platform

Every external integration needs two settings to reach its platform, plus an identity of its own:

| Setting | OpenCTI connector | OpenAEV collector or injector |
| --- | --- | --- |
| Platform URL | `OPENCTI_URL` | `OPENAEV_URL` |
| API token | `OPENCTI_TOKEN` | `OPENAEV_TOKEN` |
| Unique ID, a UUIDv4 | `CONNECTOR_ID` | `COLLECTOR_ID` or `INJECTOR_ID` |
| Display name | `CONNECTOR_NAME` | `COLLECTOR_NAME` or `INJECTOR_NAME` |

At start, the integration registers with the platform through its API, with its token. The platform then gives it the address and credentials of RabbitMQ:

- An OpenCTI connector pushes the STIX 2.1 bundles it produces to its RabbitMQ queue, and the OpenCTI workers write them into the platform. Enrichment and export connectors also receive their jobs from RabbitMQ.
- An OpenAEV injector receives its work through RabbitMQ in the same way.

::diagram{id=deploy-integrations-how-they-run}

So an integration must reach two things on the network: the platform URL you give it, and RabbitMQ at the host name and port the platform hands out. In the XTM stack, both are on the Docker network of the deployment (`xtm_default` with the default `COMPOSE_PROJECT_NAME=xtm`), as `http://opencti:8080`, `http://openaev:8080` and `rabbitmq:5672`. A connector that cannot reach RabbitMQ can talk to OpenCTI over HTTP only: see [Run an integration without XTM Composer](/deploy/integrations/manual/).

The token decides what the integration can do. OpenCTI recommends one user per connector, in the `Connectors` group. OpenAEV integrations can share an administrator token. The details are in [Run an integration without XTM Composer](/deploy/integrations/manual/#create-a-token).

## Three ways to run integrations

| Way | What you do | When to choose it |
| --- | --- | --- |
| [XTM Composer](/deploy/integrations/composer/) | Deploy, start, stop and configure integrations from the platform interface. XTM Composer runs the containers | Most deployments. It requires an Enterprise Edition license |
| The default integrations of xtm-docker | Nothing: they are part of the stack | The integrations every platform needs, always started with the stack |
| [By hand](/deploy/integrations/manual/) | Add a service to a Compose file, or run the Python program | An integration that is not in the catalog, or a deployment without Enterprise Edition |

You can combine them: the default integrations run as Compose services while XTM Composer runs the ones you add later.

## The default integrations of xtm-docker

[FiligranHQ/xtm-docker](https://github.com/FiligranHQ/xtm-docker) always starts these integrations with the stack: they are defined next to their platform, in `modules/opencti.yml` and `modules/openaev.yml`. Each one runs as a Compose service with `restart: always`, starts once its platform is healthy, and uses the admin token of its platform (`OPENCTI_ADMIN_TOKEN` or `OPENAEV_ADMIN_TOKEN` in `.env`). Its ID is in `.env` too.

::::tabs
:::tab[OpenCTI connectors]
| Service | What it does |
| --- | --- |
| `connector-export-file-stix` | Exports data as STIX 2.1 |
| `connector-export-file-csv` | Exports data as CSV |
| `connector-export-file-txt` | Exports data as plain text |
| `connector-import-file-stix` | Imports STIX 2.1 bundles |
| `connector-import-document` | Imports PDF, HTML and text documents and extracts their knowledge |
| `connector-import-file-yara` | Imports YARA rules |
| `connector-analysis` | Analyzes PDF, HTML and text documents |
| `connector-import-external-reference` | Imports the content of external references |
| `connector-opencti` | Loads the OpenCTI datasets: marking definitions, identities, locations |
| `connector-mitre` | Loads MITRE ATT&CK |

The `connector-opencti` and `connector-mitre` services set `CONNECTOR_AUTO_CREATE_SERVICE_ACCOUNT=true`. At start, each one creates a service account named `[C] <connector name>` in the `Connectors` group, then runs with its token instead of the admin token.
:::
:::tab[OpenAEV collectors and injectors]
| Service | Kind | What it does |
| --- | --- | --- |
| `collector-mitre-attack` | Collector | Loads MITRE ATT&CK attack patterns |
| `collector-openaev` | Collector | Loads the OpenAEV datasets |
| `collector-atomic-red-team` | Collector | Loads the Atomic Red Team tests |
| `collector-nvd-nist-cve` | Collector | Loads CVEs from the NVD. Set `COLLECTOR_NVD_NIST_CVE_API_KEY` in `.env`: optional, recommended |
| `injector-nmap` | Injector | Runs Nmap network scans |
| `injector-nuclei` | Injector | Runs Nuclei vulnerability scans |
:::
::::

## See them run

Each platform lists the integrations that registered with it, managed by XTM Composer or not:

- OpenCTI: :menu[Integrations], **Deployed** tab (:menu[Data > Ingestion > Connectors] before 7.260811) shows each connector, its status and the messages waiting in its RabbitMQ queue.
- OpenAEV: **Integrations** > **Collectors** and **Integrations** > **Injectors** show each collector and injector and its status.

From the host, the containers of the default integrations are Compose services:

```bash
docker compose ps --format '{{.Service}} {{.Status}}' | grep -E '^(connector|collector|injector)-'
```

:::verify
Each default integration is `Up`. A container that keeps restarting cannot reach its platform: run `docker compose logs <service>` to see why.
:::

## What's next

- [XTM Composer](/deploy/integrations/composer/), to run integrations from the platform interface.
- [Run an integration without XTM Composer](/deploy/integrations/manual/), for Docker and Python deployments, networking and tokens.
- [Integration model](/develop/integration-model/), to build your own integration.
