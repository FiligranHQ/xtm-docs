---
title: Install with Docker Compose
description: Install the XTM stack for production on one host with the official Docker Compose deployment, with secrets, public URLs, persistent data and pinned versions.
doc_type: how-to
products:
  - suite
personas: [platform-engineer]
owner: docs-platform
last_reviewed: 2026-10-31
migrated_from:
  - opencti:/deployment/installation/
  - openaev:/deployment/installation/
---

This page installs the XTM stack for production on one host with the official Docker deployment, [FiligranHQ/xtm-docker](https://github.com/FiligranHQ/xtm-docker). At the end, the whole stack, XTM One, OpenCTI, OpenAEV, OpenCRQ, XTM Composer and their default integrations, runs with strong secrets, the public addresses of your users, data kept in Docker volumes and versions that change only when you decide. To try the suite first, follow [Run the XTM suite locally](/deploy/quick-start/): it uses the same files with fewer steps.

::diagram{id=deploy-install-docker-compose}

:::note[The Compose files of the OpenCTI repository]
[OpenCTI-Platform/docker](https://github.com/OpenCTI-Platform/docker) is a second Compose deployment, for OpenCTI with XTM One and XTM Composer, without OpenAEV. It has an OpenSearch variant, `docker-compose.opensearch.yml`. Its `.env.sample` names the OpenCTI and XTM One settings as xtm-docker does, but its services differ (XTM One has its own PostgreSQL, `pgsql-xtm-one`) and so does its mail setting (`SMTP_HOSTNAME`): the commands of this page do not apply to it as they are. This page follows xtm-docker.
:::

## Before you start

- A Linux host with Docker Engine 20.10 or later and Docker Compose 2.24.4 or later: `docker compose version` prints it. An earlier v2 stops on the `include` list of `docker-compose.yml` (2.20) or on the `!override` tags of this page (2.24.4). To install them, follow the [Docker Engine installation guide](https://docs.docker.com/engine/install/).
- 16 GB of RAM at least, 32 GB recommended for production, and 50 GB of free disk. [Sizing](/deploy/plan/sizing/) says how much each module needs.
- An Intel or AMD (amd64) processor: XTM One is published for amd64 only.
- A Docker Hub account with access to `filigran/opengrc-platform`, the image of OpenCRQ. OpenCRQ is not released yet and its image is not public: without that access, `docker compose pull` and `docker compose up -d` stop at that image, and the stack does not start.
- Python 3, to generate the secrets.
- For each platform users reach, the host name they will type, such as `opencti.example.com`, and a reverse proxy that serves it over HTTPS: see [Publish the platforms over HTTPS](/deploy/install/reverse-proxy/).

## Prepare the host

:::steps
1. Raise the memory map limit that Elasticsearch needs, now and after every restart.

   ```bash
   echo 'vm.max_map_count=262144' | sudo tee /etc/sysctl.d/99-xtm-elasticsearch.conf
   sudo sysctl --system
   ```

   :::verify
   ```bash
   sysctl vm.max_map_count
   ```
   It prints `vm.max_map_count = 262144`.
   :::

2. Make Docker start with the host. Every service of the stack has `restart: always`, except `xtm-one-db-init`, which runs once at each start, so the stack comes back when Docker does.

   ```bash
   sudo systemctl enable --now docker
   ```

3. Get the deployment.

   ```bash
   git clone https://github.com/FiligranHQ/xtm-docker.git
   cd xtm-docker
   ```
:::

## Configure the stack

`docker-compose.yml` includes one file per shared service, under `infra/`, and one file per module, under `modules/`: `xtm-one.yml`, `opencti.yml`, `openaev.yml`, `opencrq.yml` and `xtm-composer.yml`. Every service they list starts, and XTM Composer manages both OpenCTI and OpenAEV.

All the configuration is in `.env`, except the ports, the image versions and the number of workers, which these Compose files set. Change those in `docker-compose.override.yml`, never in the files of xtm-docker: see [Change the stack without editing docker-compose.yml](#change-the-stack-without-editing-docker-compose-yml).

:::steps
1. Create `.env` with fresh secrets. The script copies `.env.sample` and gives every password, key and token a random value: a UUID for the IDs, admin tokens and health check keys, a 32-byte base64 key for `OPENCTI_ENCRYPTION_KEY`, and a random string for the others.

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
           v = None
       out.append(f'{k}={v}\n' if v is not None else line)
   open('.env', 'w').writelines(out)
   PY
   chmod 600 .env
   ```

   `.env` now holds every secret of the stack, including the encryption keys of OpenCTI and OpenAEV. Keep a copy in your secret store: you need the same values to restore the stack.

2. Set the admin accounts. Give `OPENCTI_ADMIN_EMAIL`, `OPENAEV_ADMIN_EMAIL` and `XTM_ONE_ADMIN_EMAIL` the address of your administrator. `XTM_ONE_ADMIN_EMAIL` must be the admin email of the platforms XTM One connects to: XTM One finds the user by this address.

   ```bash
   ADMIN=admin@example.com
   sed -i.bak -E "s/^(OPENCTI|OPENAEV|XTM_ONE)_ADMIN_EMAIL=.*/\1_ADMIN_EMAIL=${ADMIN}/" .env
   ```

3. Set the mail server. Replace the `SMTP_*` values of `.env` with those of your relay: `SMTP_HOST`, `SMTP_PORT`, `SMTP_USERNAME`, `SMTP_PASSWORD`, and `SMTP_AUTH`, `SMTP_SSL_ENABLE`, `SMTP_STARTTLS_ENABLE` set to `true` or `false`. If OpenAEV does not read a mailbox, set `OPENAEV_MAIL_IMAP_ENABLED=false` and leave the `IMAP_*` values.

   :::note
   OpenAEV uses every `SMTP_*` value. OpenCTI uses only `SMTP_HOST`, on port 25 without authentication, as `docker-compose.yml` sets it. To send OpenCTI mail through an authenticated relay, see [Configure the shared services](/deploy/configure/shared-services/).
   :::

4. Fit Elasticsearch to the host. `ELASTIC_MEMORY_SIZE` is the Java heap of Elasticsearch, `4G` in `.env.sample`.

   | RAM of the host | `ELASTIC_MEMORY_SIZE` |
   | --- | --- |
   | 16 GB | `2G` |
   | 32 GB | `4G` |
   | 64 GB or more | `8G` |

   ```bash
   sed -i.bak 's/^ELASTIC_MEMORY_SIZE=.*/ELASTIC_MEMORY_SIZE=8G/' .env
   ```

5. Set the public URLs. Each platform builds its URL from three variables of `.env`: the scheme, the host and the port that users type. For platforms served over HTTPS by a reverse proxy on port 443, set:

   ```bash
   OPENCTI_EXTERNAL_SCHEME=https
   OPENCTI_HOST=opencti.example.com
   OPENCTI_PORT=443

   OPENAEV_EXTERNAL_SCHEME=https
   OPENAEV_HOST=openaev.example.com
   OPENAEV_PORT=443

   XTM_ONE_EXTERNAL_SCHEME=https
   XTM_ONE_HOST=xtm-one.example.com
   XTM_ONE_PORT=443

   OPENCRQ_EXTERNAL_SCHEME=https
   OPENCRQ_HOST=opencrq.example.com
   OPENCRQ_PORT=443
   ```

   These URLs are also the identity each product signs its requests to the others with. Inside the stack, the containers reach each other on their service names (`http://opencti:8080`, `http://openaev:8080`, `http://xtm-one:4000`, `http://opencrq:3002`), so the public host names do not need to resolve inside Docker.

6. Publish the platforms on local ports for the reverse proxy. `docker-compose.yml` publishes OpenCTI on `OPENCTI_PORT` and XTM One on `XTM_ONE_PORT`, so with the values above Docker would take port 443 that the proxy needs. It also publishes Elasticsearch and Redis, which run without authentication, on every interface. This command creates `docker-compose.override.yml`, which binds every published port to the host itself, with OpenCTI on 8080, OpenAEV on 8081 and XTM One on 8090. `ports: !override` replaces the ports of `docker-compose.yml` instead of adding to them:

   ```bash
   cat > docker-compose.override.yml <<'EOF'
   services:
     opencti:
       ports: !override
         - "127.0.0.1:8080:8080"
     openaev:
       ports: !override
         - "127.0.0.1:8081:8080"
     xtm-one:
       ports: !override
         - "127.0.0.1:8090:4000"
     opencrq:
       ports: !override
         - "127.0.0.1:8082:3002"
     elasticsearch:
       ports: !override
         - "127.0.0.1:9200:9200"
     redis:
       ports: !override
         - "127.0.0.1:6379:6379"
     minio:
       ports: !override
         - "127.0.0.1:9000:9000"
     rabbitmq:
       ports: !override
         - "127.0.0.1:5672:5672"
         - "127.0.0.1:15672:15672"
   EOF
   ```

   :::note
   An integration that runs on another host, outside XTM Composer, needs to reach some of these services. See [Run an integration without XTM Composer](/deploy/integrations/manual/) before you close them.
   :::

7. Pin the versions. `docker-compose.yml` uses the `latest` tag, so every `docker compose pull` could upgrade a platform. Write the releases you install in `docker-compose.override.yml` with the commands of [Pin the image versions](#pin-the-image-versions), here the current ones: `OPENCTI_VERSION=7.261030.1`, `OPENAEV_VERSION=3.261029.0` and `XTM_ONE_VERSION=1.261112.0`. OpenCRQ takes its tag from `OPENCRQ_VERSION` in `.env`, `main` by default, the latest build: set `OPENCRQ_VERSION=sha-ab702fd`, the build these pages describe.

   The OpenCTI connectors and the OpenAEV collectors and injectors follow the version of their platform. xtm-docker pins RustFS, the S3 storage, to `1.0.4`. XTM Composer, the `alpine/openssl` key generator and the AWS command line interface of `opencrq-bucket-init` keep `latest`. Check which versions work together in [Compatibility between modules](/deploy/upgrade/compatibility/).

8. Size the OpenCTI workers. The worker writes the data that connectors send to OpenCTI; `modules/opencti.yml` starts 3 of them. For more throughput, set `replicas` of the `worker` service in `docker-compose.override.yml`: add the `worker` lines below under the `services:` key of the file, above the `# Pinned versions` line.

   ```yaml
   services:
     worker:
       deploy:
         replicas: 4
   ```

   [Scale: workers, platforms and clusters](/deploy/operate/scale/) explains when to add workers.

   :::verify
   ```bash
   docker compose config --quiet && echo "configuration OK"
   docker compose config --services
   ```
   It prints `configuration OK`, then the 32 services of the stack: XTM One with its worker and `xtm-one-db-init`, OpenCTI with its worker and ten connectors, OpenAEV with four collectors and two injectors, OpenCRQ with `opencrq-db-init` and `opencrq-bucket-init`, XTM Composer with `rsa-key-generator`, and the five shared services.
   :::
:::

## Start the stack

:::steps
1. Download the images, then start the stack.

   ```bash
   docker compose pull
   docker compose up -d
   ```

   The first start creates the databases and the indices: allow a few minutes.

2. Wait until the platforms are healthy.

   ```bash
   docker compose ps --format '{{.Service}} {{.Health}}' | grep -E '^(opencti|openaev|xtm-one|opencrq) '
   ```

   :::verify
   `opencti`, `openaev`, `xtm-one` and `opencrq` say `healthy`. On the host, OpenCTI, OpenAEV and XTM One answer their health check on their local port:

   ```bash
   set -a; . ./.env; set +a
   curl -fsS "http://127.0.0.1:8080/health?health_access_key=$OPENCTI_HEALTHCHECK_ACCESS_KEY" >/dev/null && echo "OpenCTI OK"
   curl -fsS "http://127.0.0.1:8081/api/health?health_access_key=$OPENAEV_HEALTHCHECK_KEY" >/dev/null && echo "OpenAEV OK"
   curl -fsS "http://127.0.0.1:8090/api/health" >/dev/null && echo "XTM One OK"
   ```
   :::

3. Create the owner account of OpenCRQ before you publish it: see [Install OpenCRQ](/deploy/install/opencrq/#create-the-first-administrator). Until it exists, anyone who reaches OpenCRQ can create it.

4. Put the reverse proxy in front of the platforms, as [Publish the platforms over HTTPS](/deploy/install/reverse-proxy/) describes, then sign in at each public URL with the admin email and password of `.env`.
:::

## Change the stack without editing docker-compose.yml

`docker-compose.yml` belongs to the xtm-docker repository: change it, and every `git pull` conflicts with your change. Put every local change in `docker-compose.override.yml`, next to it, instead. Docker Compose reads that file after `docker-compose.yml` on every command and merges the two. `.env` keeps the values xtm-docker reads from it; the override file holds the rest: environment variables, mounts, ports, image versions and the number of workers.

This example raises the log level of OpenCTI, mounts a folder of certificates in it, publishes it on the host only, and starts four workers:

```yaml
services:
  opencti:
    environment:
      - APP__APP_LOGS__LOGS_LEVEL=info
    volumes:
      - ./certs:/opt/opencti/certs:ro
    ports: !override
      - "127.0.0.1:8080:8080"
  worker:
    deploy:
      replicas: 4
```

Docker Compose merges each key in its own way:

| Key | What the override file does |
| --- | --- |
| `environment` | Adds the variable, or replaces the value of the variable of the same name |
| `volumes` | Adds the mount, or replaces the mount on the same path in the container |
| `ports` | Adds to the ports of `docker-compose.yml`. `ports: !override` followed by a list replaces them, `ports: !reset []` removes them |
| `depends_on` | Adds the dependency. `<service>: !reset null` under it removes one |
| `profiles` | Adds to the profiles. `profiles: !override ["disabled"]` keeps the service from starting |
| `image`, `deploy.replicas` and the other single values | Replaces the value |
| A service that `docker-compose.yml` does not have | Adds it |

`!reset` needs Docker Compose 2.24 or later, and `!override` 2.24.4 or later. Above the pinned versions, write one `services:` key, and each service once under it: when a page gives lines for a service your file already has, add them under that service. Docker Compose refuses a file that repeats a key, with `mapping key "opencti" already defined`.

:::verify
```bash
docker compose config --quiet && echo "configuration OK"
docker compose config | grep -E 'LOGS_LEVEL|host_ip|replicas'
```
With the example as `docker-compose.override.yml`, it prints `configuration OK`, then `APP__APP_LOGS__LOGS_LEVEL: info`, `host_ip: 127.0.0.1` and `replicas: 4`: the merged configuration has the values of the override file.
:::

Apply the change with `docker compose up -d`: Docker Compose recreates the containers whose configuration changed. After a `git pull`, run `docker compose config --quiet` again, and compare what you replaced with `!override` to the new `docker-compose.yml`.

### Pin the image versions

These commands pin the OpenCTI, OpenAEV and XTM One images of the stack. They write the image of each service, with the versions you set, at the end of `docker-compose.override.yml`. Run them again to change the versions: they replace that part of the file.

```bash
OPENCTI_VERSION=7.261030.1
OPENAEV_VERSION=3.261029.0
XTM_ONE_VERSION=1.261112.0
touch docker-compose.override.yml
sed -i.bak '/^# Pinned versions/,$d' docker-compose.override.yml
PINNED=$(docker compose config --format json | python3 -c '
import json, sys
tags = dict(zip(["opencti", "openaev", "xtmone"], sys.argv[1:]))
for name, service in sorted(json.load(sys.stdin)["services"].items()):
    repo = service["image"].rsplit(":", 1)[0]
    vendor = repo.split("/")[0]
    if vendor in tags:
        print(f"  {name}:\n    image: {repo}:{tags[vendor]}")
' "$OPENCTI_VERSION" "$OPENAEV_VERSION" "$XTM_ONE_VERSION")
printf '%s\n' '# Pinned versions: keep this part last, the pinning commands rewrite it' '---' 'services:' "$PINNED" >> docker-compose.override.yml
```

The `---` line starts a second part of the file, which Docker Compose merges like the first. Keep your other changes above the `# Pinned versions` line: the commands delete everything from that line to the end of the file.

:::verify
```bash
docker compose config --images | grep -E '^(opencti|openaev|xtmone)/'
```
Every image ends with the version you set.
:::

## Where the data is

The data of the stack lives in Docker volumes, named after `COMPOSE_PROJECT_NAME` (`xtm` in `.env.sample`). They survive `docker compose down` and a restart of the host, and only `docker compose down -v` deletes them.

| Volume | Holds |
| --- | --- |
| `xtm_esdata` | Elasticsearch: the knowledge of OpenCTI and the indices of OpenAEV |
| `xtm_pgsqlshareddata` | PostgreSQL: the `openaev`, `opencrq` and `xtm_one` databases |
| `xtm_s3data` | RustFS, the S3 object storage: files of OpenCTI, OpenAEV, OpenCRQ and XTM One |
| `xtm_redisdata` | Redis, used by OpenCTI for its event stream and by XTM One |
| `xtm_amqpdata` | RabbitMQ: the messages waiting for the workers and integrations |
| `xtm_rsakeys` | The private key of XTM Composer |

```bash
docker volume ls --filter label=com.docker.compose.project=xtm
```

Back them up together with `.env`: see [Back up and restore](/deploy/operate/backup/).

## What's next

- [Publish the platforms over HTTPS](/deploy/install/reverse-proxy/), to publish the platforms over HTTPS.
- [Connect the modules to each other](/deploy/configure/connect-modules/), to check the links between XTM One, OpenCTI, OpenAEV and XTM Composer.
- [Sign in the first time and keep an emergency account](/deploy/configure/first-access/), before you turn on single sign-on.
- [Start the whole stack on an existing deployment](/deploy/install/modules/), for a deployment that ran only some modules.
- [Install offline or with FIPS 140-3](/deploy/install/air-gapped/), for a host without internet access.
