---
title: Back up and restore
description: Back up every store of the XTM stack together, with an Elasticsearch snapshot, PostgreSQL dumps and copies of the other volumes, and restore them on an empty deployment.
doc_type: how-to
products:
  - suite
personas: [platform-engineer]
owner: docs-platform
last_reviewed: 2026-10-31
---

This page backs up a Docker Compose deployment of the XTM stack, [FiligranHQ/xtm-docker](https://github.com/FiligranHQ/xtm-docker), and restores it on an empty one. You get one consistent backup of all its data: the knowledge in Elasticsearch, the databases in PostgreSQL, the files in S3 storage, the other volumes and the secrets. The platforms are stopped during the backup, so plan a maintenance window.

::diagram{id=deploy-operate-backup}

## What to back up

The data of the stack lives in six Docker volumes and one file. Their names start with `COMPOSE_PROJECT_NAME` from `.env` (`xtm` in `.env.sample`), for example `xtm_esdata`.

| What | Where | Holds | Backed up with |
| --- | --- | --- | --- |
| Elasticsearch | Volume `esdata` | The knowledge of OpenCTI and OpenAEV and its indices | A snapshot |
| PostgreSQL | Volume `pgsqlshareddata` | The `openaev`, `opencrq` and `xtm_one` databases | `pg_dump` |
| S3 storage (RustFS) | Volume `s3data` | The files of OpenCTI, OpenAEV, OpenCRQ and XTM One | A copy of the volume |
| Redis | Volume `redisdata` | Cache, sessions and live events | A copy of the volume |
| RabbitMQ | Volume `amqpdata` | The queues and the messages waiting in them | A copy of the volume |
| XTM Composer key | Volume `rsakeys` | The RSA key XTM Composer encrypts the integrations' configurations with | A copy of the volume |
| Secrets | File `.env` | Passwords, tokens and the encryption keys of the platforms | A copy of the file |

:::warning[Back up the secrets with the data]
The platforms encrypt part of their data with keys from `.env`, such as `OPENCTI_ENCRYPTION_KEY`, `OPENAEV_ADMIN_ENCRYPTION_KEY` and `XTM_ONE_SECRET_KEY`, and XTM Composer with the key in `rsakeys`. Data restored without the same keys cannot be read. Keep the backup as safe as the secrets it holds.
:::

### Why everything is backed up together

A platform keeps one piece of information in several stores. OpenCTI writes an imported file to S3 storage and indexes it in Elasticsearch; OpenAEV and XTM One keep their records in PostgreSQL and their files in S3 storage. Stores backed up at different times disagree after a restore: a record points to a file that is missing, or the other way round. So the procedure below stops everything that writes, the platforms, their workers and the integrations, then backs up every store at the same point in time.

Elasticsearch does not support restoring from a copy of its data folder, even a stopped one: back it up with a snapshot. The other volumes are copied while their service is stopped.

## Before you start

- The stack runs and is healthy: see [Check that the platforms are healthy](/deploy/operate/health-probes/).
- Free disk space for the backup: about the size of the six volumes. `docker system df -v` gives their size.
- A snapshot repository for Elasticsearch, prepared once as below.

A stack with XTM One alone has no Elasticsearch and no RabbitMQ: skip the steps about Elasticsearch.

### Prepare the snapshot repository

Elasticsearch writes snapshots to a folder declared in its `path.repo` setting. These steps make a folder `es-snapshots`, in the `xtm-docker` folder, that repository.

:::steps
1. Create the folder. Elasticsearch runs as user `1000` and group `0` in its container, and needs to write to it.

   ```bash
   mkdir -p es-snapshots
   chmod g+rwx es-snapshots
   sudo chgrp 0 es-snapshots
   ```

2. Declare it in `docker-compose.override.yml`, as [Change the stack without editing docker-compose.yml](/deploy/install/docker-compose/#change-the-stack-without-editing-docker-compose-yml) describes. If the file exists, add the `elasticsearch` part to it.

   ```yaml
   services:
     elasticsearch:
       environment:
         - path.repo=/usr/share/elasticsearch/snapshots
       volumes:
         - ./es-snapshots:/usr/share/elasticsearch/snapshots
   ```

3. Recreate Elasticsearch, then register the repository under the name `xtm-backup`.

   ```bash
   docker compose up -d --wait elasticsearch
   docker compose exec elasticsearch curl -s -X PUT http://localhost:9200/_snapshot/xtm-backup \
     -H 'Content-Type: application/json' \
     -d '{"type":"fs","settings":{"location":"/usr/share/elasticsearch/snapshots"}}'
   ```

   :::verify
   It answers `{"acknowledged":true}`.
   :::
:::

## Back up the stack

Run every command from the `xtm-docker` folder.

:::steps
1. Create the backup folder and record the versions you run: restore on the same ones.

   ```bash
   mkdir -p backup
   git rev-parse HEAD > backup/xtm-docker-commit.txt
   docker compose images > backup/images.txt
   ```

2. Stop everything that writes: every service except the stores.

   ```bash
   docker compose stop $(docker compose config --services | grep -vxE 'elasticsearch|pgsql|redis|minio|rabbitmq|rsa-key-generator')
   ```

   :::verify
   ```bash
   docker compose ps --format '{{.Service}}'
   ```
   It lists only stores: `elasticsearch`, `pgsql`, `redis`, `minio`, `rabbitmq`, and `rsa-key-generator` if XTM Composer is deployed.
   :::

3. Take an Elasticsearch snapshot of all the indices.

   ```bash
   docker compose exec elasticsearch curl -s -X PUT \
     "http://localhost:9200/_snapshot/xtm-backup/snapshot-$(date +%Y%m%d-%H%M%S)?wait_for_completion=true"
   ```

   :::verify
   The answer contains `"state":"SUCCESS"`.
   :::

4. Dump the three PostgreSQL databases.

   ```bash
   docker compose exec -T pgsql sh -c 'pg_dump -U "$POSTGRES_USER" -Fc openaev' > backup/openaev.dump
   docker compose exec -T pgsql sh -c 'pg_dump -U "$POSTGRES_USER" -Fc opencrq' > backup/opencrq.dump
   docker compose exec -T pgsql sh -c 'pg_dump -U "$POSTGRES_USER" -Fc xtm_one' > backup/xtm_one.dump
   ```

   :::verify
   ```bash
   ls -l backup/*.dump
   ```
   The three files exist and are not empty.
   :::

5. Have Redis write its data to disk, then stop the stores.

   ```bash
   docker compose exec redis redis-cli SAVE
   docker compose stop
   ```

   Redis answers `OK`.

6. Copy the other volumes, each into an archive.

   ```bash
   set -a; . ./.env; set +a
   for v in s3data redisdata amqpdata rsakeys; do
     docker volume inspect "${COMPOSE_PROJECT_NAME}_$v" >/dev/null 2>&1 || continue
     docker run --rm -v "${COMPOSE_PROJECT_NAME}_$v:/data:ro" -v "$PWD/backup:/backup" \
       alpine:3.20 tar -czf "/backup/$v.tar.gz" -C /data .
   done
   ```

   The loop skips a volume your modules do not create, such as `rsakeys` without XTM Composer.

7. Copy the secrets, the override file and the snapshot repository.

   ```bash
   cp .env docker-compose.override.yml backup/
   tar -czf backup/es-snapshots.tar.gz es-snapshots
   ```

8. Start the stack again.

   ```bash
   docker compose up -d
   ```

   :::verify
   ```bash
   docker compose ps --format '{{.Service}} {{.Health}}' | grep -E '^(opencti|openaev|xtm-one|opencrq) '
   ```
   Each platform you run says `healthy` after a few minutes.
   :::
:::

Move the `backup` folder to storage outside the machine.

Elasticsearch snapshots are incremental: each one stores only what changed since the previous ones in the same repository, so `es-snapshots` grows slowly. Delete a snapshot you no longer need with `docker compose exec elasticsearch curl -s -X DELETE http://localhost:9200/_snapshot/xtm-backup/<name>`, never by deleting files from the folder.

## Restore the stack

The restore puts a backup back on an empty deployment: a new machine, or the same one after `docker compose down -v`.

:::warning[On an empty deployment only]
Restore before the platforms ever start on the new deployment. A platform that starts on empty stores creates its own data, and the restore then fails or mixes the two.
:::

:::steps
1. Get the same version of xtm-docker, and put the backup in it.

   ```bash
   git clone https://github.com/FiligranHQ/xtm-docker.git
   cd xtm-docker
   git checkout "$(cat /path/to/backup/xtm-docker-commit.txt)"
   cp -r /path/to/backup ./backup
   cp backup/.env backup/docker-compose.override.yml .
   tar -xzf backup/es-snapshots.tar.gz
   chmod -R g+rwX es-snapshots && sudo chgrp -R 0 es-snapshots
   ```

   Replace `/path/to/backup` with the folder of your backup.

2. Create the containers and the volumes, without starting anything.

   ```bash
   docker compose create
   ```

3. Put the copied volumes back.

   ```bash
   set -a; . ./.env; set +a
   for v in s3data redisdata amqpdata rsakeys; do
     [ -f "backup/$v.tar.gz" ] || continue
     docker run --rm -v "${COMPOSE_PROJECT_NAME}_$v:/data" -v "$PWD/backup:/backup:ro" \
       alpine:3.20 tar -xzf "/backup/$v.tar.gz" -C /data
   done
   ```

4. Restore PostgreSQL. `xtm-one-db-init` first creates the XTM One role, its database and the pgvector extension, and `opencrq-db-init` the OpenCRQ database, then each database is restored.

   ```bash
   docker compose up -d --wait pgsql
   docker compose run --rm xtm-one-db-init
   docker compose run --rm opencrq-db-init
   docker compose exec -T pgsql sh -c 'pg_restore -U "$POSTGRES_USER" -d openaev' < backup/openaev.dump
   docker compose exec -T pgsql sh -c 'pg_restore -U "$POSTGRES_USER" -d opencrq' < backup/opencrq.dump
   docker compose exec -T pgsql sh -c 'pg_restore -U "$POSTGRES_USER" -d xtm_one' < backup/xtm_one.dump
   ```

5. Start Elasticsearch, register the repository and list its snapshots.

   ```bash
   docker compose up -d --wait elasticsearch
   docker compose exec elasticsearch curl -s -X PUT http://localhost:9200/_snapshot/xtm-backup \
     -H 'Content-Type: application/json' \
     -d '{"type":"fs","settings":{"location":"/usr/share/elasticsearch/snapshots"}}'
   docker compose exec elasticsearch curl -s "http://localhost:9200/_cat/snapshots/xtm-backup?v"
   ```

6. Restore the snapshot you choose, with its indices, index templates and lifecycle policies. Replace `<snapshot>` with its name from the list.

   ```bash
   docker compose exec elasticsearch curl -s -X POST \
     "http://localhost:9200/_snapshot/xtm-backup/<snapshot>/_restore?wait_for_completion=true" \
     -H 'Content-Type: application/json' \
     -d '{"indices":"*","include_global_state":true}'
   ```

   :::verify
   The answer gives the restored indices with `"failed":0` in `shards`.
   :::

7. Start the stack.

   ```bash
   docker compose up -d
   ```

   :::verify
   Each platform says `healthy` in `docker compose ps`, and shows the data you had when you took the backup.
   :::
:::

Restore on the image versions of `backup/images.txt`. If you start newer images, the platforms migrate the restored data as they would during an upgrade: see [Upgrade](/deploy/upgrade/).

## What's next

- [Upgrade](/deploy/upgrade/): take a backup before every upgrade.
- [Manage Elasticsearch indices](/deploy/operate/rollover/), how OpenCTI organizes its Elasticsearch indices.
- [Harden the deployment](/deploy/operate/harden/), to protect the secrets and the stores.
