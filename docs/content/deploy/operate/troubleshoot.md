---
title: Troubleshoot the XTM stack
description: Find the logs of each part of the XTM stack, raise their level, and fix the common startup, ingestion and Elasticsearch errors.
doc_type: how-to
products:
  - suite
personas: [platform-engineer]
owner: docs-platform
last_reviewed: 2026-10-31
migrated_from:
  - opencti:/deployment/advanced/troubleshooting/
---

This page helps you find why a part of the XTM stack fails: where its logs are, how to make them more detailed, and what the common errors mean, from a stack that does not start to data that OpenCTI rejects. For problems with integrations that XTM Composer runs, see [Troubleshoot integrations](/deploy/integrations/troubleshoot/).

::diagram{id=deploy-operate-troubleshoot}

## Find the logs

Every service of xtm-docker writes its logs to the console of its container. From the `xtm-docker` folder:

```bash
docker compose ps
docker compose logs --tail 200 opencti
docker compose logs -f openaev xtm-one
```

`docker compose ps` shows the services that run and their health; `docker compose logs` takes the name of a service, such as `opencti`, `worker`, `openaev`, `xtm-one`, `xtm-one-worker`, `xtm-composer` or `connector-mitre`. The platforms, workers and connectors write JSON lines.

## Raise the log level

