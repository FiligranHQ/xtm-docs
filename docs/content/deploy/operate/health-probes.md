---
title: Check that the platforms are healthy
description: The health endpoints of XTM One, OpenCTI and OpenAEV, what each answer means, and how to use them in Docker Compose, Kubernetes or a load balancer.
doc_type: how-to
products:
  - suite
personas: [platform-engineer]
owner: docs-platform
last_reviewed: 2026-10-31
migrated_from:
  - opencti:/deployment/advanced/liveness-probe/
---

Each platform of the XTM stack has an HTTP endpoint that says whether it can serve requests. This page lists them, says what each answer means and how to call them, so that Docker Compose, Kubernetes or a load balancer restarts a platform or stops sending it traffic at the right time.

::diagram{id=deploy-operate-health-probes}

## The endpoints

| Platform | Endpoint | Key | Answers `200` when |
| --- | --- | --- | --- |
| XTM One | `/api/health`, on its port (`4000` in the container, `XTM_ONE_PORT` on the host) | None | The application is up |
| OpenCTI | `/health?health_access_key=<key>`, on its port (`8080` in the container, `OPENCTI_PORT` on the host) | `APP__HEALTH_ACCESS_KEY`, from `OPENCTI_HEALTHCHECK_ACCESS_KEY` in `.env` | Elasticsearch, S3 storage, RabbitMQ and Redis all answer |
| OpenAEV | `/api/health?health_access_key=<key>`, on its port (`8080` in the container, `8081` on the host) | `OPENAEV_HEALTHCHECK_KEY` in `.env` | PostgreSQL, RabbitMQ and S3 storage answer |
| OpenCRQ | Any path, such as `/health`, on the health check port (`HEALTHCHECK_HTTP_SERVER__PORT`, `8444` in xtm-docker, not published on the host) | None | Every process of the platform runs; it answers `503` while one is starting, stopped or silent |

The healthchecks of `docker-compose.yml` call these same endpoints, and `docker compose ps` shows their result. The workers, connectors, collectors and injectors of a platform wait for it to be healthy before they start, and XTM Composer waits for OpenCTI and OpenAEV.

## Check the platforms by hand

From the `xtm-docker` folder:

```bash
set -a; . ./.env; set +a
curl -fsS "http://localhost:$XTM_ONE_PORT/api/health" && echo
curl -fsS "http://localhost:$OPENCTI_PORT/health?health_access_key=$OPENCTI_HEALTHCHECK_ACCESS_KEY" && echo
curl -fsS "http://localhost:8081/api/health?health_access_key=$OPENAEV_HEALTHCHECK_KEY" && echo
```

The host port of OpenAEV is written in `docker-compose.yml` (`8081:8080`), not taken from `OPENAEV_PORT`: `OPENAEV_PORT` only builds its public URL. If you change one, change the other.

:::verify
OpenCTI and OpenAEV answer `{"status":"success"}`. A platform you did not start fails to connect.
:::

## What the answers mean

::::tabs
:::tab[OpenCTI]
| Answer | Meaning |
| --- | --- |
| `200` `{"status":"success"}` | The platform and its four dependencies are up |
| `503` with an `error` | A dependency is down, or the platform has not finished starting |
| `401` `{"status":"unauthorized"}` | The key is wrong, or the platform has no key: an empty key and the default `ChangeMe` refuse every call |

Set the key to `public` to answer without a key. Use it only on a network you trust.

:::only{opencti=">=7.261030"}
The platform checks its dependencies in the background, every `app:health_monitoring:dependency_check_interval` milliseconds (default `30000`), and the endpoint returns the last result: a call never contacts a dependency. Until the first check is done, it answers `503` with `Health monitoring not initialized yet`.

Add `&details=true` to get the state of each dependency and the usage figures (`es_used_size`, `s3_used_size`, `queue_consumers`). The details are never given when the key is `public`.
:::

:::only{opencti="<7.261030"}
Each call checks the four dependencies, with a timeout of 15 seconds each.
:::
:::
:::tab[OpenAEV]
| Answer | Meaning |
| --- | --- |
| `200` `{"status":"success"}` | PostgreSQL, RabbitMQ and S3 storage are up |
| `503` | One of them is down, or has not been checked for three intervals |
| `401` | The key is wrong or empty |
| `400` | The `health_access_key` parameter is missing |

OpenAEV checks its dependencies in the background, every `openaev.healthcheck.connectivity-probe-interval` (default `PT10S`, ten seconds), and the endpoint returns the last result. Elasticsearch is checked too, but its failure does not turn the answer into `503`: OpenAEV can still serve requests without it. Add `&details=true` to also get the storage used in PostgreSQL, Elasticsearch and S3.

:::warning[Change the default key]
The default key of OpenAEV is `ChangeMe`, and OpenAEV accepts it. In xtm-docker, the platform falls back to it when `OPENAEV_HEALTHCHECK_KEY` is not set in `.env`. Set your own value.
:::
:::
:::tab[XTM One]
XTM One answers on `/api/health` without a key. The healthcheck of xtm-docker marks it unhealthy when the endpoint does not answer or answers with an HTTP error.
:::
::::

:::only{opencti=">=7.260811"}
## OpenCTI liveness probe

The `/health` endpoint answers only once OpenCTI has started, which can take a long time when it migrates its data. An orchestrator that uses it to decide whether the process is alive may restart a platform in the middle of a migration. OpenCTI has a second endpoint for that:

| Probe | Endpoint | Port | Answers | Checks |
| --- | --- | --- | --- | --- |
| Liveness | `<base_path>/health/liveness` | `app:liveness_port`, off by default | As soon as the process starts | The process runs |
| Readiness | `<base_path>/health` | `app:port` | Once the platform has started | The dependencies |

The liveness endpoint answers `GET` with `200` and `{"status":"success"}`, without a key. Turn it on with a port number in `APP__LIVENESS_PORT`. `<base_path>` is `app:base_path`, empty by default.
:::

## Use the probes in Kubernetes

:::only{opencti=">=7.260811"}
For OpenCTI, use the liveness endpoint for the liveness probe and `/health` for the readiness probe. With `APP__LIVENESS_PORT=8082` and the platform on port 8080:

```yaml
livenessProbe:
  httpGet:
    path: /health/liveness
    port: 8082
  periodSeconds: 10
readinessProbe:
  httpGet:
    path: /health?health_access_key=<your key>
    port: 8080
  periodSeconds: 10
```
:::

For OpenAEV, use `/api/health?health_access_key=<your key>` on port 8080, and for XTM One, `/api/health` on port 4000.

::::only{opencti=">=7.261030"}
:::note[Keep-alive behind a load balancer]
OpenCTI keeps idle connections open for `app:keep_alive_timeout` milliseconds (default `65000`), longer than the idle timeout of most load balancers. The liveness server uses the same value, so a probe sent through a load balancer behaves like the rest of the traffic.
:::
::::

## What's next

- [Observability metrics](/deploy/operate/metrics/), to follow the dependencies over time.
- [Scale: workers, platforms and clusters](/deploy/operate/scale/), to run several OpenCTI nodes behind a load balancer.
- [Troubleshoot the XTM stack](/deploy/operate/troubleshoot/), when a platform stays unhealthy.
- [HTTP endpoints](/reference/opencti/http-endpoints/): every endpoint OpenCTI serves besides GraphQL, health and metrics included.
