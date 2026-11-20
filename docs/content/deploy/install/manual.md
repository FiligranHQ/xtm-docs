---
title: Install without containers
description: Install OpenCTI or OpenAEV from their release archives on a Linux host, connected to shared services you run yourself.
doc_type: how-to
products:
  - opencti
  - openaev
personas: [platform-engineer]
owner: docs-platform
last_reviewed: 2026-10-31
migrated_from:
  - opencti:/deployment/installation/
  - openaev:/deployment/installation/
---

This page installs OpenCTI or OpenAEV without Docker, from the release archive of the platform, on a Linux host. You run the shared services yourself and point the platform to them. It suits hosts where containers are not allowed; everywhere else, [Install with Docker Compose](/deploy/install/docker-compose/) is shorter and is the official deployment. XTM One and XTM Composer are not covered here: run XTM One with Docker Compose, and XTM Composer as [Install XTM Composer](/deploy/integrations/install-composer/) describes.

::diagram{id=deploy-install-manual}

## Before you start

- The shared services of the platform, running and reachable from the host. The versions below are those the official deployment runs; [Requirements](/deploy/plan/requirements/) gives the supported ones.

  | Service | OpenCTI | OpenAEV | Version in xtm-docker |
  | --- | --- | --- | --- |
  | Elasticsearch, or OpenSearch | Yes | Yes | Elasticsearch 8.19.18 |
  | RabbitMQ | Yes | Yes | 4.3 |
  | S3 object storage, such as RustFS or MinIO | Yes | Yes | RustFS 1.0.4 |
  | Redis | Yes | | 8.8.0 |
  | PostgreSQL | | Yes | 17 |

- For RabbitMQ, these two lines in `rabbitmq.conf`, as in the official deployment: large bundles and long jobs need them.

  ```ini
  max_message_size = 536870912
  consumer_timeout = 86400000
  ```

