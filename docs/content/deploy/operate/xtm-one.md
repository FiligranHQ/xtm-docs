---
title: Monitor and troubleshoot XTM One
description: Watch XTM One through its health endpoints, metrics, logs and security audit, and find the cause when it does not start, chat does not stream or agents do not answer.
doc_type: how-to
products:
  - xtm-one
personas: [platform-engineer]
owner: docs-xtm-one
last_reviewed: 2026-11-12
migrated_from:
  - xtm-one:/deployment/monitoring/
  - xtm-one:/deployment/troubleshooting/
---

This page tells a platform engineer how XTM One is doing and what to do when it is not. XTM One answers two health endpoints, exposes Prometheus metrics behind a token, writes one JSON line per request, and audits its own configuration; the second half of the page goes from a symptom to its cause. For the logs of the whole stack, see [Troubleshoot the XTM stack](/deploy/operate/troubleshoot/); for what agents did and what it cost, administrators read **Observability** in XTM One.

::diagram{id=xtm-one-deploy-operate-signals}

## Health endpoints

| Endpoint | Answers 200 when | Use it for |
| --- | --- | --- |
| `GET /api/health` | The server of `xtm-one` listens | Liveness: restart the container when it fails |
| `GET /api/health/ready` | The start is complete, and PostgreSQL, Redis and the S3 bucket answer within 4 seconds each | Readiness: send traffic only when it passes |

Neither needs authentication. Both answer on port 4000 of the container, at its root even when XTM One is served under a path.

```bash
docker compose exec xtm-one curl -sS http://localhost:4000/api/health/ready
```

Ready, it prints `{"database":"ok","redis":"ok","s3":"ok","status":"ok"}`. Otherwise it answers HTTP 503: `"status": "starting"` while the platform migrates its database and creates its built-in content, with `startup_error` when that start failed; or `"status":"degraded"`, with the error of each service that did not answer, such as `"s3":"error: bucket not found"`.

The healthcheck of xtm-docker asks `/api/health`, so `healthy` in `docker compose ps` says the server listens, not that it is ready: monitor `/api/health/ready` as well. Each web process answers for itself, and one that is still starting holds a request up to 8 seconds before it answers 503 with `Retry-After`.

The worker has no HTTP port. It writes a heartbeat that `python -m app.healthcheck reconciler` checks: the command exits with 0 while the heartbeat is fresh, and 1 when it is stale or missing. xtm-docker runs no healthcheck on `xtm-one-worker`; to add one, put these lines under `services:` in `docker-compose.override.yml`:

```yaml
  xtm-one-worker:
    healthcheck:
      test: ["CMD", "python", "-m", "app.healthcheck", "reconciler"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 60s
```

On Kubernetes, point the `livenessProbe` of the platform at `/api/health`, its `readinessProbe` at `/api/health/ready`, and the `livenessProbe` of the worker at that command. See [Health checks](/deploy/operate/health-probes/) for the other platforms.

## Metrics

XTM One exposes Prometheus metrics on `/metrics`, port 4000, once both variables are set on `xtm-one`:

| Variable | Default | What it sets |
| --- | --- | --- |
| `PROMETHEUS_ENABLED` | `false` | Turns `/metrics` on |
| `PROMETHEUS_BEARER_TOKEN` | None | The token a scraper sends as `Authorization: Bearer`. 16 characters or more: without it, or shorter, XTM One logs a warning at start and keeps `/metrics` off |

While off, `/metrics` answers 404; without the right token, 401. The metrics:

| Metric | What it measures |
| --- | --- |
| `xtmone_quota_usage` | A gauge of quota consumption, by `user_id`, `usage_type`, `model`, `period` (`daily`, `monthly`, `yearly`) and `platform_id`. It is computed again at each scrape |
| `csp_violation_reports_total` | The violations of the Content Security Policy that browsers reported, by `directive`, `disposition` (`report` or `enforce`) and `origin` (`page` or `extension`) |
| `xtmone_sandbox_*`, `xtmone_turn_resume_results_total` | The code interpreter's workspaces, and the tool results an interrupted chat passed to the one that resumed it |
| `process_*`, `python_*` | The runtime of the process |

