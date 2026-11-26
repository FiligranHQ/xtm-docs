---
title: Upgrade XTM One
description: "Move XTM One to a new version in the XTM stack: serve it over HTTP/2 for 1.261112.0, follow its migrations, rename retired variables, and roll back from a backup."
doc_type: how-to
products:
  - xtm-one
personas: [platform-engineer]
owner: docs-xtm-one
last_reviewed: 2026-11-12
migrated_from:
  - xtm-one:/deployment/upgrade/
---

This page moves XTM One to a new version. XTM One migrates its database on its own when the new platform starts, so an upgrade is a backup, new images and a check; going back means restoring that backup. In an XTM stack, [Upgrade the XTM stack](/deploy/upgrade/) upgrades XTM One with the other platforms: read this page for what XTM One does during it, or to upgrade XTM One alone.

::diagram{id=xtm-one-deploy-upgrade-steps}

## What happens during an upgrade

- **The platform migrates.** The first `xtm-one` process to start on the new version applies the migrations of the database, then creates or updates the built-in agents and content. Other processes and replicas wait for it, and run the steps that are left. `/api/health/ready` answers 503 until the start is complete.
- **The worker waits.** `xtm-one-worker` waits for the migrations before it takes jobs. xtm-docker starts it once `xtm-one` is healthy.
- **No step back.** A migration can change the schema in a way the previous version does not read. To go back, restore the database and the bucket from before the upgrade: see [Roll back](#roll-back).
- **Same version on both containers.** Run `xtmone/platform` and `xtmone/worker` at the same tag.

A version reads `1.YYMMDD.patch`, such as `1.261112.0`: the date of the release, then its patch number. Docker Hub lists the tags of [xtmone/platform](https://hub.docker.com/r/xtmone/platform/tags). Check which versions of OpenCTI and OpenAEV go with it in [Compatibility between modules](/deploy/upgrade/compatibility/).

## Before you start

- Read the [breaking changes of XTM One](/deploy/upgrade/breaking-changes/#xtm-one) between your version and the new one, and the [renamed variables](#renamed-variables).
- From a version before 1.261112.0, behind a reverse proxy, [prepare the proxy](#prepare-the-reverse-proxy-for-1-261112-0) first: the chat needs HTTP/2.
- Pin the version you run today, if you have not, so that a `docker compose pull` changes nothing you did not choose: see [Pin the image versions](/deploy/install/docker-compose/#pin-the-image-versions).
- Plan a few minutes without XTM One: chats and running assignments stop while `xtm-one` restarts.

## Prepare the reverse proxy for 1.261112.0

From 1.261112.0, the chat of XTM One streams over HTTP, as server-sent events, instead of a WebSocket: the **Chat** page, the rails of the deck and video studios, voice mode and the Office add-in. Every open tab holds some of these streams, and a browser opens at most 6 HTTP/1.1 connections per host. Behind a proxy that speaks HTTP/1.1 to browsers, a few open tabs use them all, and XTM One stops loading. Nothing migrates: change the proxy before you upgrade. [XTM One 1.261112.0](/deploy/upgrade/breaking-changes/#xtm-one-1-261112-0) lists the whole change.

:::steps
1. In the server block of XTM One, turn HTTP/2 on, and keep the two settings the streams need: no response buffering, and a read timeout above 15 seconds, the interval of their keep-alive. With Nginx 1.25.1 or later:

   ```nginx
   server {
       listen 443 ssl;
       http2 on;
       server_name xtm-one.example.com;
       # certificates and client_max_body_size as before

       location / {
           # proxy_pass and the headers as before, with:
           proxy_buffering    off;
           proxy_read_timeout 300s;
       }
   }
   ```

   Before Nginx 1.25.1, write `listen 443 ssl http2;` instead of the `listen` and `http2` lines. An answer can take minutes, hence the 300 seconds. Nginx also follows the `X-Accel-Buffering: no` header the streams send; another proxy needs response buffering turned off for `/api/`. Keep forwarding the `Upgrade` and `Connection` headers: XTM One Desktop, the runners and the relays still connect over a WebSocket. [Publish the platforms over HTTPS](/deploy/install/reverse-proxy/) gives the whole block.

2. Check the configuration and load it.

   ```bash
   sudo nginx -t
   sudo systemctl reload nginx
   ```

   :::verify
   ```bash
   curl -sS --http2 -o /dev/null -w '%{http_version}\n' https://xtm-one.example.com/api/health
   ```
   It prints `2`. `1.1` means the proxy does not serve HTTP/2 yet.
   :::
:::

A script that opens the chat WebSocket, `/api/v1/chat/conversations/{id}/ws`, stops working with the upgrade: move it to the [XTM One chat API](/develop/apis/chat/). A script that sends messages with `POST /api/v1/chat/conversations/{id}/messages` keeps working.

## Upgrade XTM One in the XTM stack

:::steps
1. Back up the database and the files of XTM One, and keep `.env` with them: the backup is useless without the same `XTM_ONE_SECRET_KEY`.

   ```bash
   mkdir -p backup
   docker compose exec -T pgsql sh -c 'pg_dump -U "$POSTGRES_USER" -Fc xtm_one' > backup/xtm_one.dump
   cp .env backup/
   ```

   The bucket of XTM One is in the `s3data` volume, shared with the other platforms: back it up as [Back up and restore](/deploy/operate/backup/) describes.

   :::verify
   `ls -lh backup/xtm_one.dump` shows a file that is not empty.
   :::

2. Set the new version. Run the pinning commands of [Pin the image versions](/deploy/install/docker-compose/#pin-the-image-versions) with the new `XTM_ONE_VERSION`, here `1.261112.0`, and the versions of OpenCTI and OpenAEV you keep.

   :::verify
   ```bash
   docker compose config --images | grep '^xtmone/'
   ```
   It prints `xtmone/platform:1.261112.0` and `xtmone/worker:1.261112.0`.
   :::

3. Download the images and restart XTM One.

   ```bash
   docker compose pull xtm-one xtm-one-worker
   docker compose up -d xtm-one xtm-one-worker
   ```

4. Follow the start until it is complete.

   ```bash
   docker compose logs -f xtm-one
   ```

   `Database migrations applied`, then the creation of the built-in content, then the requests. A large database takes longer to migrate: after 300 seconds without the end of its start, the platform logs which step runs and which database sessions block it, then again every 300 seconds.

   :::verify
   ```bash
   docker compose exec xtm-one curl -sS http://localhost:4000/api/health/ready
   docker compose images xtm-one xtm-one-worker
   ```
   The answer ends with `"status":"ok"`, and both images show the new tag.
   :::

5. Check the links and the configuration. In XTM One, **My Profile** > **MCP Endpoint** shows each registered platform as `Connected`; OpenCTI, OpenAEV and OpenCRQ register again within 5 minutes. Then run the **Security audit** in :menu[Administration > Settings > Access and security > Security audit]: a release can add checks on settings you have not set.
:::

If the start fails in a migration, the log names it. Do not run migrations by hand and do not start the previous version on the migrated database: [roll back](#roll-back), and keep the log for Filigran support.

On a very large database, two settings bound the migrations, both in seconds: `MIGRATION_LOCK_TIMEOUT_SECONDS` (`300`), the longest a statement waits for a lock, and `MIGRATION_STATEMENT_TIMEOUT_SECONDS` (`1800`), the longest a statement runs. Past either, the migration is tried again, then the start fails with a message that names the setting. Raise the one it names on `xtm-one`, in `docker-compose.override.yml`.

## Renamed variables

A release that renames a variable keeps reading the old name for a while, and logs a warning that names the new one. Rename them in your override file and your secrets before the date:

| Old name | New name | Read until |
| --- | --- | --- |
| `OPENGRC_API_URL` | `OPENCRQ_API_URL` | 31 December 2026. When both are set, `OPENCRQ_API_URL` wins |

## Leave the published registration token

An XTM One whose `PLATFORM_REGISTRATION_TOKEN` is `xtm-default-registration-token` runs with a published value, and so does one installed with the product's Compose file and an empty `PLATFORM_REGISTRATION_TOKEN`, which falls back to it. That value keeps the registered platforms connected, but registers no new one and changes no registered address, and XTM One sends the platforms no license certificate: a platform whose Enterprise Edition comes from the license of XTM One cannot prove it. In xtm-docker, an empty token refuses every registration instead, and the secrets script of [Install with Docker Compose](/deploy/install/docker-compose/) sets one of your own.

:::steps
1. Generate a token: `openssl rand -hex 32`.
2. Set it as `PLATFORM_REGISTRATION_TOKEN` on XTM One and as the registration token of every OpenCTI, OpenAEV and OpenCRQ connected to it: `XTM__XTM_ONE_TOKEN`, `OPENAEV_XTM_ONE_TOKEN`, `XTM_ONE__TOKEN`. A platform that keeps the old value is refused.
3. Restart XTM One, then the platforms.
4. Run the **Security audit**: the finding on the registration token passes. Then review the registered platforms in :menu[Administration > Settings > Integrations > Catalog]: a new token does not end a registration made with the published value, and the audit names any registration it does not expect.
:::

## Roll back

:::steps
1. Stop XTM One.

   ```bash
   docker compose stop xtm-one xtm-one-worker
   ```

2. Restore the `xtm_one` database from the backup of step 1, and the bucket from the same moment, as [Back up and restore](/deploy/operate/backup/) describes. A database and a bucket from two different moments leave files without records, or records without files.
3. Pin the previous version again with the pinning commands, then start XTM One.

   ```bash
   docker compose up -d xtm-one xtm-one-worker
   ```
:::

:::example[Upgrade XTM One at Grand Meridian Hotels]
On Sunday night, before the Monday 08:00 run of the "Weekly Velvet Harbor brief", Marc Dupuis moves the XTM One of Grand Meridian Hotels from `1.261110.2` to `1.261112.0`. The Nginx in front of it speaks HTTP/1.1 to browsers, so he first adds one line under `listen 443 ssl;` in its server block, `xtm-one.grand-meridian.example`, which already turns buffering off and waits 300 seconds:

```nginx
http2 on;
```

After `sudo nginx -t` and `sudo systemctl reload nginx`, the `curl` check prints `2`. He dumps `xtm_one` to `backup/xtm_one.dump`, copies `.env`, and pins the versions:

```bash
OPENCTI_VERSION=7.261030.1
OPENAEV_VERSION=3.261029.0
XTM_ONE_VERSION=1.261112.0
```

After the pinning commands, `docker compose pull xtm-one xtm-one-worker` and `docker compose up -d xtm-one xtm-one-worker`, the log shows `Database migrations applied`, and the readiness answer is `"status":"ok"` two minutes later. His override file still sets `OPENGRC_API_URL=http://opencrq:3002`: the log warns about it, and he renames it `OPENCRQ_API_URL` before the end of December.
:::

## What's next

- [Upgrade the XTM stack](/deploy/upgrade/), to upgrade the other platforms with it.
- [Compatibility between modules](/deploy/upgrade/compatibility/), for the versions that work together.
- [Monitor and troubleshoot XTM One](/deploy/operate/xtm-one/), when the new version does not start.
