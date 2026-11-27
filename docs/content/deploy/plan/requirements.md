---
title: Requirements
description: What the host and the shared services must provide to run OpenCTI, OpenAEV, OpenCRQ and XTM One, with the versions each module supports.
doc_type: reference
products:
  - opencti
  - openaev
personas: [platform-engineer]
owner: docs-platform
last_reviewed: 2026-11-26
sidebar_position: 3
---

This page lists what a deployment of the XTM stack needs before you install it: the host for the Docker deployment, the versions of the shared services each module supports and the settings they need. For the amount of CPU, memory and disk, see [Sizing](/deploy/plan/sizing/).

::diagram{id=deploy-plan-requirements}

## The host, for the Docker deployment

[FiligranHQ/xtm-docker](https://github.com/FiligranHQ/xtm-docker) needs:

| Requirement | Value |
| --- | --- |
| Docker Engine | 20.10 or later |
| Docker Compose | 2.24.4 or later: `docker compose version` prints it. The `docker-compose.yml` of xtm-docker uses `required: false` under `depends_on`, which needs 2.20, and the `!reset` and `!override` tags of the override files in this documentation need 2.24.4. An earlier v2 stops on the first of them |
| Processor | `linux/amd64`. XTM One images exist for this platform only: on an ARM machine they run through emulation. On a Mac with an Apple chip, turn on **Use Rosetta for x86_64/amd64 emulation** in Docker Desktop |
| Memory and disk | 16 GB of RAM and 50 GB of free disk at least: see [Sizing](/deploy/plan/sizing/) |
| Free ports | The ports the stack publishes: see [Network and ports](/deploy/plan/network/) |
| Kernel setting (Linux) | `vm.max_map_count` raised for Elasticsearch, below |

### Raise vm.max_map_count on Linux

Elasticsearch needs more memory map areas than Linux allows by default. The OpenCTI installation guide sets `vm.max_map_count` to 1048575; the xtm-docker troubleshooting section uses 262144. 1048575 covers both.

:::steps
1. Set the value for the running system.

   ```bash
   sudo sysctl -w vm.max_map_count=1048575
   ```

2. Keep it after a restart.

   ```bash
   echo "vm.max_map_count=1048575" | sudo tee -a /etc/sysctl.conf
   ```

   :::verify
   ```bash
   sysctl vm.max_map_count
   ```
   It prints `vm.max_map_count = 1048575`.
   :::
:::

## Shared services

The versions each module supports, and the version xtm-docker runs. A module marked "Not used" does not connect to that service.

| Service | xtm-docker runs | OpenCTI | OpenAEV | XTM One | OpenCRQ |
| --- | --- | --- | --- | --- | --- |
| Elasticsearch or OpenSearch | Elasticsearch 8.19.18 | Elasticsearch 8.0 or later, or OpenSearch 2.9 or later | Elasticsearch 8.19 or later | Not used | Not used |
| RabbitMQ | 4.3, with the management plugin | 3.11 or later | 4.1 or later | Not used | Not used |
| Redis | 8.8.0 | 7.1 or later | Not used | Used, no minimum version documented | Not used |
| S3-compatible storage | RustFS 1.0.4 | Any S3-compatible service; MinIO RELEASE.2023-02 or later | MinIO RELEASE.2025-06-13T11-33-47Z or later | Used, no minimum version documented | Any S3-compatible service, its bucket created beforehand |
| PostgreSQL | 17, with the pgvector extension | Not used | 17 or later | With the pgvector extension | 17 recommended, 16 at least; it creates the `unaccent` extension |

The OpenAEV versions are the recommended ones: OpenAEV may work with earlier versions, but they are not supported. OpenAEV 3.261029.0 detects the search engine from its URL and works with Elasticsearch 8, Elasticsearch 9 or OpenSearch. OpenCTI also works with search, cache and queue services that speak the same protocols, such as managed cloud services: see [Topologies](/deploy/plan/topologies/).

### Settings the shared services need

xtm-docker already sets these. Set them yourself when you run the services another way.

| Service | Setting | Why |
| --- | --- | --- |
| Elasticsearch | `thread_pool.search.queue_size=5000` | Filigran recommends it for OpenCTI on a new installation |
| RabbitMQ | `max_message_size = 536870912` and `consumer_timeout = 86400000` in `rabbitmq.conf` | Filigran recommends them for OpenCTI on a new installation |
| PostgreSQL | `max_connections=200` | OpenAEV opens up to 20 connections, XTM One up to 35 per process over its two web workers and its worker, and OpenCRQ about 50 at peak: more than the default of 100 |
| PostgreSQL | The `vector` extension in the `xtm_one` database, created by a superuser | pgvector is not a trusted extension, and the XTM One role is not a superuser. In xtm-docker, `xtm-one-db-init` creates it |
| PostgreSQL in a container | At least 1 GB of shared memory: `shm_size: 1gb` in Docker Compose, a memory `emptyDir` on Kubernetes | Parallel queries pass their rows through `/dev/shm`: with the 64 MB a container gets by default, the large queries of OpenCRQ fail with `could not resize shared memory segment`. See [Configure OpenCRQ](/deploy/configure/opencrq/#database) |

## What's next

- [Sizing](/deploy/plan/sizing/): CPU, memory and disk for each component.
- [Network and ports](/deploy/plan/network/): the ports to open.
- [Install with Docker Compose](/deploy/install/docker-compose/).