Each web process keeps its own counters, and a scrape reads the process that answers it. Apart from `xtmone_quota_usage`, a series is a sample of one process: do not read it with `rate()` or `increase()`. The logs are the exact record.

:::example[Scrape XTM One from the Prometheus of Grand Meridian Hotels]
Marc Dupuis, of IT operations at Grand Meridian Hotels, adds the scrape token to `.env` as `XTM_ONE_PROMETHEUS_TOKEN`, generated with `openssl rand -hex 32`, then sets on `xtm-one` in `docker-compose.override.yml`:

```yaml
services:
  xtm-one:
    environment:
      - PROMETHEUS_ENABLED=true
      - PROMETHEUS_BEARER_TOKEN=${XTM_ONE_PROMETHEUS_TOKEN}
```

The Prometheus of the Paris datacenter scrapes XTM One through the reverse proxy, over HTTPS, and keeps the token in a file:

```yaml
scrape_configs:
  - job_name: xtm-one
    scheme: https
    metrics_path: /metrics
    authorization:
      type: Bearer
      credentials_file: /etc/prometheus/secrets/xtm-one-token
    static_configs:
      - targets: ["xtm-one.grand-meridian.example"]
```

The `xtm-one` target turns up, and `xtmone_quota_usage{period="monthly"}` shows what the CTI team and the SOC spent this month.
:::

## Logs

`xtm-one` and `xtm-one-worker` write to their standard output. `LOG_LEVEL` (`debug`, `info`, `warning`, `error`, `critical`) and `LOG_FORMAT` (`json` or `console`) set them; in xtm-docker, `XTM_ONE_LOG_LEVEL` and `XTM_ONE_LOG_FORMAT` in `.env` do, `info` and `json` by default. Keep `json` in production and ship the lines to your log aggregator.

```bash
docker compose logs -f --since 10m xtm-one xtm-one-worker
```

Every request writes one `http_request` line, with `http_method`, `http_path`, `http_status`, `duration_ms` and `client_ip`. Successful requests to the health endpoints and to `/metrics` are logged at `debug` only, client errors at `warning`, server errors at `error`. Set `debug` for an investigation only, then go back to `info`.

## Security audit

**Settings** > **Access and security** > **Security audit** checks the configuration of XTM One and reports each finding with what to change: a published registration token, a reverse proxy XTM One does not trust, a Content Security Policy left in `report-only`, a missing `FORWARDED_ALLOW_IPS`. The same report is at `GET /api/v1/security-audit`, for an administrator. Read it after an installation, an upgrade or a change of the proxy.

::screenshot{id=xtm-one-settings-security-audit alt="The Security audit of a new platform: its posture needs attention with 4 warnings and no critical finding, 20 of 24 checks pass, each category shows its findings and pass rate, and each warning ends on a recommendation that names the setting or the variable to change."}

## What to watch

Alert on:

