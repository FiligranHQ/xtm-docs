---
title: Run the XTM suite locally
description: Start the whole XTM stack, XTM One, OpenCTI, OpenAEV and XTM Composer,
  on one machine with Docker Compose.
doc_type: tutorial
products:
  - suite
personas: [platform-engineer, evaluator]
owner: docs-platform
last_reviewed: 2026-10-31
---

This tutorial starts the XTM suite on one machine with the official Docker deployment, [FiligranHQ/xtm-docker](https://github.com/FiligranHQ/xtm-docker), in about fifteen minutes. Two commands start the whole stack: XTM One, at the center of the suite, with OpenCTI, OpenAEV, OpenCRQ, XTM Composer, their default integrations and the shared services. No module is optional. It is meant to try the suite or to develop against it, not for production.

::diagram{id=deploy-quick-start}

:::tip[Let an AI agent do it]
Every step below has a command and a check, so a coding agent such as Claude Code can follow it on its own. See [Deploy with an AI agent](/deploy/quick-start/with-an-ai-agent/).
:::

## What you need

- Docker Engine 20.10 or later with Docker Compose 2.20.2 or later (the stack is split into included files): `docker compose version` answers.
- On a Mac with an Apple chip: Docker Desktop with **Use Rosetta for x86_64/amd64 emulation** on. XTM One is published for Intel processors only and runs through emulation.
- 16 GB of RAM for the whole suite, 32 GB recommended, and 50 GB of free disk.
- Python 3, to generate the secrets.
- A Docker Hub account with access to `filigran/opengrc-platform`, the image of OpenCRQ, signed in with `docker login`. OpenCRQ is not released yet and its image is not public: without that access, the stack does not start.
- Free ports on the machine: 8080 (OpenCTI), 8081 (OpenAEV), 8082 (OpenCRQ), 8090 (XTM One), and 9200, 9000, 5672, 15672 and 6379 for the shared services.

## What the stack runs

Every module starts, with the shared services they use, and each platform answers on its own address.

::stack-builder

## Start the suite

:::steps
1. Get the deployment.

   ```bash
   git clone https://github.com/FiligranHQ/xtm-docker.git
   cd xtm-docker
   ```

2. Create the configuration with fresh secrets. The script copies `.env.sample` to `.env` and gives every password, key and token a random value: a UUID for the IDs and admin tokens, a 32-byte base64 key for the OpenCTI encryption key. Mail goes to the local machine.

   ```bash
   cp .env.sample .env
   python3 - <<'PY'
   import base64, os, re, secrets, uuid
   out = []
   for line in open('.env'):
       m = re.match(r'^([A-Z0-9_]+)=', line)
       k = m.group(1) if m else ''
       if k == 'OPENCTI_ENCRYPTION_KEY':
           v = base64.b64encode(os.urandom(32)).decode()
       elif k.endswith('_ID') or k.endswith('_ADMIN_TOKEN') or 'HEALTHCHECK' in k:
           v = str(uuid.uuid4())
       elif re.search(r'PASSWORD|_PASS$|SECRET|SALT|REGISTRATION_TOKEN|ENCRYPTION_KEY', k):
           v = secrets.token_hex(24)
       else:
           v = {'SMTP_HOST': 'localhost', 'SMTP_PORT': '25', 'SMTP_AUTH': 'false', 'SMTP_SSL_ENABLE': 'false'}.get(k)
       out.append(f'{k}={v}\n' if v is not None else line)
   open('.env', 'w').writelines(out)
   PY
   ```

   :::verify
   ```bash
   docker compose config --quiet && echo "configuration OK"
   ```
   It prints `configuration OK`.
   :::

3. Fit the configuration to your machine. On a machine with 16 GB of RAM, give Elasticsearch 2 GB instead of 4.

   ```bash
   sed -i.bak 's/^ELASTIC_MEMORY_SIZE=.*/ELASTIC_MEMORY_SIZE=2G/' .env
   ```

   Then look for ports another program already uses.

   ```bash
   for p in 8080 8081 8090 9200 9000 5672 15672 6379; do lsof -nP -iTCP:$p -sTCP:LISTEN >/dev/null && echo "port $p is taken"; done
   ```

   For a port it lists, choose a free one: `OPENCTI_PORT` and `XTM_ONE_PORT` are set in `.env`. The port of OpenAEV is set in `docker-compose.yml` (`"8081:8080"`), and so are the ports of the shared services: to move one, replace its mapping in `docker-compose.override.yml`, as [Network and ports](/deploy/plan/network/#ports-published-on-the-host) shows, and for OpenAEV change `OPENAEV_PORT` in `.env` too.

   :::verify
   The loop prints nothing, or only ports you have moved.
   :::

4. Download the current images, then start the suite.

   ```bash
   docker compose pull
   docker compose up -d
   ```

   Download first even if you have run XTM before: Docker Compose would otherwise start an older image kept under the same `latest` tag. The first start creates the databases: allow a few minutes.

   :::verify
   ```bash
   docker compose config --services | wc -l
   ```
   It prints `32`: every service of the stack.
   :::

5. Wait until the platforms are healthy.

   ```bash
   docker compose ps --format '{{.Service}} {{.Health}}' | grep -E '^(opencti|openaev|xtm-one|opencrq) '
   ```

   :::verify
   `opencti`, `openaev`, `xtm-one` and `opencrq` say `healthy`. From the host, OpenCTI, OpenAEV and XTM One answer their health check:

   ```bash
   set -a; . ./.env; set +a
   curl -fsS "http://localhost:$OPENCTI_PORT/health?health_access_key=$OPENCTI_HEALTHCHECK_ACCESS_KEY" >/dev/null && echo "OpenCTI OK"
   curl -fsS "http://localhost:$OPENAEV_PORT/api/health?health_access_key=$OPENAEV_HEALTHCHECK_KEY" >/dev/null && echo "OpenAEV OK"
   curl -fsS "http://localhost:$XTM_ONE_PORT/api/health" >/dev/null && echo "XTM One OK"
   ```
   :::

6. Sign in at the addresses listed above, with the admin email and password of each platform in `.env`.

   ```bash
   grep -E '^(OPENCTI|OPENAEV|XTM_ONE)_ADMIN_(EMAIL|PASSWORD)=' .env
   ```
:::

## Stop and start over

| To | Run | Your data |
| --- | --- | --- |
| Stop the suite | `docker compose stop` | Kept |
| Remove the containers | `docker compose down` | Kept in the volumes |
| Start from scratch | `docker compose down -v` | Deleted, with every volume |

## What's next

- [Deploy with an AI agent](/deploy/quick-start/with-an-ai-agent/), to repeat this without typing.
- [The XTM suite](/start/), what each module does and how they work together.
