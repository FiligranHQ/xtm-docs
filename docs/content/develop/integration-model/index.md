---
title: Integration model
description: How the XTM products connect to other systems through connectors, collectors, injectors and executors, and what every integration shares, OpenCTI first.
doc_type: explanation
products:
  - opencti
  - openaev
  - composer
personas: [developer, platform-engineer]
owner: docs-develop
last_reviewed: 2026-11-01
---

An integration is a program that connects an XTM product to another system: a threat feed, a SIEM, a sandbox, an endpoint security tool. OpenCTI has connectors; OpenAEV has collectors, injectors and executors. This page explains what the four kinds share and where they differ, so you can pick the kind to build, the library to build it with, and how it will be deployed.

::diagram{id=develop-integration-model}

## The four kinds

| Kind | Product | Direction | What it does | Library | Source repository |
| --- | --- | --- | --- | --- | --- |
| [Connector](/develop/integration-model/connectors/) | OpenCTI | In and out | Imports, enriches, analyzes, streams or exports threat intelligence as STIX 2.1 | [pycti](/develop/build/python-client/), and `connectors-sdk` on top of it | [OpenCTI-Platform/connectors](https://github.com/OpenCTI-Platform/connectors) |
| [Collector](/develop/integration-model/collectors/) | OpenAEV | In | Reads detection and prevention results, or reference data, from a security tool or a source | `pyoaev` | [OpenAEV-Platform/collectors](https://github.com/OpenAEV-Platform/collectors) |
| [Injector](/develop/integration-model/injectors/) | OpenAEV | Out | Runs the injects of a simulation: sends emails, runs scans, calls other tools | `pyoaev` | [OpenAEV-Platform/injectors](https://github.com/OpenAEV-Platform/injectors) |
| [Executor](/develop/integration-model/executors/) | OpenAEV | Out | Deploys and runs OpenAEV agents on endpoints, natively or through an endpoint security product | Built into OpenAEV | [OpenAEV-Platform/openaev](https://github.com/OpenAEV-Platform/openaev) |

To get data into OpenCTI or out of it, build a connector. To measure what your defenses saw during a simulation, build a collector; to make a simulation act on a system, build an injector.

## What every integration shares

The kinds differ in what they exchange, not in how they live next to their platform:

- **A process of its own.** An external integration runs as its own container or Python process, never inside the platform. It can crash, restart or scale without touching the platform.
- **An identity.** It has a unique identifier, a UUIDv4 (`CONNECTOR_ID`, `COLLECTOR_ID` or `INJECTOR_ID`), and a display name. The platform keeps its state and its history under that identifier, so keep it across restarts and redeployments.
- **A user and a token.** It calls the platform API with the token of a user (`OPENCTI_TOKEN`, `OPENAEV_TOKEN`). In OpenCTI, that user decides what the connector may read and write and with which confidence level: give each connector its own user.
- **A registration.** At start, it registers with the platform through the API. The registration returns what it needs next: for an OpenCTI connector, the RabbitMQ connection and the names of its queues, and the state it saved at its last run.
- **A heartbeat.** While it runs, it reports to the platform, which shows it as active. An OpenCTI connector pings every 40 seconds and sends its state with each ping; after 5 minutes without a heartbeat, OpenCTI shows it as inactive.
- **The same configuration rule.** An environment variable wins over the key of `config.yml`, which wins over the default of the code. In Docker, you set environment variables; in development, `config.yml` is often easier.

## OpenCTI connectors in one paragraph

A connector speaks STIX 2.1. An import or enrichment connector does not write objects one by one: it builds a STIX bundle and sends it to its RabbitMQ queue, and the OpenCTI workers write it, with the rights and the confidence level of the connector's user. Work that the platform asks for, such as an enrichment, a file import or an export, reaches the connector through a second queue. A stream connector follows a live stream of the platform instead. [Connectors](/develop/integration-model/connectors/) details each type: what it receives, what it returns, as which user.

## How integrations are deployed

| Way | Who runs the container | What the integration must provide |
| --- | --- | --- |
| [XTM Composer](/deploy/integrations/composer/) | XTM Composer, on Docker, Podman, Kubernetes, Docker Swarm or Portainer, from the catalog of the platform. Requires the Enterprise Edition | A Docker image, and an entry in a catalog with a configuration schema: see [Integration manifest](/develop/build/manifest/). An XTM Composer that sets a tag suffix, `-fips` in its FIPS image, pulls the `-fips` or `-ubi9` tag of that image |
| [By hand](/deploy/integrations/manual/) | You, with Docker Compose, Kubernetes or Python | A Docker image or the Python sources, and a `README.md` listing its settings |

XTM Composer reads what it deploys from the catalog: the image, its version, and the settings to ask for in the form. A connector that should be deployable from the interface therefore ships a manifest next to its code. [Community resources](/develop/integration-model/resources/) lists where the catalogs, the repositories and the SDKs live.

## What's next

- [Connectors](/develop/integration-model/connectors/), the connector types of OpenCTI.
- [Build an OpenCTI connector](/develop/build/connector/), a tutorial from an empty folder to objects in OpenCTI.
- [Script OpenCTI in Python](/develop/build/python-client/), to script the OpenCTI API.
- [How integrations run](/deploy/integrations/how-they-run/), the deployment side.
