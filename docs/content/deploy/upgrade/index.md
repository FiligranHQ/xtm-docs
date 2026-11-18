---
title: Upgrade
description: Upgrade an XTM stack deployed with Docker Compose to new images, after a backup, and know what each platform migrates on its own.
doc_type: how-to
products:
  - suite
  - opencti
  - openaev
personas: [platform-engineer]
owner: docs-platform
last_reviewed: 2026-11-16
migrated_from:
  - opencti:/deployment/upgrade/
  - openaev:/deployment/upgrade/
---

This page upgrades an XTM stack deployed with the official Docker deployment, [FiligranHQ/xtm-docker](https://github.com/FiligranHQ/xtm-docker): back up, get the new deployment files, download the new images and start them. OpenCTI and OpenAEV migrate their databases when they start, so you can upgrade from any version to the latest one, even across several major releases.

::diagram{id=deploy-upgrade}

## Before you start

- Read the [breaking changes](/deploy/upgrade/breaking-changes/) between your version and the new one. Some need an action before or after the upgrade.
- Check the [compatibility matrix](/deploy/upgrade/compatibility/) when you upgrade one module and not the others.
- Plan a maintenance window: the platforms are stopped during the backup and while they migrate their data. The first start after an upgrade can take longer than usual.
- Run the commands from the `xtm-docker` folder that holds your `.env`.

## What migrates on its own

| What | When |
| --- | --- |
| OpenCTI database schema and data | When the new OpenCTI starts |
| OpenAEV database schema and data | When the new OpenAEV starts |
| XTM One database schema and data | When the new XTM One starts, as [Upgrade XTM One](/deploy/upgrade/xtm-one/) describes |
| Changes that need you | Listed per version in [Breaking changes](/deploy/upgrade/breaking-changes/), with a guide for each |

To go back to the version you had, restore the backup you take in step 1: the older images alone are not enough. XTM One refuses to start on a database that a newer version migrated: its log says `The database is ahead of this image`, and names the revision it does not know. Do not re-stamp the database to get past it: restore the backup, or deploy an XTM One image that contains that revision.

## Upgrade the stack

:::steps
1. Take a snapshot of Elasticsearch, as in [Back up the stack](/deploy/operate/backup/#back-up-the-stack): Elasticsearch cannot be restored from a copy of its volume. Then stop the stack and back up the other volumes. The volume names start with `COMPOSE_PROJECT_NAME` from `.env` (`xtm` by default).

   ```bash
   docker compose stop
   P=$(grep -E '^COMPOSE_PROJECT_NAME=' .env | cut -d= -f2)
   mkdir -p backup
   for v in pgsqlshareddata pgsqldata pgsqlxtmonedata s3data redisdata amqpdata rsakeys; do
     docker volume inspect "${P}_$v" >/dev/null 2>&1 || continue
     docker run --rm -v "${P}_$v":/data:ro -v "$PWD/backup":/backup alpine tar czf "/backup/$v.tgz" -C /data .
   done
   ```

   This copy suits a single machine. For a production deployment, see [Back up and restore](/deploy/operate/backup/).

   :::verify
   ```bash
   ls -lh backup
   ```
   It lists one `.tgz` file for each volume of your deployment but `esdata`, none of them empty.
   :::

2. Get the new deployment files.

   ```bash
   git pull
   ```

   The new files split the stack: `docker-compose.yml` includes one file per shared service, in `infra/`, and one per module, in `modules/`. The services, their settings and the volume names stay the same, so your data is kept. If `git pull` refuses because you changed `docker-compose.yml`, move your changes to `docker-compose.override.yml` first: see [Stay on a version or an LTS line](#stay-on-a-version-or-an-lts-line). If you pinned the versions, set the new ones now, as that section describes. If `docker volume ls` lists a volume whose name ends in `pgsqlxtmonedata`, your deployment still has two PostgreSQL containers: follow [Move to the shared PostgreSQL instance](#move-to-the-shared-postgresql-instance) instead of steps 2 and 3.

   The whole stack always starts, and XTM Composer always manages both OpenCTI and OpenAEV. A `COMPOSE_PROFILES` line, `XTM_COMPOSER_OPENCTI_ENABLE` or `XTM_COMPOSER_OPENAEV_ENABLE` left in `.env` does nothing: remove them.

   ```bash
   sed -i.bak -E '/^(COMPOSE_PROFILES|XTM_COMPOSER_OPENCTI_ENABLE|XTM_COMPOSER_OPENAEV_ENABLE)=/d' .env
   ```

   A service of `docker-compose.override.yml` with a `profiles:` line starts only while one of its profiles is turned on. Without `COMPOSE_PROFILES`, a service you added with a line such as `profiles: ["opencti-connectors"]` no longer starts: remove that line from it.

   :::verify
   ```bash
   docker compose config --quiet && echo "configuration OK"
   ```
   It prints `configuration OK`.
   :::

3. Download the new images, then start the stack.

   ```bash
   docker compose pull
   docker compose up -d
   ```

   Download first: Docker Compose would otherwise start the older image kept under the same `latest` tag. The platforms migrate their data as they start: allow several minutes.

4. Wait until the platforms are healthy.

   ```bash
   docker compose ps --format '{{.Service}} {{.Health}}' | grep -E '^(opencti|openaev|xtm-one|opencrq) '
   ```

   :::verify
   Each platform says `healthy`. From the host, each one answers its health check:

   ```bash
   set -a; . ./.env; set +a
   curl -fsS "http://localhost:$OPENCTI_PORT/health?health_access_key=$OPENCTI_HEALTHCHECK_ACCESS_KEY" >/dev/null && echo "OpenCTI OK"
   curl -fsS "http://localhost:8081/api/health?health_access_key=$OPENAEV_HEALTHCHECK_KEY" >/dev/null && echo "OpenAEV OK"
   curl -fsS "http://localhost:$XTM_ONE_PORT/api/health/ready" >/dev/null && echo "XTM One OK"
   ```

   The container of XTM One stays `healthy` even when its start fails, so its line checks readiness instead, which succeeds only once the migrations and the start are complete. Until then, `curl -sS "http://localhost:$XTM_ONE_PORT/api/health/ready"` answers `"status":"starting"`, with a `startup_error` that gives the reason when the start failed, such as `The database is ahead of this image` after a rollback without the backup.
   :::

5. Apply the actions the [breaking changes](/deploy/upgrade/breaking-changes/) of your new version ask for after the upgrade.
:::

If a platform does not become healthy, read its logs, for example `docker compose logs opencti`, and see [Troubleshoot the XTM stack](/deploy/operate/troubleshoot/).

## Tune the OpenCTI migrations

OpenCTI logs each data migration with the `[MIGRATION]` prefix, and runs each one once. Two of them read your configuration.

### Re-index of the targets relationships

On a platform that has not run it yet, a migration re-indexes the `targets` relationships of every region, country and sector. Its logs start with `Re-indexing targets for region / countries / sectors`. On a large platform, its bulk requests to Elasticsearch can time out and stop the start. Three keys set them; they are read only while this migration runs.

| Key | Default | What it does |
| --- | --- | --- |
| `migrations:reindex_targets_rel:max_bulk_operations` | `1000` | Lines of each bulk request, two for each location. Lower it if the migration fails on timeouts |
| `migrations:reindex_targets_rel:bulk_timeout` | `30m` | Timeout of each bulk request, as an Elasticsearch duration |
| `migrations:reindex_targets_rel:max_concurrency` | `2` | Bulk requests sent in parallel. Lower it to `1` on a loaded cluster |

For example, in `docker-compose.override.yml`, before you start the new version again:

```yaml
services:
  opencti:
    environment:
      - MIGRATIONS__REINDEX_TARGETS_REL__MAX_BULK_OPERATIONS=200
      - MIGRATIONS__REINDEX_TARGETS_REL__BULK_TIMEOUT=60m
      - MIGRATIONS__REINDEX_TARGETS_REL__MAX_CONCURRENCY=1
```

Remove these lines once the platform is healthy. The other migration keys are in the [Technical](/reference/opencti/configuration/#technical) section of the configuration reference.

### AI switch

When you upgrade from a version released before September 2025, a migration sets the **Generative AI (AI Insight, NLQ)** switch of :menu[Settings > Filigran Experience] from `ai:enabled` (`AI__ENABLED`): on when the key is unset or `true`, off when it is `false`. After that, you change the switch in the interface, and `ai:enabled` must still be `true` for OpenCTI to reach its model. The model itself is set as [Connect OpenCTI to your own language model](/deploy/configure/ai-extraction/#connect-opencti-to-your-own-language-model) describes.

## Move to the shared PostgreSQL instance

Earlier versions of xtm-docker ran two PostgreSQL containers: `pgsql` for OpenAEV and `pgsql-xtm-one` for XTM One. OpenAEV and XTM One now share one pgvector instance, on a new `pgsqlshareddata` volume. The old volumes are left untouched, so the new version starts with empty databases unless you move the data.

:::steps
1. Before you get the new files, dump both databases.

   ```bash
   docker compose start pgsql pgsql-xtm-one
   docker compose stop openaev xtm-one xtm-one-worker
   docker compose exec -T pgsql sh -c 'pg_dump -U "$POSTGRES_USER" -Fc openaev' > openaev.dump
   docker compose exec -T pgsql-xtm-one sh -c 'pg_dump -U "$POSTGRES_USER" -Fc xtm_one' > xtm_one.dump
   docker compose down
   ```

   :::verify
   `ls -lh openaev.dump xtm_one.dump` lists two files, none of them empty.
   :::

2. Get the new files, prepare the shared instance and restore both databases. The `xtm-one-db-init` service creates the XTM One role, its database and the pgvector extension.

   ```bash
   git pull
   docker compose run --rm xtm-one-db-init
   docker compose exec -T pgsql sh -c 'pg_restore -U "$POSTGRES_USER" -d openaev' < openaev.dump
   docker compose exec -T pgsql sh -c 'pg_restore -U "$POSTGRES_USER" -d xtm_one' < xtm_one.dump
   ```

3. Download the new images and start the stack, then continue at step 4 of [Upgrade the stack](#upgrade-the-stack).

   ```bash
   docker compose pull
   docker compose up -d
   ```

4. Once OpenAEV and XTM One show your data, remove the old volumes. Their names start with your `COMPOSE_PROJECT_NAME`.

   ```bash
   docker volume rm xtm_pgsqldata xtm_pgsqlxtmonedata
   ```
:::

:::warning[Started the new version before the dump?]
Your data is still in the old volumes. The `xtm_pgsqlshareddata` volume holds the databases the platforms created on that start, with everything written to them since: removing it deletes those writes. If you need them, dump both databases from `pgsql` first with the `pg_dump` commands above. Then run `docker compose down`, remove the `xtm_pgsqlshareddata` volume, check out the previous version of xtm-docker, and start this procedure again.
:::

## Stay on a version or an LTS line

xtm-docker starts every image with the `latest` tag, so each upgrade moves to the latest release. To stay on a version, for example an [LTS line](/deploy/upgrade/lifecycle/) of OpenCTI, set the image tags in `docker-compose.override.yml` with the commands of [Pin the image versions](/deploy/install/docker-compose/#pin-the-image-versions). With `OPENCTI_VERSION=7.260811.0-lts.1` in them, every OpenCTI image (platform, worker and default connectors) uses `7.260811.0-lts.1`. Give `OPENAEV_VERSION` and `XTM_ONE_VERSION` the releases you run, or `latest`.

:::verify
```bash
docker compose config --images | grep '^opencti/'
```
Every OpenCTI image ends in `:7.260811.0-lts.1`.
:::

An LTS release needs an LTS license key: see [Versioning and LTS](/deploy/upgrade/lifecycle/#get-an-lts-release). The lines the documentation describes today:

| OpenCTI line | Release documented | Supported until |
| --- | --- | --- |
| Latest | 7.261030.1 | Replaced by each new release |
| 7.260811-lts | 7.260811.0-lts.1 | September 2027 |
| 7.260309-lts | 7.260309.0-lts.8 | April 2027 |

To upgrade later, run the same commands with the release you upgrade to, such as `OPENCTI_VERSION=7.260811.0-lts.2`, then download the images and start the stack as in [Upgrade the stack](#upgrade-the-stack).

A changed `docker-compose.yml`, or a changed file of `infra/` or `modules/`, makes `git pull` refuse to merge. List your changes with `git diff`, and write each one in `docker-compose.override.yml`, as [Change the stack without editing docker-compose.yml](/deploy/install/docker-compose/#change-the-stack-without-editing-docker-compose-yml) describes: pin image tags with the commands of [Pin the image versions](/deploy/install/docker-compose/#pin-the-image-versions). Then put the files of xtm-docker back and pull:

```bash
git checkout -- .
git pull
```

## Other deployments

::::tabs
:::tab[Docker Swarm]
Set the new image tags in your stack file, then force each service to restart on its new image:

```bash
docker service update --force <service_name>
```
:::
:::tab[Manual installation]
Replace the application files with those of the new release, then start the platform: it migrates its databases as it starts. See [Install without containers](/deploy/install/manual/).

- OpenCTI, from the `opencti-platform/opencti-graphql` folder: `yarn install`, `yarn build`, then `yarn serv`
- OpenAEV: `java -jar openaev-api.jar`
:::
::::

## What's next

- [Breaking changes](/deploy/upgrade/breaking-changes/), the actions each version asks for.
- [Upgrade XTM One](/deploy/upgrade/xtm-one/), to upgrade XTM One alone or on its own host.
- [Versioning and LTS](/deploy/upgrade/lifecycle/), to choose between the latest release and an LTS line.
- [Compatibility matrix](/deploy/upgrade/compatibility/), the versions tested together.
- [Back up and restore](/deploy/operate/backup/), for production backups.