- For Elasticsearch, the search queue raised to 5000: `thread_pool.search.queue_size=5000`.
- The version to install, from the releases of [OpenCTI](https://github.com/OpenCTI-Platform/opencti/releases) or [OpenAEV](https://github.com/OpenAEV-Platform/openaev/releases).

## Install the platform

::::::tabs
:::::tab[OpenCTI]
OpenCTI is a Node.js application with a Python part, and a Python worker that writes the incoming data. The commands are for a Debian or Ubuntu host.

:::steps
1. Install the build tools and Python, and create a Python virtual environment for OpenCTI. Activate it in every terminal where you run the commands below.

   ```bash
   sudo apt-get update
   sudo apt-get install -y build-essential python3 python3-pip python3-dev python3-venv
   sudo mkdir -p /opt/opencti && sudo chown "$USER" /opt/opencti
   python3 -m venv /opt/opencti/venv
   . /opt/opencti/venv/bin/activate
   ```

2. Install Node.js 22.18.0 or later, from [nodejs.org](https://nodejs.org/en/download) or with [nvm](https://github.com/nvm-sh/nvm): the packages of some distributions are older. Then install Corepack, which provides the Yarn version OpenCTI uses.

   ```bash
   npm install -g corepack
   node --version
   ```

   `node --version` prints `v22.18.0` or later.

3. Download and extract the release. On a distribution that uses musl, such as Alpine, download `opencti-release-${OPENCTI_VERSION}_musl.tar.gz` instead.

   :::only{opencti=">=7.261030"}
   ```bash
   OPENCTI_VERSION=7.261030.1
   ```
   :::

   :::only{opencti=">=7.260811 <7.261030"}
   ```bash
   OPENCTI_VERSION=7.260811.0-lts.1
   ```
   :::

   :::only{opencti="<7.260811"}
   ```bash
   OPENCTI_VERSION=7.260309.0-lts.8
   ```
   :::

   ```bash
   cd /opt/opencti
   wget "https://github.com/OpenCTI-Platform/opencti/releases/download/${OPENCTI_VERSION}/opencti-release-${OPENCTI_VERSION}.tar.gz"
   tar xzf "opencti-release-${OPENCTI_VERSION}.tar.gz"
   ```

   :::only{opencti=">=7.261030"}
   The archive extracts to `opencti/`, with the platform in `opencti/opencti-platform/opencti-graphql` and the worker in `opencti/worker`. The platform is already built: `build/` and the web interface in `public/` are in the archive.
   :::

   :::only{opencti="<7.261030"}
   The archive extracts to `opencti/`, with the platform directly in `opencti/` (its configuration in `opencti/config`) and the worker in `opencti/worker`. The platform is already built: `build/` is in the archive.
   :::

4. Write the configuration. OpenCTI reads `config/production.json` over `config/default.json`. Set the addresses and credentials of your shared services first, then run the script: it writes `config/production.json` with only the keys below, so that every other key keeps the default of the release you run, also after an upgrade. It generates the admin token, the health check key and the encryption key, and points `app.python_execution_venv` to the packages of the virtual environment, where OpenCTI loads its Python modules from. OpenCTI does not start without `app.encryption_key`, 32 bytes or more in base64.

   :::only{opencti=">=7.261030"}
   ```bash
   cd /opt/opencti/opencti/opencti-platform/opencti-graphql
   ```
   :::

   :::only{opencti="<7.261030"}
   ```bash
   cd /opt/opencti/opencti
   ```
   :::

   ```bash
   export ADMIN_EMAIL=admin@example.com ADMIN_PASSWORD='a-strong-password'
   export ES_URL=http://localhost:9200 REDIS_HOST=localhost
   export RABBITMQ_HOST=localhost RABBITMQ_USER=opencti RABBITMQ_PASS='rabbitmq-password'
   export S3_HOST=localhost S3_ACCESS_KEY=opencti S3_SECRET_KEY='s3-secret'
   python3 - <<'PY'
   import base64, json, os, sysconfig, uuid
   e = os.environ
   c = {
       'app': {
           'base_url': 'http://localhost:4000/',
           'admin': {'email': e['ADMIN_EMAIL'], 'password': e['ADMIN_PASSWORD'], 'token': str(uuid.uuid4())},
           'health_access_key': str(uuid.uuid4()),
           'encryption_key': base64.b64encode(os.urandom(32)).decode(),
           'python_execution_venv': sysconfig.get_paths()['purelib'],
       },
       'elasticsearch': {'url': e['ES_URL']},
       'redis': {'hostname': e['REDIS_HOST']},
       'rabbitmq': {'hostname': e['RABBITMQ_HOST'], 'username': e['RABBITMQ_USER'], 'password': e['RABBITMQ_PASS']},
       'minio': {'endpoint': e['S3_HOST'], 'access_key': e['S3_ACCESS_KEY'], 'secret_key': e['S3_SECRET_KEY']},
   }
   json.dump(c, open('config/production.json', 'w'), indent=2)
   PY
   chmod 600 config/production.json
   ```

   Keep a copy of `config/production.json` in your secret store: it holds the encryption key. Set `app.base_url` to the public URL when you put a reverse proxy in front. Every other key is in the [OpenCTI configuration reference](/reference/opencti/configuration/).

5. Install the Python modules of the platform.

   ```bash
   pip3 install -r src/python/requirements.txt
   ```

6. Install, build and start the platform. It creates its database schema on the first start, then listens on port 4000.

   ```bash
   yarn install
   yarn build
   yarn serv
   ```

   :::verify
   In another terminal, from the folder of the platform (the one of step 4):
   ```bash
   . /opt/opencti/venv/bin/activate
   KEY=$(python3 -c "import json;print(json.load(open('config/production.json'))['app']['health_access_key'])")
   curl -fsS "http://localhost:4000/health?health_access_key=$KEY" >/dev/null && echo "OpenCTI OK"
   ```
   It prints `OpenCTI OK`. You can sign in at `http://localhost:4000` with `ADMIN_EMAIL` and `ADMIN_PASSWORD`.
   :::

7. Configure the worker, in another terminal. The archive has the worker code, not its Python requirements: install them from the release tag. The worker reads `config.yml` next to `worker.py`, with the URL of the platform and the admin token.

   ```bash
   . /opt/opencti/venv/bin/activate
   cd /opt/opencti/opencti/worker
   ```

   :::only{opencti=">=7.261030"}
   ```bash
   PLATFORM=../opencti-platform/opencti-graphql
   ```
   :::

   :::only{opencti="<7.261030"}
   ```bash
   PLATFORM=..
   ```
   :::

   ```bash
   OPENCTI_VERSION=$(python3 -c "import json;print(json.load(open('$PLATFORM/package.json'))['version'])")
   pip3 install -r "https://raw.githubusercontent.com/OpenCTI-Platform/opencti/${OPENCTI_VERSION}/opencti-worker/requirements.txt"
   TOKEN=$(python3 -c "import json;print(json.load(open('$PLATFORM/config/production.json'))['app']['admin']['token'])")
   cat > config.yml <<EOF
   opencti:
     url: 'http://localhost:4000'
     token: '${TOKEN}'
     json_logging: true

   worker:
     log_level: 'info'
   EOF
   ```

8. Start as many workers as you need: each one is a process. The official deployment runs 3.

   ```bash
   python3 worker.py &
   python3 worker.py &
   python3 worker.py &
   ```
:::

These keys matter most on a host. In `config/production.json`, a key such as `app:python_execution` is the field `python_execution` of the `app` object.

| Key or variable | Default | What it does |
| --- | --- | --- |
| `app:port` | `4000` | Port the platform listens on |
| `app:python_execution` | `native` | How OpenCTI runs its Python scripts, which check indicator patterns and generate STIX patterns. `native` runs them in a Python interpreter embedded in the platform process. Any other value starts a `python3` process for each call, the one found in the `PATH` of the platform: slower |
| `app:python_execution_venv` | `../../../.python-venv/lib/python3.12/site-packages` | Folder of Python packages the embedded interpreter loads, relative to the folder the platform runs from. The script of step 4 sets it to your virtual environment |
| `NODE_ENV`, or `ENV`, or `-e` | `production` | The environment. It selects `config/<environment>.json`, and any value other than `production` turns on development mode. Leave it unset |
| `CONF`, or `-c` | not set | A configuration file to load instead of `config/production.json`, such as a file in `/etc`. `config/default.json` still applies under it |

For example, to keep the configuration in `/etc/opencti` and listen on port 4100, set `"port": 4100` in the `app` object of the file, then start the platform with:

```bash
CONF=/etc/opencti/production.json yarn serv
```

The other keys of a host installation are in the [Technical customization](/reference/opencti/configuration/#technical-customization) section of the configuration reference.
:::::

:::::tab[OpenAEV]
OpenAEV is a Java application, distributed as one JAR file.

:::steps
1. Install a Java 21 runtime. On Ubuntu 24.04:

   ```bash
   sudo apt-get update
   sudo apt-get install -y openjdk-21-jre-headless
   java -version
   ```

   `java -version` prints version 21.

2. Download and extract the release. On a distribution that uses musl, such as Alpine, download `openaev-release-${OPENAEV_VERSION}_musl.tar.gz` instead.

   ```bash
   OPENAEV_VERSION=3.261029.0
   sudo mkdir -p /opt/openaev && sudo chown "$USER" /opt/openaev
   cd /opt/openaev
   wget "https://github.com/OpenAEV-Platform/openaev/releases/download/${OPENAEV_VERSION}/openaev-release-${OPENAEV_VERSION}.tar.gz"
   tar xzf "openaev-release-${OPENAEV_VERSION}.tar.gz"
   cd openaev
   ls
   ```

   `ls` shows `application.properties` and `openaev-api.jar`.

3. Write your configuration in a file of its own, `openaev.properties`, which you pass to OpenAEV at start and which takes precedence over `application.properties`. Replace the credentials and host names with those of your shared services and of your mail relay.

   ```bash
   cat > openaev.properties <<EOF
   openaev.base-url=http://localhost:8080
   openaev.admin.email=admin@example.com
   openaev.admin.password=a-strong-password
   openaev.admin.token=$(python3 -c 'import uuid; print(uuid.uuid4())')
   openaev.admin.encryption_key=$(openssl rand -hex 24)
   openaev.admin.encryption_salt=$(openssl rand -hex 24)
   openaev.healthcheck.key=$(python3 -c 'import uuid; print(uuid.uuid4())')
   spring.datasource.url=jdbc:postgresql://localhost:5432/openaev
   spring.datasource.username=openaev
   spring.datasource.password=postgres-password
   engine.url=http://localhost:9200
   minio.endpoint=localhost
   minio.port=9000
   minio.access-key=openaev
   minio.access-secret=s3-secret
   openaev.rabbitmq.hostname=localhost
   openaev.rabbitmq.user=openaev
   openaev.rabbitmq.pass=rabbitmq-password
   spring.mail.host=smtp.example.com
   spring.mail.port=587
   spring.mail.username=openaev@example.com
   spring.mail.password=smtp-password
   EOF
   chmod 600 openaev.properties
   ```

   Keep a copy of `openaev.properties` in your secret store: it holds the encryption key and salt. The salt must be at least 8 bytes long. Every other key is in the [OpenAEV configuration reference](/reference/openaev/configuration/).

4. Start OpenAEV once the shared services are healthy. It listens on port 8080.

   ```bash
   java -jar openaev-api.jar --spring.config.additional-location=file:./openaev.properties
   ```

   :::verify
   In another terminal:
   ```bash
   cd /opt/openaev/openaev
   KEY=$(grep '^openaev.healthcheck.key=' openaev.properties | cut -d= -f2)
   curl -fsS "http://localhost:8080/api/health?health_access_key=$KEY" >/dev/null && echo "OpenAEV OK"
   ```
   It prints `OpenAEV OK`. You can sign in at `http://localhost:8080` with the admin email and password of `openaev.properties`.
   :::
:::
:::::
::::::

## What's next

- [Publish the platforms over HTTPS](/deploy/install/reverse-proxy/), to publish the platform over HTTPS. Set the public URL in `app.base_url` (OpenCTI) or `openaev.base-url` (OpenAEV).
- [Configure the shared services](/deploy/configure/shared-services/), for S3, mail and proxies.
- [Run an integration without XTM Composer](/deploy/integrations/manual/), to add connectors, collectors and injectors.
