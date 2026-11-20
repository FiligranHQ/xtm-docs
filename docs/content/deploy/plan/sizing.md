---
title: Sizing
description: How much CPU, memory and disk the XTM stack needs, on one host with xtm-docker and component by component for production.
doc_type: reference
products:
  - opencti
  - openaev
personas: [platform-engineer]
owner: docs-platform
last_reviewed: 2026-10-31
sidebar_position: 4
---

This page gives the CPU, memory and disk the XTM stack needs: first for the whole stack on one host with [FiligranHQ/xtm-docker](https://github.com/FiligranHQ/xtm-docker), then the minimum for each component when you run them on separate machines, and the settings that decide how much memory each process takes.

::diagram{id=deploy-plan-sizing}

## One host with xtm-docker

| Resource | Minimum | Recommended |
| --- | --- | --- |
| RAM | 16 GB | 32 GB for production |
| Free disk | 50 GB | |

Elasticsearch takes the largest share. Its Java heap is `ELASTIC_MEMORY_SIZE` in `.env`, `4G` in `.env.sample`. Set it from the RAM of the host:

| RAM of the host | `ELASTIC_MEMORY_SIZE` |
| --- | --- |
| 16 GB | `2G` |
| 32 GB | `4G` |
| 64 GB or more | `8G` |

:::verify[Check the heap Elasticsearch runs with]
```bash
docker compose exec elasticsearch sh -c 'echo $ES_JAVA_OPTS'
```
It prints `-Xms2G -Xmx2G` for `ELASTIC_MEMORY_SIZE=2G`.
:::

## Each component, for production

The minimum for each component on its own machine or container, as the OpenCTI and OpenAEV documentation give them. A service shared by several modules holds the data of all of them.

### Shared services

| Service | Needed by | CPU | RAM | Disk type | Disk space |
| --- | --- | --- | --- | --- | --- |
| Elasticsearch or OpenSearch | OpenCTI, OpenAEV | 2 cores | 8 GB or more | SSD | 16 GB or more |
| PostgreSQL | OpenAEV | 2 cores | 8 GB or more | SSD | 16 GB or more |
| Redis | OpenCTI | 1 core | 1 GB or more | SSD | 16 GB or more |
| RabbitMQ | OpenCTI, OpenAEV | 1 core | 512 MB or more | Standard | 2 GB or more |
| S3 storage | OpenCTI, OpenAEV | 1 core | 128 MB or more | SSD | 16 GB or more |

### Platforms and integrations

The platforms, workers and integrations keep no data on disk: their state is in the shared services.

::::tabs
:::tab[OpenCTI]
| Component | CPU | RAM |
| --- | --- | --- |
| OpenCTI platform | 2 cores | 8 GB or more |
| Each worker | 1 core | 128 MB or more |
| Each connector | 1 core | 128 MB or more |
| XTM Composer | 1 core | 128 MB or more |
:::
:::tab[OpenAEV]
| Component | CPU | RAM |
| --- | --- | --- |
| OpenAEV platform | 2 cores | 8 GB or more |
| Each injector | 1 core | 128 MB or more |
| Each collector | 1 core | 128 MB or more |
:::
::::

No figures are documented yet for XTM One and its worker, nor for the PostgreSQL and Redis load XTM One adds.

## What decides the memory of each process

| Process | Setting | In xtm-docker |
| --- | --- | --- |
| OpenCTI platform | The Node.js heap limit, `NODE_OPTIONS=--max-old-space-size=<MB>`. Raise it if the platform stops with out-of-memory errors | `--max-old-space-size=8096`, about 8 GB |
| Elasticsearch | The Java heap, `ES_JAVA_OPTS` | `-Xms` and `-Xmx` set to `ELASTIC_MEMORY_SIZE` |
| Redis | The length of the OpenCTI event stream, `REDIS__TRIMMING`: 2,000,000 events by default, about 8 GB of memory | Default |
| RabbitMQ | RabbitMQ takes memory up to its own threshold: set it together with the memory limit of its container | Default |
| S3 storage | RustFS has no memory setting in xtm-docker: limit it with the memory limit of its container | Default |
| OpenCRQ | The heaps of its two processes: 4,096 MB for the worker (`PROCESS_MANAGER__SPLIT__WORKER__MEMORY`) and 512 MB for the HTTP server (`PROCESS_MANAGER__SPLIT__HTTP__MEMORY`) | Default |
| Workers, connectors, collectors, injectors | Python processes: limit their memory with the limits of their containers | No limit |
| PostgreSQL | `max_connections`: OpenAEV opens up to 20 connections, XTM One up to 35 per process over its two web workers and its worker, OpenCRQ about 50 at peak | `200` |

## Workers

xtm-docker starts three OpenCTI workers. More workers write faster, until the throughput of Elasticsearch becomes the limit. Count one OpenCTI platform for every three workers, and no more than three to four workers per platform: nine workers are spread over three platforms. Why, and how the platform and its workers share the work, is in [OpenCTI architecture](/opencti/concepts/architecture/).

To change the number of workers in xtm-docker, set `replicas` under `worker` > `deploy` in `docker-compose.override.yml`, as [Change the stack without editing docker-compose.yml](/deploy/install/docker-compose/#change-the-stack-without-editing-docker-compose-yml) describes, then run `docker compose up -d`. To add platforms, see [Scale: workers, platforms and clusters](/deploy/operate/scale/).

## What's next

- [Requirements](/deploy/plan/requirements/): versions and settings of the shared services.
- [Topologies](/deploy/plan/topologies/): one host, a cluster or Kubernetes.
- [Scale: workers, platforms and clusters](/deploy/operate/scale/).
