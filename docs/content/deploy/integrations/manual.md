---
title: Run an integration without XTM Composer
description: Run an OpenCTI connector or an OpenAEV collector or injector yourself, with Docker Compose or Python, with its own token and network access.
doc_type: how-to
products:
  - opencti
  - openaev
personas: [platform-engineer]
owner: docs-platform
last_reviewed: 2026-10-31
---

This page runs an integration yourself, without XTM Composer: as a Docker Compose service next to the XTM stack, or as a Python program. Use it for an integration that is not in the catalog, or a platform without Enterprise Edition. Read [How integrations run](/deploy/integrations/how-they-run/) first for what an integration needs from its platform.

::diagram{id=deploy-integrations-manual}

## Before you start

- The platform running: OpenCTI, OpenAEV or both.
- The name of the integration and the settings it needs: each one documents them in the `README.md` of its folder, in the repositories [OpenCTI-Platform/connectors](https://github.com/OpenCTI-Platform/connectors), [OpenAEV-Platform/collectors](https://github.com/OpenAEV-Platform/collectors) and [OpenAEV-Platform/injectors](https://github.com/OpenAEV-Platform/injectors).
- Network access from where the integration runs to the platform URL and to RabbitMQ, at the host and port the platform gives out. In the XTM stack, both are on the `xtm_default` network.

## Create a token

::::tabs
:::tab[OpenCTI]
Run each connector with its own user, so you can tell what each one wrote and limit what it can do. OpenCTI has a `Connectors` group for them.

| Connector type | Group of its user | Why |
| --- | --- | --- |
| `EXTERNAL_IMPORT`, `INTERNAL_ENRICHMENT`, `INTERNAL_IMPORT_FILE`, `STREAM` | `Connectors` | It imports, enriches or reads the stream as its own user |
| `INTERNAL_EXPORT_FILE` | A group with an Administrator role that bypasses capabilities | It exports with the rights of the user who asked for the export, which it impersonates |

1. In :menu[Settings > Security > Users], create a user named after the connector, such as `[C] MISP`.
2. Add it to the `Connectors` group, or the administrator group for an export connector.
3. Create an API token for it and copy it: it is the `OPENCTI_TOKEN` of the connector.

A connector can also create its own user. With `CONNECTOR_AUTO_CREATE_SERVICE_ACCOUNT=true`, it creates a service account named `[C] <CONNECTOR_NAME>` in the `Connectors` group, then runs with a token of that account. Its `OPENCTI_TOKEN` must then be allowed to create users. `CONNECTOR_AUTO_CREATE_SERVICE_ACCOUNT_CONFIDENCE_LEVEL` sets its confidence level, 50 by default. xtm-docker starts its datasets and MITRE ATT&CK connectors this way.
:::
:::tab[OpenAEV]
Collectors and injectors can share a token: use the admin token of OpenAEV (`OPENAEV_ADMIN_TOKEN` in the `.env` of xtm-docker) or the token of another administrator account. One account per integration is not needed.

With multi-tenancy, an Enterprise Edition feature, also set `OPENAEV_TENANT_ID` to the UUID of the tenant.
:::
::::

Each integration also needs a unique identifier, a UUIDv4, as `CONNECTOR_ID`, `COLLECTOR_ID` or `INJECTOR_ID`. Generate one per integration and keep it: the platform recognizes the integration by it.

```bash
python3 -c 'import uuid; print(uuid.uuid4())'
```

Beside its URL, token, identifier and name, an OpenCTI connector reads a few settings that every connector shares. The `README.md` of the connector says which ones it needs.

| Variable | Default | What it sets |
| --- | --- | --- |
| `CONNECTOR_TYPE` | Set by most images | The kind of connector, such as `EXTERNAL_IMPORT`, `INTERNAL_ENRICHMENT` or `STREAM`. Set it when the `docker-compose.yml` of the connector does |
| `CONNECTOR_SCOPE` | `not-applicable` | What the connector handles: entity types for an enrichment connector, such as `IPv4-Addr,Domain-Name`, or MIME types for a file import connector, such as `application/pdf` |
| `CONNECTOR_AUTO` | `false` | `true` makes an enrichment or file import connector run on its own on what enters its scope, instead of on request |
| `CONNECTOR_LOG_LEVEL` | `error` | `debug`, `info`, `warn` or `error` |

## Run it with Docker Compose

This example runs the MITRE ATT&CK connector of OpenCTI in its own Compose project, attached to the network of the XTM stack. xtm-docker already runs this connector, as its `connector-mitre` service: for another integration, change the image and the variables its `README.md` lists.

:::::steps
1. Create a folder for the integration with its secrets.

   ```bash
   mkdir connector-mitre && cd connector-mitre
   cat > .env <<EOF
   INTEGRATION_ID=$(python3 -c 'import uuid; print(uuid.uuid4())')
   INTEGRATION_TOKEN=your-integration-token
   EOF
   chmod 600 .env
   ```

   Replace `your-integration-token` with the token of the previous section.

2. Write `docker-compose.yml`. The `default` network is the one of the XTM stack, `xtm_default` with the default `COMPOSE_PROJECT_NAME=xtm`, so the integration reaches `opencti` and `rabbitmq` by name.

   ::::tabs
   :::tab[OpenCTI connector]
   ```yaml
   services:
     connector-mitre:
       image: opencti/connector-mitre:latest
       environment:
         - OPENCTI_URL=http://opencti:8080
         - OPENCTI_TOKEN=${INTEGRATION_TOKEN}
         - CONNECTOR_ID=${INTEGRATION_ID}
         - CONNECTOR_NAME=MITRE ATT&CK
         - CONNECTOR_LOG_LEVEL=error
       restart: always
   networks:
     default:
       external: true
       name: xtm_default
   ```
   :::
   :::tab[OpenAEV injector]
   ```yaml
   services:
     injector-nmap:
       image: openaev/injector-nmap:latest
       environment:
         - OPENAEV_URL=http://openaev:8080
         - OPENAEV_TOKEN=${INTEGRATION_TOKEN}
         - INJECTOR_ID=${INTEGRATION_ID}
         - INJECTOR_NAME=Nmap
       restart: always
   networks:
     default:
       external: true
       name: xtm_default
   ```
   :::
   ::::

3. Start it.

   ```bash
   docker compose up -d
   ```

   :::verify
   ```bash
   docker compose logs --tail 20
   ```
   The logs show the integration registering with the platform and no connection error. It appears in :menu[Integrations], **Deployed** tab (:menu[Data > Ingestion > Connectors] before 7.260811) in OpenCTI, or in **Integrations** > **Collectors** or **Injectors** in OpenAEV.
   :::
:::::

To add the integration to the Compose project of the platform instead, add the same service to its `docker-compose.override.yml` with `OPENCTI_URL=http://opencti:8080`, without the `networks` block, and with the values of `INTEGRATION_ID` and `INTEGRATION_TOKEN` in place of the variables: see [Change the stack without editing docker-compose.yml](/deploy/install/docker-compose/#change-the-stack-without-editing-docker-compose-yml). A separate project keeps the integration and its secrets apart from the stack.

Each integration folder of the repositories also has its own `docker-compose.yml` to start from. To use the files of one release, such as OpenCTI 7.261030.1:

```bash
wget https://github.com/OpenCTI-Platform/connectors/archive/refs/tags/7.261030.1.zip
unzip 7.261030.1.zip
cd connectors-7.261030.1/external-import/mitre
```

The OpenAEV repositories are tagged with the OpenAEV version, such as `3.261029.0`, with one folder per integration at their root, such as `nmap` or `mitre-attack`.

## Run it with Python

Without Docker, an integration runs as a Python program with the same settings, written in a `config.yml` file instead of environment variables.

:::steps
1. Install Python 3 and pip.

   ```bash
   sudo apt install python3 python3-pip
   ```

2. Get the files of the release, as above, and go to the folder of the integration.
3. Copy the sample configuration to `config.yml` and fill it: `opencti.url` and `opencti.token`, or `openaev.url` and `openaev.token`, and the `connector`, `collector` or `injector` block with its `id` and `name`.
4. Install the dependencies and start the program as the `README.md` of the integration says. OpenCTI connectors use a `requirements.txt` in their `src` folder, for example `pip3 install -r requirements.txt` then `python3 -m __main__` for MITRE ATT&CK. OpenAEV collectors and injectors use Poetry.
:::

## Networking

An integration reaches RabbitMQ at the host name and port that the platform gives it when it registers: the RabbitMQ settings of the platform. From another network, that name may not resolve.

::::tabs
:::tab[Override the RabbitMQ address]
Give the connector the address and credentials it must use instead, with these variables or the `mq` block of `config.yml`:

| Variable | `config.yml` key |
| --- | --- |
| `MQ_HOST` | `mq.host` |
| `MQ_PORT` | `mq.port` |
| `MQ_VHOST` | `mq.vhost` |
| `MQ_USE_SSL` | `mq.use_ssl` |
| `MQ_USER` | `mq.user` |
| `MQ_PASS` | `mq.pass` |

These variables are read by OpenCTI connectors.
:::
:::tab[Use HTTP instead of RabbitMQ (OpenCTI)]
An OpenCTI connector can skip RabbitMQ entirely.

- `CONNECTOR_QUEUE_PROTOCOL=api` sends the bundles it produces to the OpenCTI API over HTTP instead of RabbitMQ (`amqp`, the default). It applies to import, enrichment and file import connectors. OpenCTI refuses a request larger than `app:max_payload_body_size` (`APP__MAX_PAYLOAD_BODY_SIZE`), `50mb` by default: raise it on the platform for a connector that sends large bundles.
- `CONNECTOR_LISTEN_PROTOCOL=api` makes the connector receive its jobs over HTTP: it starts an HTTP server, and the OpenCTI workers call it. It applies to enrichment and export connectors.

| Variable | Default | What it sets |
| --- | --- | --- |
| `CONNECTOR_LISTEN_PROTOCOL_API_PORT` | `7070` | The port of the HTTP server |
| `CONNECTOR_LISTEN_PROTOCOL_API_PATH` | `/api/callback` | The path the workers call |
| `CONNECTOR_LISTEN_PROTOCOL_API_URI` | `http://127.0.0.1:7070` | The URL, without the path, at which the workers reach the connector |
| `CONNECTOR_LISTEN_PROTOCOL_API_SSL` | `false` | `true` to serve HTTPS, with `LISTEN_PROTOCOL_API_SSL_KEY`, `LISTEN_PROTOCOL_API_SSL_CERT` and `LISTEN_PROTOCOL_API_SSL_PASSPHRASE` |

An enrichment connector with both set never connects to RabbitMQ. Set `CONNECTOR_LISTEN_PROTOCOL_API_URI` to an address the workers can reach, such as `http://connector-enrichment:7070` on the network of the stack.

The workers make these calls. Three settings of the `worker` service shape them:

| Worker variable | Default | What it sets |
| --- | --- | --- |
| `WORKER_LISTEN_API_SSL_VERIFY` | `false` | `true` makes the workers check the certificate of a connector that serves HTTPS |
| `WORKER_LISTEN_API_HTTP_PROXY` | | A proxy for the calls to `http://` connectors |
| `WORKER_LISTEN_API_HTTPS_PROXY` | | A proxy for the calls to `https://` connectors |
:::
::::

## Check that it works

The platform lists the integration with its status: :menu[Integrations], **Deployed** tab (:menu[Data > Ingestion > Connectors] before 7.260811) in OpenCTI, with the messages waiting in its queue; **Integrations** > **Collectors** or **Integrations** > **Injectors** in OpenAEV. An integration that does not appear has not registered: read its logs for the platform URL or the token it could not use.

## What's next

- [Migrate to XTM Composer](/deploy/integrations/migrate/), to manage the integration from the interface later.
- [Integration model](/develop/integration-model/), to build your own.
