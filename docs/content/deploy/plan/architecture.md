---
title: "The XTM stack: modules and shared services"
description: How the XTM stack is built around XTM One, which shared service each module uses, and how the modules connect to each other.
doc_type: explanation
products:
  - suite
personas: [platform-engineer, evaluator]
owner: docs-platform
last_reviewed: 2026-11-27
sidebar_position: 1
migrated_from:
  - opencti:/deployment/overview/
  - openaev:/deployment/platform/overview/
---

The XTM platform runs as one stack. XTM One is at its center. OpenCTI, OpenAEV, OpenCRQ, their default integrations and XTM Composer always start with it, and all of them share one set of backing services: Elasticsearch, RabbitMQ, Redis, S3 storage and PostgreSQL. This page explains what each part is, which shared service each module uses and how the modules talk to each other, as the official Docker deployment, [FiligranHQ/xtm-docker](https://github.com/FiligranHQ/xtm-docker), defines them. How each module works inside is on its own architecture page.

## What the stack is made of

| Part | What it is | In xtm-docker |
| --- | --- | --- |
| Modules | The XTM applications: XTM One, OpenCTI, OpenAEV, OpenCRQ, XTM Composer and the default integrations | One Compose file per module, in `modules/`. Every module always starts |
| Shared services | The search engine, queue, cache, file storage and database the modules keep their data in | One instance of each, one Compose file per service, in `infra/` |
| Init jobs | Short tasks that prepare something for a module, then stop or wait | `xtm-one-db-init`, `opencrq-db-init`, `opencrq-bucket-init` and `rsa-key-generator` |

## The modules

### XTM One

The AI layer at the center of the platform. It runs as two services: `xtm-one`, the web application and its API, and `xtm-one-worker`, which runs the background work. Before XTM One starts, the `xtm-one-db-init` job creates its PostgreSQL role, its `xtm_one` database and the pgvector extension. XTM One images are published for `linux/amd64` only. See [XTM One](/xtm-one/).

### OpenCTI

The threat intelligence platform. It runs as the `opencti` service and the `worker` service, with three worker replicas in xtm-docker. Its default connectors run with it. How the platform, its workers and its connectors work together is in [OpenCTI architecture](/opencti/concepts/architecture/).

### OpenAEV

The exposure validation platform, as the `openaev` service. Its default collectors and injectors run with it. How the platform, its agents and its integrations work together is in [OpenAEV architecture](/openaev/concepts/architecture/).

### OpenCRQ

The cyber risk quantification platform, as the `opencrq` service. Its image is `filigran/opencrq`: see [What xtm-docker runs for OpenCRQ](/deploy/configure/opencrq/#what-xtm-docker-runs-for-opencrq). Before it starts, the `opencrq-db-init` job creates its PostgreSQL role, which is not a superuser, and the `opencrq` database that role owns, and the `opencrq-bucket-init` job creates its bucket. It reads threats from OpenCTI and validation results from OpenAEV, and registers with XTM One, without which it does not start. Inside the container, a process manager supervises an `http` process, for the interface and the API, and a `worker` process, for the background jobs and the connectors, or a single `main` process in the `monolith` profile. It answers the health check for all of them, and reports unhealthy when one stops sending heartbeats for `PROCESS_MANAGER__HEARTBEAT_TIMEOUT`; its Prometheus endpoint merges the metrics of every process. How they work together is in [OpenCRQ architecture](/opencrq/concepts/architecture/).

### XTM Composer

XTM Composer deploys and runs integrations from the interface of OpenCTI and OpenAEV. It manages the integrations of both platforms, and starts once they are healthy. It starts integration containers through the Docker socket of the host, on the network of the stack. For an integration on a moving tag (`latest`, `rolling` or no tag), it asks the registry which image the tag points to, every 10 minutes by default, and redeploys the integration on a new image. Before it starts, the `rsa-key-generator` job creates its RSA private key: XTM Composer gives the public key to OpenCTI and OpenAEV, which encrypt the secrets of the integrations with it, and decrypts them before it starts a container. See [XTM Composer](/deploy/integrations/composer/).

## The shared services

| Shared service | Compose service | What it holds | Compose file |
| --- | --- | --- | --- |
| Elasticsearch | `elasticsearch` | The knowledge of OpenCTI and OpenAEV, and its search indices | `infra/elasticsearch.yml` |
| RabbitMQ | `rabbitmq` | The queues between the platforms, their workers and their integrations | `infra/rabbitmq.yml` |
| Redis | `redis` | Cache, sessions and live events | `infra/redis.yml` |
| S3 storage (RustFS) | `minio` | Files, imports and exports | `infra/rustfs.yml` |
| PostgreSQL with pgvector | `pgsql` | The databases of OpenAEV, OpenCRQ and XTM One | `infra/postgresql.yml` |

Every shared service starts with the stack. Each module waits until the shared services it depends on are healthy before it starts.

## Which module uses which service

| Module | Elasticsearch | RabbitMQ | Redis | S3 storage | PostgreSQL |
| --- | --- | --- | --- | --- | --- |
| XTM One | No | No | Yes | Yes, bucket `xtm-one-files` | Yes, database `xtm_one` |
| OpenCTI | Yes | Yes | Yes | Yes, bucket `opencti-bucket` | No |
| OpenAEV | Yes | Yes | No | Yes, bucket `openaev` | Yes, database `openaev` |
| OpenCRQ | No | No | No | Yes, bucket `opencrq` | Yes, database `opencrq` |
| XTM Composer | No | No | No | No | No |
| Default integrations | No | Connectors and injectors | No | No | No |

One instance of each service is enough because each module keeps to its own data. OpenCTI and OpenAEV write Elasticsearch indices under their own prefix, `opencti` and `openaev` by default. Each module has its own bucket. OpenAEV and XTM One each have their own database, and XTM One connects with its own role, which is not the PostgreSQL superuser.

Two rules surprise people:

- XTM Composer uses no shared service. It talks to the APIs of OpenCTI and OpenAEV, to Docker, and over HTTPS to the registry of each integration on a moving tag, such as the `rolling` images of the OpenAEV catalog.
- By default, OpenCTI connectors and OpenAEV injectors connect to RabbitMQ, although nothing in their configuration names it. They use the connection details their platform gives them when they register, so they must be able to reach RabbitMQ under the host name the platform knows it by.

## How the modules connect

::diagram{id=deploy-plan-architecture}

- **OpenCTI, OpenAEV and OpenCRQ register with XTM One**, and XTM One calls their APIs. They all share one secret, `PLATFORM_REGISTRATION_TOKEN`: every platform connected to the same XTM One must use the same value.
- **OpenCTI and OpenAEV call each other's API**, each with the admin token of the other.
- **OpenCRQ reads OpenCTI and OpenAEV** through its connectors, set with their admin tokens for its **Default** tenant.
- **XTM Composer manages the integrations of both platforms** through their APIs, with their admin tokens.
- **The default integrations talk to their platform**: OpenCTI connectors to OpenCTI, OpenAEV collectors and injectors to OpenAEV.

Inside the stack, the modules reach each other by service name, such as `http://opencti:8080`. Browsers reach them on public URLs built from `.env`. [Network and ports](/deploy/plan/network/) lists every address, port and token.

## What's next

- [What each XTM module brings](/deploy/plan/modules/): the services, shared services and settings of each module.
- [Network and ports](/deploy/plan/network/): who talks to whom, on which port.
- [Requirements](/deploy/plan/requirements/) and [Sizing](/deploy/plan/sizing/) for the machines.
- [How OpenCTI processes data in the background](/opencti/concepts/architecture/#managers): the platform managers.
