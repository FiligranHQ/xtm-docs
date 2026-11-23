---
title: Compatibility matrix
description: The OpenCTI, OpenAEV, XTM One and XTM Composer versions documented together, the version constraints between them, and the minimum versions of the shared services.
doc_type: reference
products:
  - suite
  - opencti
  - openaev
  - xtm-one
  - composer
personas: [platform-engineer]
owner: docs-platform
last_reviewed: 2026-11-21
---

This page ties the versions of the XTM modules together: the releases the documentation describes as one platform, the constraints that the [breaking changes](/deploy/upgrade/breaking-changes/) set between modules, and the versions of the shared services each platform needs. It lists only what has been checked: a pair of versions that is not here is not known to be incompatible, only not documented.

## Versions documented together

The documentation, and the stack its screenshots are taken from, run these versions together:

| Module | Version | Image |
| --- | --- | --- |
| OpenCTI | 7.261030.1 | `opencti/platform:7.261030.1` |
| OpenAEV | 3.261029.0 | `openaev/platform:3.261029.0` |
| XTM One | 1.261112.0 | `xtmone/platform:1.261112.0` |
| XTM Composer | 3.261121.0 | `filigran/xtm-composer:3.261121.0`, or `filigran/xtm-composer:3.261121.0-fips` for FIPS 140-3 |

The OpenCTI LTS lines, [7.260811-lts and 7.260309-lts](/deploy/upgrade/lifecycle/#supported-lts-lines), are documented for OpenCTI alone: no OpenAEV or XTM One version is documented with them.

## Constraints between modules

::diagram{id=deploy-upgrade-compatibility}

| When you run | You need | Source |
| --- | --- | --- |
| OpenCTI 7.260224.0 or later | XTM Composer 2 or later, for the new token management and the JSON Web Token (JWT) authentication of OpenCTI 7 | [OpenCTI 7.260224.0](/deploy/upgrade/breaking-changes/#opencti-7-260224-0) |
| OpenCTI 7.260224.0 or later, connected to OpenAEV | OpenAEV 2.2 or later | [OpenCTI 7.260224.0](/deploy/upgrade/breaking-changes/#opencti-7-260224-0) |
| OpenAEV 2.2.0, connected to OpenCTI | OpenCTI 7. OpenAEV 2.2.1 and later work with older OpenCTI versions again | [Scenario generation from OpenCTI security coverage](/deploy/upgrade/breaking-changes/openaev/2-2-0-security-coverage/) |
| OpenAEV 2.3.4 or later | Collectors, injectors, OpenAEV agents and custom integrations released for OpenAEV 2.3.4 or later | [CSRF token enforcement](/deploy/upgrade/breaking-changes/openaev/2-3-4-csrf-token-enforcement/) |
| XTM Composer 3.261120.0 or later, with `MANAGER__IMAGE_TAG_SUFFIX` set, as in its FIPS image (`-fips`) | Integrations published with that tag, for every platform XTM Composer serves. OpenAEV publishes no FIPS images: an integration without the tag is not deployed | [Configure XTM Composer](/deploy/integrations/configure-composer/) |
| XTM Composer 3.261120.0 or later, serving OpenAEV on Kubernetes, Docker Swarm or Portainer from a private registry | The registry in the `openaev` block, `OPENAEV__DAEMON__REGISTRY__*`: these engines no longer read the `opencti` block for OpenAEV | [Pull integrations from a private registry](/deploy/integrations/private-registry/) |
| XTM Composer 3.261021.1 or earlier, after a rollback from 3.261120.0 | Nothing more: 3.261120.0 keeps the `opencti-manager`, `opencti-connector-id` and `opencti-platform` labels and `OPENCTI_CONFIG_HASH` next to its `xtm-*` keys, so an older XTM Composer still recognizes the integrations | [XTM Composer architecture](/deploy/integrations/architecture/) |
| XTM Composer 3.261120.0 or earlier on Kubernetes or Docker Swarm, after a rollback from 3.261121.0 | Integration names that are valid Kubernetes names, 63 characters at most. 3.261121.0 shortens longer names and adds a hash; an older XTM Composer removes those Deployments and services, then fails to create them under the full name | [Troubleshoot XTM Composer](/deploy/integrations/troubleshoot/#an-integration-starts-then-fails) |
| XTM One 1.261112.0 or later, behind a reverse proxy | A proxy that serves HTTP/2 to browsers, buffers no response under `/api/` and keeps a read timeout above 15 seconds: the chat streams over HTTP | [XTM One 1.261112.0](/deploy/upgrade/breaking-changes/#xtm-one-1-261112-0) |
| xtm-docker with public and internal URLs that differ | XTM One, OpenCTI and OpenAEV releases that include [xtm-one#4883](https://github.com/XTM-One-Platform/xtm-one/pull/4883), [opencti#18585](https://github.com/OpenCTI-Platform/opencti/pull/18585) and [openaev#8143](https://github.com/OpenAEV-Platform/openaev/pull/8143) | [xtm-docker README](https://github.com/FiligranHQ/xtm-docker#readme) |

## Shared services

::diagram{id=deploy-upgrade-compatibility-services}

The minimum versions each platform needs, and the images xtm-docker runs. xtm-docker starts Elasticsearch and RabbitMQ only with OpenCTI or OpenAEV; XTM One uses PostgreSQL, Redis and the S3 storage.

| Service | OpenCTI needs | OpenAEV recommends | xtm-docker image |
| --- | --- | --- | --- |
| Elasticsearch | 8.0 or later | 8.19 or later | `docker.elastic.co/elasticsearch/elasticsearch:8.19.18` |
| OpenSearch, instead of Elasticsearch | 2.9 or later | Not documented | Not used |
| Redis | 7.1 or later | Not used | `redis:8.8.0` |
| RabbitMQ | 3.11 or later | 4.1 or later | `rabbitmq:4.3-management` |
| S3 storage (RustFS, MinIO) | MinIO RELEASE.2023-02 or later | MinIO RELEASE.2025-06-13T11-33-47Z or later | `rustfs/rustfs:1.0.4` |
| PostgreSQL | Not used | 17.0 or later | `pgvector/pgvector:pg17`, shared by OpenAEV, OpenCRQ and XTM One |

OpenAEV may work with older versions than those it recommends, but older versions are not supported. Sizing for each service is in [Requirements](/deploy/plan/requirements/).

## Images of xtm-docker

xtm-docker starts every XTM image with the `latest` tag, so one `docker compose pull` brings all modules to their latest releases together:

| Module | Images |
| --- | --- |
| XTM One | `xtmone/platform`, `xtmone/worker` |
| OpenCTI | `opencti/platform`, `opencti/worker` |
| OpenCTI default connectors | `opencti/connector-*` |
| OpenAEV | `openaev/platform` |
| OpenAEV default integrations | `openaev/collector-*`, `openaev/injector-*` |
| XTM Composer | `filigran/xtm-composer` |

To stay on given versions, set the tags yourself: see [Stay on a version or an LTS line](/deploy/upgrade/#stay-on-a-version-or-an-lts-line).

## What's next

- [Upgrade](/deploy/upgrade/), to move the stack to new versions.
- [Breaking changes](/deploy/upgrade/breaking-changes/), the details behind each constraint.
- [Requirements](/deploy/plan/requirements/), the resources each service needs.