In production, keep the platforms at `error` or `warn` level. To investigate, raise the level of one service, then lower it again: debug logs are large. Set the variable in `docker-compose.override.yml`, as [Change the stack without editing docker-compose.yml](/deploy/install/docker-compose/#change-the-stack-without-editing-docker-compose-yml) describes, or in `.env` where the table says so, then run `docker compose up -d <service>`.

::::tabs
:::tab[OpenCTI]
| Part | Variable | Values |
| --- | --- | --- |
| Platform | `APP__APP_LOGS__LOGS_LEVEL` | `error`, `warn`, `info`, `debug`. xtm-docker sets `error` |
| Workers | `WORKER_LOG_LEVEL` | `error`, `warning`, `info` (default), `debug` |
| Connectors | `CONNECTOR_LOG_LEVEL` | `error`, `warning`, `info`, `debug` |

`APP__APP_LOGS__LOGS_CONSOLE=true` writes the platform logs to the console, the default. Workers and connectors write JSON while `OPENCTI_JSON_LOGGING` is `true`, the default.

```yaml
services:
  opencti:
    environment:
      - APP__APP_LOGS__LOGS_LEVEL=info
```
:::
:::tab[OpenAEV]
OpenAEV takes the logging settings of Spring Boot. To log its own code at debug level:

```yaml
services:
  openaev:
    environment:
      - LOGGING_LEVEL_IO_OPENAEV=debug
```
:::
:::tab[XTM One]
`XTM_ONE_LOG_LEVEL` in `.env` sets the level of `xtm-one` and `xtm-one-worker` (default `info`), and `XTM_ONE_LOG_FORMAT` their format (default `json`).
:::
::::

## Control what OpenCTI logs

Besides the console, the platform writes two log files: `error.log`, errors only, and `opencti.log`, every entry at the configured level. They rotate daily. In a container, they stay inside it and are lost when it is recreated: collect the console output instead, and turn the files off.

| Key | Default | What it does |
| --- | --- | --- |
| `app:app_logs:logs_files` | `true` | Writes the log files |
| `app:app_logs:logs_directory` | `./logs` | Folder of the files, relative to the folder the platform runs from |
| `app:app_logs:logs_max_files` | `7` | Daily files kept for each log |

Each entry carries metadata, such as the data of an error. OpenCTI cuts it to keep log lines small:

| Key | Default | What it does |
| --- | --- | --- |
| `app:app_logs:control:max_string_size` | `5000` | Longest string, in characters. A longer one is cut and ends with `...` |
| `app:app_logs:control:max_array_size` | `50` | Elements kept from a list |
| `app:app_logs:control:max_depth_size` | `10` | Nesting depth. Deeper objects are dropped |
| `app:app_logs:control:max_depth_keys` | `100` | Keys written for one entry. The next ones are dropped |

Two settings help an investigation, and are off by default because of what they write:

- `app:app_logs:extended_error_message` adds the input data to error logs, and the GraphQL query and its variables to the logs of failed API calls. The logs then hold sensitive data.
- `app:performance_logger` logs every successful API call at `info` level, with its duration, size and memory use. A failed call keeps its own line, at the level of its error. It writes a lot of logs.

For example, to investigate a failing import from the console only:

```yaml
services:
  opencti:
    environment:
      - APP__APP_LOGS__LOGS_LEVEL=info
      - APP__APP_LOGS__LOGS_FILES=false
      - APP__APP_LOGS__EXTENDED_ERROR_MESSAGE=true
```

Set `APP__APP_LOGS__EXTENDED_ERROR_MESSAGE` back to `false` once you have found the cause. The other log keys are in the [Errors](/reference/opencti/configuration/#errors), [Logging](/reference/opencti/configuration/#logging) and [Audit](/reference/opencti/configuration/#audit) sections of the configuration reference.

:::only{opencti=">=7.261127"}
The platform logs the errors of a GraphQL API call by what they mean. A refused authentication or access writes no line in these logs. An input error of the caller is logged at `info` level, a functional error, such as `LOCK_ERROR` or `MISSING_REFERENCE_ERROR`, at `warn`, and any other error at `error`. A client that leaves a live stream while the platform processes it writes `[STREAM] Client disconnected during processing` at `info` level, not a processing error. With the `error` level xtm-docker sets, the lock timeouts of API calls, such as those of the workers, no longer show in the platform logs: raise the level to `warn` to see them.
:::

::::only{opencti=">=7.261127"}
## Find the error behind a request

When a user or a script reports an error of OpenCTI, its request id leads you to the full error in the platform logs. OpenCTI answers every request with an `X-Request-ID` header, and writes the same id, as `request_id`, in the log line of the error:

- An error of the GraphQL API carries the id in its `extensions`, as `request_id`.
- An error of the other endpoints, such as a file download or a CSV feed, carries it in its body, as `request_id`. The TAXII endpoints name it `error_id`.

A request that brings its own `X-Request-ID`, such as one your reverse proxy sets, keeps it when it holds 1 to 128 letters, digits, dots, underscores or hyphens; any other value is replaced by a random UUID: see [Follow a request from the proxy to OpenCTI](/deploy/install/reverse-proxy/#follow-a-request-from-the-proxy-to-opencti).

Outside the GraphQL API, a platform failure answers only `An unexpected error occurred`: its cause can name hosts, indices or paths, so it stays in the log. The errors written for the caller, such as a refused access, an invalid input or an element not found, keep their message.

The GraphQL API returns the message of every error by default. `app:graphql:show_error_details` (`APP__GRAPHQL__SHOW_ERROR_DETAILS`, default `true`) set to `false` answers its platform failures the same way: `An unexpected error occurred`, with the error code and the request id only. The errors of the caller keep their message, without the cause of the error. Users and scripts then get that generic message: ask them for the request id.

:::example[Find the error an analyst reports]
An analyst of Example CERT reports that the GraphQL API answered `An unexpected error occurred`, with the `request_id` `4f1c2a9e-7b3d-4e8a-9c61-0d5f2b8e3a17`. The platform hides the details of its failures from the API:

```yaml
services:
  opencti:
    environment:
      - APP__GRAPHQL__SHOW_ERROR_DETAILS=false
```

From the `xtm-docker` folder, find the error with its id:

```bash
docker compose logs opencti | grep '4f1c2a9e-7b3d-4e8a-9c61-0d5f2b8e3a17'
```

The JSON line holds the message of the error, its `cause`, and the `operation` and `user_id` of the call.
:::
::::

## The stack does not start

| Symptom | Cause and fix |
| --- | --- |
| Elasticsearch stops or restarts in a loop | Its logs say why. On Linux, it often needs more virtual memory areas: run `sudo sysctl -w vm.max_map_count=262144` on the host, and add `vm.max_map_count=262144` to `/etc/sysctl.conf` to keep it after a reboot |
| OpenCTI stops at start with `You need to configure the environment vars`, `Email must be a valid email address` or `Token must be a valid UUID` | The bootstrap admin of `.env` is incomplete or invalid. `OPENCTI_ADMIN_EMAIL` must be an email address and `OPENCTI_ADMIN_TOKEN` a UUID, such as the output of `uuidgen`. Neither the password nor the token may be empty or `ChangeMe`. See [Sign in the first time and keep an emergency account](/deploy/configure/first-access/#the-bootstrap-admins) |
| A platform stays `starting` or `unhealthy` | One of its dependencies is not healthy: check `docker compose ps`, then the logs of the platform and of the dependency. Check that every variable of `.env` has a value |
| OpenAEV stops at start with `FATAL: password authentication failed for user` | The PostgreSQL credentials do not match. PostgreSQL takes `POSTGRES_USER` and `POSTGRES_PASSWORD` only when it creates its data volume, `pgsqlshareddata`: changing them in `.env` afterwards does not change the existing user. Set them back to the values the volume was created with, or change the password inside PostgreSQL with `ALTER USER`. On a manual installation, `spring.datasource.username` and `spring.datasource.password` must be those of the PostgreSQL user |
| `xtm-one-db-init` fails | Its logs say why: `XTM_ONE_POSTGRES_USER` and `XTM_ONE_POSTGRES_PASSWORD` must be set, and `XTM_ONE_POSTGRES_USER` must differ from `POSTGRES_USER` |
| The platforms run but do not see each other | Check the tokens. `PLATFORM_REGISTRATION_TOKEN` must be the same for XTM One, OpenCTI and OpenAEV, also when they run in separate deployments |

## OpenCTI ingestion errors

OpenCTI reports the errors of the bundles it could not write in the works of each connector, with an error code.

### Technical errors

| Error | Meaning and fix |
| --- | --- |
| `MISSING_REFERENCE_ERROR`, `Element(s) not found` | An object refers to another one that does not exist, such as a relationship whose target is missing. It is raised after several retries, in case the missing object arrives in the meantime. It usually comes from a connector that produces inconsistent STIX 2.1 bundles |
| `Cant upsert inferred entity. Too many entities resolved` | An inference rule produced an entity that matches several existing entities, such as two entities that share a name or an alias, and OpenCTI does not choose which one to update. Data from connectors does not raise it: OpenCTI updates one of the matching entities, and merges them only when the data asks for it (`update: true`) or when its STIX ID points to another of them. Merge the duplicates in the platform |
| `LOCK_ERROR`, `Execution timeout, too many concurrent call on the same entities` | Several workers changed the same entities at the same time and waited too long for each other. It depends on the throughput of Elasticsearch, the number of workers, the dependencies between the data and the merges. Lower the number of workers, then find the balance between workers, connectors and Elasticsearch sizing. See [Scale](/deploy/operate/scale/) |

### Data errors

| Code | Meaning and fix |
| --- | --- |
| `INCORRECT_INDICATOR_FORMAT` | The pattern of an Indicator is not valid, such as a malformed YARA rule. Fix it at the source |
| `INCORRECT_OBSERVABLE_FORMAT` | The value of an observable is not valid for its type, such as an IPv4 address that is not one |
| `INDICATOR_PATTERN_EXCLUDED` | The pattern of an Indicator contains a value of one of your exclusion lists, so the Indicator is rejected on purpose; the rest of the bundle is imported. The error gives the value and the list. If the value should not be excluded, edit the list in :menu[Settings > Customization > Exclusion lists] |

## Elasticsearch errors

| Error | Meaning and fix |
| --- | --- |
| `TOO_MANY_REQUESTS/12/disk usage exceeded flood-stage watermark` | The disk of Elasticsearch is almost full, and Elasticsearch blocks writes to protect itself. Free or add disk space: Elasticsearch lifts the block once usage falls back under its high watermark |
| `Engine execution fail, some shards are not available` | Some shards of an index cannot be read. Check the health of the cluster with `docker compose exec elasticsearch curl -s 'http://localhost:9200/_cluster/health?pretty'` |

To look into the data that OpenCTI stores, connect Kibana or OpenSearch Dashboards to your cluster.

## Ask for help

Generate a [support package](/deploy/operate/support-package/) of OpenCTI, then ask on the [Filigran community Slack](https://community.filigran.io) or open an issue for [OpenCTI](https://github.com/OpenCTI-Platform/opencti/issues) or [OpenAEV](https://github.com/OpenAEV-Platform/openaev/issues).

## What's next

- [Check that the platforms are healthy](/deploy/operate/health-probes/), to know which dependency is down.
- [Observability metrics](/deploy/operate/metrics/), to see a problem coming.
- [Collect logs for support](/deploy/operate/support-package/), to send the logs of every OpenCTI node at once.
- [Errors](/reference/opencti/errors/): the functional errors OpenCTI returns, by area, and what to do for the most common ones.
