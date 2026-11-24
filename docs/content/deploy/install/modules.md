---
title: Start the whole stack on an existing deployment
description: Bring a deployment that ran only some modules of the XTM stack to the whole stack, which xtm-docker now always starts, with what to check and what happens to the data.
doc_type: how-to
products:
  - suite
personas: [platform-engineer]
owner: docs-platform
last_reviewed: 2026-11-13
---

The official Docker deployment, [FiligranHQ/xtm-docker](https://github.com/FiligranHQ/xtm-docker), has no optional module: `docker compose up -d` starts the whole XTM platform, 32 services. XTM One, OpenCTI, OpenAEV, OpenCRQ, XTM Composer, their default integrations and the shared services always run together. This page brings a deployment that ran only some of them, with a `COMPOSE_PROFILES` line in `.env`, to the whole stack: what to clean in `.env`, what to check on the host, and what happens to the data.

::diagram{id=deploy-install-modules}

## Before you start

- A stack installed from xtm-docker, as [Install with Docker Compose](/deploy/install/docker-compose/) describes. Run the commands of this page from its folder.
- For OpenCRQ, the admin of XTM One, who signs in to it with XTM One and lands in its first tenant, `Default`, which OpenCRQ creates at its first start: see [Install OpenCRQ](/deploy/install/opencrq/).
- A host sized for the whole stack: 16 GB of RAM at least, 32 GB recommended for production, and 50 GB of free disk. [Sizing](/deploy/plan/sizing/) says how much each module needs.
- A backup of the volumes and of `.env`: see [Back up and restore](/deploy/operate/backup/).

## What the stack runs

`docker-compose.yml` includes one file per shared service, under `infra/`, and one file per module, under `modules/`. Every service they list starts.

| File | Services |
| --- | --- |
| `modules/xtm-one.yml` | `xtm-one`, `xtm-one-worker`, `xtm-one-db-init` |
| `modules/opencti.yml` | `opencti`, `worker`, and the default connectors: `connector-export-file-stix`, `connector-export-file-csv`, `connector-export-file-txt`, `connector-import-file-stix`, `connector-import-document`, `connector-import-file-yara`, `connector-analysis`, `connector-import-external-reference`, `connector-opencti`, `connector-mitre` |
| `modules/openaev.yml` | `openaev`, and the default collectors and injectors: `collector-mitre-attack`, `collector-openaev`, `collector-atomic-red-team`, `collector-nvd-nist-cve`, `injector-nmap`, `injector-nuclei` |
| `modules/opencrq.yml` | `opencrq`, `opencrq-db-init`, `opencrq-bucket-init` |
| `modules/xtm-composer.yml` | `xtm-composer`, `rsa-key-generator` |
| `infra/*.yml` | `elasticsearch`, `rabbitmq`, `redis`, `pgsql` (PostgreSQL with pgvector), `minio` (RustFS, the S3 storage) |

XTM Composer always manages both OpenCTI and OpenAEV.

## Move to the whole stack

:::steps
1. Get the current deployment.

   ```bash
   git pull
   ```

2. Remove the lines that no longer do anything from `.env`: `COMPOSE_PROFILES`, which listed the modules to start, and `XTM_COMPOSER_OPENCTI_ENABLE` and `XTM_COMPOSER_OPENAEV_ENABLE`, which turned XTM Composer off for one platform.

   ```bash
   sed -i.bak -E '/^(COMPOSE_PROFILES|XTM_COMPOSER_OPENCTI_ENABLE|XTM_COMPOSER_OPENAEV_ENABLE)=/d' .env
   ```

3. Check that the modules you did not run have your own secrets, not the values of `.env.sample`. OpenCTI does not start with the sample values.

   ```bash
   grep -E '^(OPENCTI|OPENAEV)_(ADMIN_(PASSWORD|TOKEN|ENCRYPTION_KEY|ENCRYPTION_SALT)|ENCRYPTION_KEY|HEALTHCHECK_(ACCESS_)?KEY)=' .env
   ```

   A value such as `changeme`, `ChangeMe_UUIDv4`, `00000000-0000-0000-0000-000000000000` or an empty one needs a fresh one: a UUID for the admin tokens and health check keys, a 32-byte base64 key for `OPENCTI_ENCRYPTION_KEY` (`openssl rand -base64 32`), and a long random string for the others (`openssl rand -hex 24`). Set the admin email and the public URL of each new platform too, as [Configure the stack](/deploy/install/docker-compose/#configure-the-stack) describes.

4. Prepare the host for the services you did not run. If Elasticsearch did not run, raise `vm.max_map_count` to `262144`, as [Prepare the host](/deploy/install/docker-compose/#prepare-the-host) does. If `docker-compose.override.yml` binds the published ports to the host itself, add the services that were not in it, such as `openaev` or `elasticsearch`.

5. If you pinned the versions, run the commands of [Pin the image versions](/deploy/install/docker-compose/#pin-the-image-versions) again: they pinned only the images of the services that ran, and now pin those of the whole stack.

6. Download the images, then start the stack.

   ```bash
   docker compose pull
   docker compose up -d
   ```

   Docker Compose creates the services that did not run, recreates those whose configuration changed, such as `xtm-composer`, and leaves the others alone. The first start of OpenCTI or OpenAEV creates its indices and databases: allow a few minutes.

   :::verify
   ```bash
   docker compose config --services | wc -l
   docker compose ps --format '{{.Service}} {{.Health}}' | grep -E '^(opencti|openaev|xtm-one|opencrq) '
   ```
   The first command prints `32`. Each platform then says `healthy`.
   :::
:::

Then register the new platforms with XTM One and check the links between the modules, as [Connect the modules to each other](/deploy/configure/connect-modules/) describes.

## What happens to the data

The data of the modules that already ran stays in their volumes, and a platform that starts for the first time creates its own beside it. The volumes are shared between modules:

| Volume | Shared by |
| --- | --- |
| `esdata` | OpenCTI and OpenAEV |
| `pgsqlshareddata` | OpenAEV, XTM One and OpenCRQ |
| `s3data` | OpenCTI, OpenAEV, XTM One and OpenCRQ |
| `redisdata` | OpenCTI and XTM One |
| `amqpdata` | OpenCTI and OpenAEV |
| `rsakeys` | XTM Composer |

Their names start with `COMPOSE_PROJECT_NAME`, `xtm_` by default.

## Stop the whole stack

`docker compose down` stops every container of the stack. The volumes stay. `docker compose down -v` deletes them, with the data of every module.

## What's next

- [Connect the modules to each other](/deploy/configure/connect-modules/), after a platform starts for the first time.
- [XTM Composer](/deploy/integrations/composer/), to run more integrations than the default ones.
- [Back up and restore](/deploy/operate/backup/), to keep the volumes and `.env` together.
