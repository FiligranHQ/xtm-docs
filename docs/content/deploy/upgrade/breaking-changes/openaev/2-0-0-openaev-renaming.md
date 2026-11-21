---
title: OpenAEV renaming
description: Upgrade from OpenBAS to OpenAEV 2.0.0, with the renamed images, variables, bucket, queues and indices, and the OpenAEV agents to reinstall.
doc_type: how-to
products:
  - openaev
personas: [platform-engineer]
owner: docs-platform
last_reviewed: 2026-10-31
migrated_from:
  - openaev:/deployment/breaking-changes/2.0.0-openaev-renaming/
---

OpenBAS (Open Breach and Attack Simulation) became OpenAEV (Open Adversarial Exposure Validation) in version 2.0.0. The upgrade renames images, configuration variables and storage names. Most of it is automatic; this page lists what you check or change by hand.

::diagram{id=breaking-openaev-2-0-0-openaev-renaming}

| | Version |
| --- | --- |
| Deprecated in | OpenBAS 1.18.20 |
| Removed in | OpenAEV 2.0.0 |

The first start of OpenAEV 2.0.0 takes longer than usual: it applies every change.

## Before you start

- Let the running simulations finish: the RabbitMQ queues are renamed, and a simulation in progress during the upgrade can be affected.

## What to change

| What | Before | From 2.0.0 | What to do |
| --- | --- | --- | --- |
| Docker images | `openbas/*` | `openaev/*`, for example `openaev/platform` | Pull the images from the [OpenAEV namespace on Docker Hub](https://hub.docker.com/u/openaev). xtm-docker already uses it |
| Configuration | `OPENBAS_*` variables, such as `OPENBAS_PROVIDER_XXX` for SAML | `OPENAEV_*`, such as `OPENAEV_PROVIDER_XXX` | Migrated on its own. If OpenAEV does not start, compare your variables with the [OpenAEV configuration reference](/reference/openaev/configuration/), SAML ones first |
| CrowdStrike response scripts | `OpenBAS Subprocessor (XXX)` | `OpenAEV Subprocessor (XXX)` | Check the response script names in the CrowdStrike settings of your platform properties or environment variables: see [How integrations run](/deploy/integrations/how-they-run/) |
| S3 bucket (`minio.bucket`) | `openbas` | `openaev` | If you used the default bucket, move its content to `openaev`, or set `MINIO_BUCKET=openbas` to keep it |
| RabbitMQ queue prefix (`openaev.rabbitmq.prefix`) | `openbas` | `openaev` | Let running simulations finish before the upgrade |
| Elasticsearch or OpenSearch index prefix (`engine.index-prefix`) | `openbas` | `openaev` | After the migration, delete the `openbas` indices to free space |

## Reinstall the OpenAEV agents installed as user services

OpenAEV moves the agent folders on the assets from OpenBAS to OpenAEV, except for agents installed as user services through the advanced installation. The old folder keeps the attack logs and execution files, so no data is lost.

:::steps
1. On each endpoint whose agent runs as a user service, uninstall the OpenBAS agent, with the command for its operating system in [OpenAEV agents](/openaev/use/agents/).
2. Install the OpenAEV agent on it.

   :::verify
   The endpoint shows a new OpenAEV agent. The old OpenBAS agent is obsolete.
   :::
:::

## What's next

- [Upgrade](/deploy/upgrade/), the upgrade procedure.
- [OpenAEV agents](/openaev/use/agents/), to install and manage agents.
- [Breaking changes](/deploy/upgrade/breaking-changes/), the other changes per version.
