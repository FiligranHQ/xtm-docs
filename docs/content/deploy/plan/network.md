---
title: Network and ports
description: The ports the XTM stack publishes, the internal addresses its services use to reach each other, and the outbound connections it makes.
doc_type: reference
products:
  - suite
personas: [platform-engineer]
owner: docs-platform
last_reviewed: 2026-10-31
sidebar_position: 6
---

This page lists the network side of the XTM stack as [FiligranHQ/xtm-docker](https://github.com/FiligranHQ/xtm-docker) defines it: the ports it publishes on the host, the service names and ports the containers use to reach each other, who talks to whom, the public URLs, and the connections the stack makes to the outside. Use it to open a firewall, to check for port conflicts or to put the stack behind a reverse proxy.

::diagram{id=deploy-plan-network}

## Ports published on the host

| Host port | Service | Container port | What it serves | Starts with | Set in |
| --- | --- | --- | --- | --- | --- |
| `OPENCTI_PORT`, `8080` by default | `opencti` | 8080 | OpenCTI web interface and GraphQL API | `opencti` | `.env` |
| 8081 | `openaev` | 8080 | OpenAEV web interface and API | `openaev` | `docker-compose.yml`, with `OPENAEV_PORT` in `.env` |
| `XTM_ONE_PORT`, `8090` by default | `xtm-one` | 4000 | XTM One web interface and API | Always | `.env` |
| 8082 | `opencrq` | 3002 | OpenCRQ web interface and API | Always | `modules/opencrq.yml`, with `OPENCRQ_PORT` in `.env` for the public URL |
| 9200 | `elasticsearch` | 9200 | Elasticsearch HTTP API | `opencti` or `openaev` | `docker-compose.yml` |
| 5672 | `rabbitmq` | 5672 | RabbitMQ, Advanced Message Queuing Protocol (AMQP) | `opencti` or `openaev` | `docker-compose.yml` |
| 15672 | `rabbitmq` | 15672 | RabbitMQ management interface and API | `opencti` or `openaev` | `docker-compose.yml` |
| 6379 | `redis` | 6379 | Redis | Always | `docker-compose.yml` |
| 9000 | `minio` | 9000 | S3 API of RustFS | Always | `docker-compose.yml` |
| Not published | `pgsql` | 5432 | PostgreSQL | Always | |

The host port of OpenAEV is written in `docker-compose.yml` (`"8081:8080"`). To move it, replace the mapping in `docker-compose.override.yml`, as [Change the stack without editing docker-compose.yml](/deploy/install/docker-compose/#change-the-stack-without-editing-docker-compose-yml) describes, and set `OPENAEV_PORT` to the same value: `OPENAEV_PORT` builds the public URL of OpenAEV, not the port mapping. For port 8082:

```yaml
services:
  openaev:
    ports: !override
      - "8083:8080"
```

Without `!override`, Docker Compose adds 8083 to 8081 instead of replacing it. The host port of OpenCRQ, 8082, moves the same way, in the `opencrq` service.

:::verify[See what your stack publishes]
```bash
docker compose ps --format '{{.Service}}\t{{.Ports}}'
```
Each running service is listed with its published ports, such as `0.0.0.0:6379->6379/tcp` for Redis.
:::

:::warning[The shared services are open on every interface]
xtm-docker publishes Elasticsearch, RabbitMQ, Redis and S3 storage on every network interface of the host. Elasticsearch runs with its security turned off (`xpack.security.enabled=false`) and Redis without a password. The modules reach these services on the internal network and do not need the published ports. On a host others can reach, remove the ports of these services in `docker-compose.override.yml` with `ports: !reset []`, or bind them to the local machine only with `ports: !override` and, for example, `"127.0.0.1:9200:9200"`. See [Harden the deployment](/deploy/operate/harden/#close-elasticsearch-and-redis-to-the-network).
:::

## Inside the stack

Every service is on one Docker network, `<COMPOSE_PROJECT_NAME>_default`: `xtm_default` with the `COMPOSE_PROJECT_NAME=xtm` of `.env.sample`. The containers reach each other by service name, so the public host names do not need to resolve inside Docker.

| Address | Reached by |
| --- | --- |
| `http://opencti:8080` | The OpenCTI worker and connectors, XTM Composer, XTM One, OpenAEV (on `/graphql`), and OpenCRQ once a tenant is connected to it |
| `http://openaev:8080` | The OpenAEV collectors and injectors, XTM Composer, XTM One, OpenCTI, and OpenCRQ once a tenant is connected to it |
| `http://xtm-one:4000` | OpenCTI, OpenAEV, OpenCRQ |
| `http://opencrq:3002` | XTM One |
| `elasticsearch:9200` | OpenCTI, OpenAEV |
| `rabbitmq:5672` | OpenCTI, OpenAEV, and the integrations and workers they give the address to |
| `rabbitmq:15672` | OpenCTI and OpenAEV, on the RabbitMQ management API |
| `redis:6379` | OpenCTI, XTM One and its worker |
| `minio:9000` | OpenCTI, OpenAEV, XTM One and its worker, OpenCRQ and the `opencrq-bucket-init` job |
| `pgsql:5432` | OpenAEV (database `openaev`), XTM One and its worker (database `xtm_one`), the `xtm-one-db-init` job, OpenCRQ (database `opencrq`) and the `opencrq-db-init` job |

The containers XTM Composer starts join the same network: xtm-docker sets `OPENCTI__DAEMON__DOCKER__NETWORK_MODE` and `OPENAEV__DAEMON__DOCKER__NETWORK_MODE` to `${COMPOSE_PROJECT_NAME}_default`. An integration you run yourself must join it too, to reach its platform and RabbitMQ: see [Run an integration without XTM Composer](/deploy/integrations/manual/).

## Who talks to whom

| From | To | Address | Credential |
| --- | --- | --- | --- |
| OpenCTI | XTM One | `XTM__XTM_ONE_URL=http://xtm-one:4000` | `PLATFORM_REGISTRATION_TOKEN` |
| OpenAEV | XTM One | `OPENAEV_XTM_ONE_URL=http://xtm-one:4000` | `PLATFORM_REGISTRATION_TOKEN` |
| XTM One | OpenCTI | `OPENCTI_API_URL=http://opencti:8080` | |
| XTM One | OpenAEV | `OPENAEV_API_URL=http://openaev:8080` | |
| OpenCRQ | XTM One | `XTM_ONE__URL` | `PLATFORM_REGISTRATION_TOKEN` |
| XTM One | OpenCRQ | `OPENCRQ_API_URL=http://opencrq:3002` | |
| OpenCTI | OpenAEV | `XTM__OPENAEV_API_URL=http://openaev:8080` | `OPENAEV_ADMIN_TOKEN` |
| OpenAEV | OpenCTI | `openaev.xtm.opencti.api-url`, `http://opencti:8080/graphql` | `OPENCTI_ADMIN_TOKEN` |
| OpenCRQ | OpenCTI, for one tenant | `CONNECTORS__OPENCTI__<tenant>__URL`, such as `http://opencti:8080` | `CONNECTORS__OPENCTI__<tenant>__TOKEN` |
| OpenCRQ | OpenAEV, for one tenant | `CONNECTORS__OPENAEV__<tenant>__URL`, such as `http://openaev:8080` | `CONNECTORS__OPENAEV__<tenant>__TOKEN` |
| XTM Composer | OpenCTI and OpenAEV | `http://opencti:8080`, `http://openaev:8080` | `OPENCTI_ADMIN_TOKEN`, `OPENAEV_ADMIN_TOKEN` |
| XTM Composer | Docker on the host | `/var/run/docker.sock`, mounted in the container | |
| Worker and connectors | OpenCTI | `OPENCTI_URL=http://opencti:8080` | `OPENCTI_ADMIN_TOKEN` |
| Collectors and injectors | OpenAEV | `OPENAEV_URL=http://openaev:8080` | `OPENAEV_ADMIN_TOKEN` |

OpenCTI connectors and OpenAEV injectors also connect to RabbitMQ, with the host name, port and credentials their platform gives them when they register: `rabbitmq` and `5672` in xtm-docker. If an integration runs where that name does not resolve, it cannot reach the queue.

OpenCRQ reaches OpenCTI and OpenAEV with the address and token set for each of its tenants on the `opencrq` service. xtm-docker sets them for its first tenant, `Default`; [Configure OpenCRQ](/deploy/configure/opencrq/#connectors-of-each-tenant) lists the variables. It does not only read through that link: at each start, it creates in OpenCTI the Live Stream `OpenCRQ threat feed`, or brings it up to date. See [Link OpenCRQ to OpenCTI and OpenAEV](/deploy/configure/connect-modules/#link-opencrq-to-opencti-and-openaev) for what its token needs.

XTM Composer controls Docker on the host through its socket. Treat access to the `xtm-composer` container like access to the host.

## Public URLs

Browsers, and the links one platform shows to another, use the public URL of each platform. xtm-docker builds it from three variables per platform:

| Platform | Variables | Default | Passed to the platform as |
| --- | --- | --- | --- |
| OpenCTI | `OPENCTI_EXTERNAL_SCHEME`, `OPENCTI_HOST`, `OPENCTI_PORT` | `http://localhost:8080` | `APP__BASE_URL`, and `OPENAEV_XTM_OPENCTI_URL` on OpenAEV |
| OpenAEV | `OPENAEV_EXTERNAL_SCHEME`, `OPENAEV_HOST`, `OPENAEV_PORT` | `http://localhost:8081` | `OPENAEV_BASE-URL`, and `XTM__OPENAEV_URL` on OpenCTI |
| XTM One | `XTM_ONE_EXTERNAL_SCHEME`, `XTM_ONE_HOST`, `XTM_ONE_PORT` | `http://localhost:8090` | `BASE_URL` and `FRONTEND_URL`, on `xtm-one` and `xtm-one-worker` |
| OpenCRQ | `OPENCRQ_EXTERNAL_SCHEME`, `OPENCRQ_HOST`, `OPENCRQ_PORT` | `http://localhost:8082` | `XTM_ONE__PLATFORM_URL`, on `opencrq` |

The public URL is also the identity each module signs its requests to the others with. XTM One also accepts tokens addressed to its internal URL: xtm-docker sets `XTM_AUTH_ACCEPTED_AUDIENCES=http://xtm-one:4000`.

To publish the platforms over TLS behind a reverse proxy, set the scheme to `https` together with the host and the port, for example `OPENCTI_EXTERNAL_SCHEME=https`, `OPENCTI_HOST=opencti.example.com` and `OPENCTI_PORT=443`. See [Publish the platforms over HTTPS](/deploy/install/reverse-proxy/).

## Outbound connections

| From | To | Set by |
| --- | --- | --- |
| OpenCTI | Your mail server, on port 25 | `SMTP_HOST`. The port is written in `docker-compose.yml` (`SMTP__PORT=25`) |
| OpenAEV | Your mail server | `SMTP_HOST` and `SMTP_PORT` |
| OpenAEV | Your IMAP server, when `OPENAEV_MAIL_IMAP_ENABLED=true` | `IMAP_HOST` and `IMAP_PORT`, `993` in `.env.sample` |
| Docker on the host | Docker Hub, and `docker.elastic.co` for Elasticsearch, to download the images | The `image` of each service |
| XTM Composer | The registry of each integration image it deploys | See [Pull integrations from a private registry](/deploy/integrations/private-registry/) |
| OpenCRQ | The public CVE sources: `services.nvd.nist.gov` (NIST NVD), `api.first.org` (FIRST EPSS) and `www.cisa.gov` (CISA KEV) | A tenant administrator, who turns each source on in **Integrations**: see [How CVE sources weigh your vulnerabilities](/opencrq/use/sources/vulnerability-sources/#turn-them-on) |
| OpenCRQ | `raw.githubusercontent.com`, for the default threats it loads | `CONNECTORS__DEFAULT_THREATS__DATASET_URL` points it to a mirror, or, empty, turns the default threats off. See the [OpenCRQ configuration](/reference/opencrq/configuration/#connectors) |
| OpenCRQ | `telemetry.ocrq.filigran.io`, for the anonymous usage telemetry | On by default. `TELEMETRY__USAGE__ENABLED=false` on the `opencrq` service turns it off |
| OpenCRQ | `login.microsoftonline.com` and `graph.microsoft.com` for Microsoft Defender, `public-api.drata.com` by default for Drata, for a tenant connected to them | The variables of each tenant: see [Configure OpenCRQ](/deploy/configure/opencrq/#connectors-of-each-tenant) |

Each integration also connects to the systems it works with. Its own documentation says which.

## What's next

- [The XTM stack](/deploy/plan/architecture/): the modules and the shared services.
- [Publish the platforms over HTTPS](/deploy/install/reverse-proxy/).
- [Connect the modules to each other](/deploy/configure/connect-modules/).
- [Harden the deployment](/deploy/operate/harden/).
