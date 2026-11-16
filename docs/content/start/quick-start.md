---
title: Try OpenCTI with Docker
description: Run OpenCTI on your machine with the official Docker deployment, sign in, and browse MITRE ATT&CK and reference data in about fifteen minutes.
doc_type: tutorial
products:
  - opencti
  - xtm-one
personas: [evaluator, platform-engineer]
owner: docs-platform
last_reviewed: 2026-11-01
sidebar_position: 1
---

In about fifteen minutes, most of it spent downloading images, you run OpenCTI on your own machine and sign in as its administrator. You then browse real data: the MITRE ATT&CK techniques and threat groups, and the countries, regions and sectors that OpenCTI uses as reference. You use the official Docker deployment of the XTM suite, [FiligranHQ/xtm-docker](https://github.com/FiligranHQ/xtm-docker), which starts the whole stack; this tutorial looks at OpenCTI and its default connectors. This setup is meant for trying the platform, not for production: to deploy for real, start from [Deploy the platforms](/deploy/).

XTM One, the AI layer at the center of the suite, OpenAEV, OpenCRQ and XTM Composer start with OpenCTI: every module of the stack runs. To try each of them, follow [Run the XTM suite locally](/deploy/quick-start/) instead.

::diagram{id=start-quick-start}

:::note
The deployment starts the latest release of each product, whatever version of this documentation you read. Long-term support (LTS) versions need a license and their own image tags: see [Release lifecycle](/deploy/upgrade/lifecycle/).
:::

## What you need

- Docker Engine 20.10 or later with Docker Compose 2.20.2 or later (the stack is split into included files): `docker compose version` answers.
- On a Mac with an Apple chip, Docker Desktop with **Use Rosetta for x86_64/amd64 emulation** on. XTM One is published for Intel processors only and runs through emulation.
- 16 GB of RAM and 50 GB of free disk.
- Git, and Python 3 to generate the secrets.
- A Docker Hub account with access to `filigran/opengrc-platform`, the image of OpenCRQ, signed in with `docker login`. OpenCRQ is not released yet and its image is not public: without that access, the stack does not start.
- These ports free on the machine: 8080 (OpenCTI), 8081 (OpenAEV), 8082 (OpenCRQ), 8090 (XTM One), and 9200, 9000, 5672, 15672 and 6379 for the shared services.

## Step 1: Get the deployment

```bash
git clone https://github.com/FiligranHQ/xtm-docker.git
cd xtm-docker
```

The folder holds `docker-compose.yml`, which describes every service of the suite, and `.env.sample`, the settings to copy.

## Step 2: Create the configuration

The script below copies `.env.sample` to `.env` and gives every password, key and token a random value. Identifiers and admin tokens get a UUID, the OpenCTI encryption key a 32-byte base64 key. OpenCTI does not start with the sample values. Mail goes to the local machine, so no mail server is needed.

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

Keep `.env` to yourself: it holds the administrator password, and anyone who can read it can take over the platform.

## Step 3: Fit the stack to your machine

Give Elasticsearch 2 GB of memory instead of 4, which suits a 16 GB machine:

```bash
sed -i.bak 's/^ELASTIC_MEMORY_SIZE=.*/ELASTIC_MEMORY_SIZE=2G/' .env
```

The deployment starts OpenCTI with its worker and ten connectors: file imports and exports, document analysis, and the two that bring data in, **OpenCTI Datasets** and the MITRE ATT&CK connector. XTM One, OpenAEV with its collectors and injectors, XTM Composer and the shared services start with it.

:::verify
```bash
docker compose config --services | sort
```
The list holds `opencti`, `worker`, `connector-mitre`, `connector-opencti` and eight other `connector-` services, `xtm-one` with its worker, `openaev`, `opencrq` with its two init jobs, `xtm-composer`, and the shared services: 32 services in all.
:::

## Step 4: Start the stack

```bash
docker compose pull
docker compose up -d
```

Download first even if you have used xtm-docker before: Docker Compose would otherwise start an older image kept under the same `latest` tag. The first start creates the databases and indices: allow a few minutes.

:::verify
```bash
docker compose ps --format '{{.Service}} {{.Health}}' | grep -E '^(opencti|openaev|xtm-one|opencrq) '
```
The four lines end with `healthy`. OpenCTI also answers its health check from the host:

```bash
set -a; . ./.env; set +a
curl -fsS "http://localhost:$OPENCTI_PORT/health?health_access_key=$OPENCTI_HEALTHCHECK_ACCESS_KEY" >/dev/null && echo "OpenCTI OK"
```
It prints `OpenCTI OK`.
:::

If a container restarts in a loop, read its logs with `docker compose logs opencti`. Elasticsearch on Linux may need a larger memory map: `sudo sysctl -w vm.max_map_count=262144`.

## Step 5: Sign in

Read the administrator account that the script kept in `.env`:

```bash
grep -E '^OPENCTI_ADMIN_(EMAIL|PASSWORD)=' .env
```

Open `http://localhost:8080` in your browser and sign in with this email and password. The home dashboard opens.

The administrator is created from `.env` and reset to its values at every start: a password you change in the interface lasts until the next restart. See [Sign in the first time and keep an emergency account](/deploy/configure/first-access/).

## Step 6: Watch the data arrive

As soon as OpenCTI is up, the two data connectors send their bundles, and the three workers write them into the knowledge base. MITRE ATT&CK is the larger one: allow about ten minutes for it to land.

1. Open :menu[Integrations] and stay on the **Deployed** tab. **OpenCTI Datasets** and the MITRE ATT&CK connector show their queued messages. The number goes down as the workers process them.

   ::screenshot{id=opencti-ingest-connectors-deployed alt="The Deployed tab of Integrations: the platform figures at the top, with queued bundles and connected workers, then each connector with its queued messages, its last activity and its status."}

2. Open :menu[Techniques > Attack patterns]. The ATT&CK techniques fill the list, such as **Phishing** and its sub-techniques.
3. Open :menu[Threats > Intrusion sets]. The groups that ATT&CK tracks are there, each with the techniques it uses on its **Knowledge** tab, under **Attack patterns**.
4. Open :menu[Locations > Countries] and :menu[Entities > Sectors]. The reference data that reports and threats point to is in place.

:::example[Check ATT&CK coverage before a hotel-sector project]
The architect of Example CERT, which serves hotel chains, wants to see how OpenCTI models an attack technique before proposing the platform. On a 16 GB laptop, `.env` reads:

```bash
ELASTIC_MEMORY_SIZE=2G
```

Ten minutes after the start, the architect types `Phishing` in **Search the platform...** at the top and opens the attack pattern **Phishing** (T1566). Its **Overview** gives its sub-techniques and the courses of action that mitigate it, and its **Knowledge** tab lists the intrusion sets that use it, all imported from ATT&CK.
:::

## Stop and start over

| To | Run | Your data |
| --- | --- | --- |
| Stop the stack | `docker compose stop` | Kept |
| Remove the containers | `docker compose down` | Kept in the volumes |
| Start from scratch | `docker compose down -v` | Deleted, with every volume |

## What you learned

- OpenCTI is a platform, workers that write data, and connectors that bring it in: [How OpenCTI works](/opencti/concepts/architecture/).
- What you browsed is a knowledge graph of STIX objects and relationships: [Data model](/opencti/concepts/data-model/).
- Connectors are how data reaches OpenCTI, and the **Deployed** tab is where you follow them: [Import with connectors](/opencti/use/ingest/connectors/).

To go further, explore the knowledge you imported in [Explore knowledge](/opencti/use/explore/), try OpenAEV and XTM One with [Run the XTM suite locally](/deploy/quick-start/), or plan a real deployment in [Deploy the platforms](/deploy/).