- `/api/health/ready` failing for more than a few minutes: XTM One does not serve.
- A rising rate of `http_status` 500 and above in the `http_request` lines.
- `xtm-one-worker` restarting again and again: assignments and background work stop.
- PostgreSQL connections close to `max_connections`: see [Size the platform and the worker](/deploy/configure/xtm-one/#size-the-platform-and-the-worker).
- The disk of the `pgsqlshareddata` and `s3data` volumes, which grow with the knowledge bases, the attachments and the traces.

## Troubleshoot XTM One

Start with the state of the services, the readiness answer and the last lines of both logs:

```bash
docker compose ps
docker compose exec xtm-one curl -sS http://localhost:4000/api/health/ready
docker compose logs --tail 100 xtm-one xtm-one-worker
```

| Symptom | Cause and fix |
| --- | --- |
| Readiness stays `"starting"` | The platform migrates its database: a large database takes longer at the first start of a new version. After `STARTUP_WATCHDOG_SECONDS` (300), the log names the step in progress and the sessions that block the migration. With several platform replicas, one migrates while the others wait |
| Readiness says `"degraded"` | The service in error does not answer. Check it in `docker compose ps`, and the `DATABASE_URL`, `REDIS_URL` or `S3_*` values. `bucket not found`: the bucket was deleted; restart `xtm-one`, which creates it again |
| The start fails in a migration | The log names the migration that failed. Never run the migrations by hand: keep the log, and restore the backup taken before the upgrade, as [Upgrade XTM One](/deploy/upgrade/xtm-one/#roll-back) describes |
| Integrations fail and people are signed out after a restart | `SECRET_KEY` changed. Put the former value back: no other way recovers the credentials it encrypted |
| The administrator cannot sign in | `ADMIN_EMAIL` (`admin@filigran.io` when unset) and `ADMIN_PASSWORD` in `.env` win at every start: check them, then restart `xtm-one`. The restart sets that password again, makes the account an administrator and enables it again if someone disabled it. A password changed in the interface lasts until the next restart |
| An answer shows only once it is complete | The reverse proxy buffers the chat, which streams as server-sent events under `/api/v1/web/chat`. Turn response buffering off for `/api/`. Nginx honours the `X-Accel-Buffering: no` header XTM One sends; another proxy needs the setting. See [Publish the platforms over HTTPS](/deploy/install/reverse-proxy/) |
| The chat hangs or cuts off in the middle of an answer | The proxy closes the stream. An idle stream writes a keep-alive every 15 seconds, so the read timeout must stay above 15 seconds; an agent's answer can take minutes, so keep a request open 300 seconds (`proxy_read_timeout 300s;`). A tunnel with a short origin timeout cuts the chat too |
| The app stops loading once a few tabs are open | The proxy speaks HTTP/1.1 to browsers. A browser opens 6 HTTP/1.1 connections per host at most, and every tab holds its streams. Turn HTTP/2 on between browsers and the proxy (`http2 on;` in Nginx 1.25.1 and later); the proxy may still reach XTM One over HTTP/1.1 |
| The audit log shows the same address for every person | XTM One does not trust the proxy: the log says `A request carried X-Forwarded-For` and names its address. Set `FORWARDED_ALLOW_IPS` to it on `xtm-one`, and make the proxy send `X-Forwarded-For`: XTM One reads neither `X-Real-IP` nor `Forwarded` |
| A platform is registered, but every action on its data fails with 401 | The log of `xtm-one` says why. `Untrusted JWT issuer`: the platform signs with another URL than the one it registered with. `Failed to resolve signing key for`: XTM One cannot reach the URL it names; set `OPENCTI_API_URL`, `OPENAEV_API_URL` or `OPENCRQ_API_URL`. `audience mismatch`: the platform addresses its tokens to another URL than `BASE_URL`; list that URL in `XTM_AUTH_ACCEPTED_AUDIENCES`. Background work fails alone: `BASE_URL` differs on `xtm-one-worker` |
| A platform does not register | Its token differs from `PLATFORM_REGISTRATION_TOKEN`, or XTM One runs with the published default. See [Connect the modules to each other](/deploy/configure/connect-modules/#register-the-platforms-with-xtm-one) |
| Agents answer nothing, or fail at once | No language model in **Settings** > **AI and models** > **Models**, a wrong key, or no outbound HTTPS from `xtm-one` and `xtm-one-worker` to the provider. "You have reached your agentic execution quota for the current period" means the person's quota is spent: see **Settings** > **Usage and quotas** > **Quotas**. The trace of the run, in **Observability** > **Traces**, names the call that failed |
| Assignments do not run | `xtm-one-worker` is stopped or restarts, or Redis does not answer. A run whose owner is over quota fails with `Quota exceeded` |
| Knowledge base uploads fail | The proxy refuses the body: allow 50 MB or more (`client_max_body_size 50m;`). Otherwise check the `S3_*` values on both containers |
| PostgreSQL refuses connections | The pools of every process exceed `max_connections`. Count them again after a change of `WEB_WORKERS` or of the worker replicas |

## Ask for help

Give the version of XTM One (`docker compose images xtm-one xtm-one-worker`), the output of `docker compose ps` and of the readiness endpoint, and the lines of both logs around the problem, with the secrets removed. Customers of the Enterprise Edition ask Filigran support; others ask the [Filigran community on Slack](https://community.filigran.io).

## What's next

- [Configure XTM One](/deploy/configure/xtm-one/), for the variables this page names.
- [Upgrade XTM One](/deploy/upgrade/xtm-one/), before the next version.
- [Troubleshoot the XTM stack](/deploy/operate/troubleshoot/), for the other services.
- [Back up and restore](/deploy/operate/backup/).
