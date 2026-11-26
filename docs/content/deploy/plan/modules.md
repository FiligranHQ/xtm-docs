---
title: What each XTM module brings
description: What each module of the XTM stack brings, the services it runs, the shared services it uses and the settings it needs in .env. The whole stack is always deployed.
doc_type: explanation
products:
  - suite
personas: [platform-engineer, evaluator]
owner: docs-platform
last_reviewed: 2026-10-31
sidebar_position: 2
---

The XTM stack is always deployed whole: XTM One, OpenCTI, OpenAEV, OpenCRQ, their default integrations and XTM Composer start together, with the shared services they use. No module is optional. This page says what each module adds, which services it runs, which shared services it uses and what it needs in `.env`.

::diagram{id=deploy-plan-modules}

## The whole stack

::stack-builder

## What starts

In [FiligranHQ/xtm-docker](https://github.com/FiligranHQ/xtm-docker), `docker-compose.yml` includes one file per module, in `modules/`, and one file per shared service, in `infra/`. `docker compose pull`, then `docker compose up -d`, starts the 32 services of these files. No setting leaves a module out.

| Module | Compose file | Services it runs | Shared services it uses |
| --- | --- | --- | --- |
| XTM One | `modules/xtm-one.yml` | `xtm-one`, `xtm-one-worker`, and the `xtm-one-db-init` job | Redis, S3 storage, PostgreSQL |
| OpenCTI | `modules/opencti.yml` | `opencti`, `worker` (3 replicas), and 10 default connectors | Elasticsearch, RabbitMQ, Redis, S3 storage |
| OpenAEV | `modules/openaev.yml` | `openaev`, 4 default collectors and 2 default injectors | Elasticsearch, RabbitMQ, S3 storage, PostgreSQL |
| OpenCRQ | `modules/opencrq.yml` | `opencrq`, and the `opencrq-db-init` and `opencrq-bucket-init` jobs | S3 storage, PostgreSQL |
| XTM Composer | `modules/xtm-composer.yml` | `xtm-composer`, and the `rsa-key-generator` job | None |

:::verify[Check what will start]
```bash
docker compose config --services
```
It lists the 32 services of the stack, among them `xtm-one`, `opencti`, `openaev`, `opencrq` and `xtm-composer`.
:::

## XTM One

XTM One is the AI layer at the center of the platform. Its AI agents work across the platforms connected to it. It needs these values in `.env`:

| Variable | What to put |
| --- | --- |
| `XTM_ONE_ADMIN_EMAIL` | The admin email of the platforms you connect, so that XTM One finds their user |
| `XTM_ONE_ADMIN_PASSWORD` | The password of the XTM One admin |
| `XTM_ONE_SECRET_KEY` | A long random string, such as the output of `openssl rand -hex 32`. It signs sessions and tokens |
| `XTM_ONE_POSTGRES_USER`, `XTM_ONE_POSTGRES_PASSWORD` | The PostgreSQL role `xtm-one-db-init` creates. It must differ from `POSTGRES_USER` |
| `PLATFORM_REGISTRATION_TOKEN` | A long random string, the same for every platform connected to this XTM One |

XTM One images are published for `linux/amd64` only. On an ARM machine, such as a Mac with an Apple chip, they run through emulation: see [Requirements](/deploy/plan/requirements/).

## OpenCTI

The threat intelligence platform: it collects, structures and shares knowledge about threats. `modules/opencti.yml` runs the platform and three worker replicas.

| Variable | What to put |
| --- | --- |
| `OPENCTI_ADMIN_EMAIL`, `OPENCTI_ADMIN_PASSWORD` | The first admin account |
| `OPENCTI_ADMIN_TOKEN` | A UUIDv4, such as the output of `uuidgen`. The platform does not start with another format |
| `OPENCTI_ENCRYPTION_KEY` | A 32-byte base64 key, the output of `openssl rand -base64 32`, not a UUID |
| `OPENCTI_HEALTHCHECK_ACCESS_KEY` | A random string that protects the health endpoint |
| `OPENCTI_EXTERNAL_SCHEME`, `OPENCTI_HOST`, `OPENCTI_PORT` | The public URL of OpenCTI: `http`, `localhost` and `8080` by default |

### Default OpenCTI connectors

Ten connectors run with OpenCTI. Each one reads its ID from a `CONNECTOR_*_ID` variable, a UUIDv4 that `.env.sample` already sets.

| Connector | Service | What it does |
| --- | --- | --- |
| Export File STIX | `connector-export-file-stix` | Exports data as STIX 2.1 |
| Export File CSV | `connector-export-file-csv` | Exports data as CSV |
| Export File TXT | `connector-export-file-txt` | Exports data as plain text |
| Import File STIX | `connector-import-file-stix` | Imports STIX 2.1 bundles |
| Import Document | `connector-import-document` | Imports PDF, HTML and text documents |
| Import File YARA | `connector-import-file-yara` | Imports YARA rules |
| Analysis | `connector-analysis` | Analyzes PDF, HTML and text documents |
| Import External Reference | `connector-import-external-reference` | Imports external references |
| OpenCTI Datasets | `connector-opencti` | Loads the default marking definitions, identities and locations |
| MITRE ATT&CK | `connector-mitre` | Loads the MITRE ATT&CK framework |

## OpenAEV

The exposure validation platform: it runs scenarios and simulations to test how your defenses respond. `modules/openaev.yml` runs the platform. It stores its data in the `openaev` database of the shared PostgreSQL.

| Variable | What to put |
| --- | --- |
| `OPENAEV_ADMIN_EMAIL`, `OPENAEV_ADMIN_PASSWORD` | The first admin account |
| `OPENAEV_ADMIN_TOKEN` | A UUIDv4, such as the output of `uuidgen` |
| `OPENAEV_HEALTHCHECK_KEY` | A random string that protects the health endpoint |
| `OPENAEV_ADMIN_ENCRYPTION_KEY`, `OPENAEV_ADMIN_ENCRYPTION_SALT` | A random key and a salt of at least 8 bytes |
| `OPENAEV_EXTERNAL_SCHEME`, `OPENAEV_HOST`, `OPENAEV_PORT` | The public URL of OpenAEV: `http`, `localhost` and `8081` by default |
| `SMTP_*` | The mail server OpenAEV sends emails through |
| `OPENAEV_MAIL_IMAP_ENABLED` and `IMAP_*` | `true` and the IMAP mailbox OpenAEV reads, or `false` (the default in `.env.sample`) |

The host port of OpenAEV is written in `modules/openaev.yml` (`"8081:8080"`), not in `.env`: to move it, replace the mapping in `docker-compose.override.yml` and change `OPENAEV_PORT` together. See [Network and ports](/deploy/plan/network/#ports-published-on-the-host).

### Default OpenAEV collectors and injectors

Four collectors and two injectors run with OpenAEV. Each one reads its ID from a `COLLECTOR_*_ID` or `INJECTOR_*_ID` variable that `.env.sample` already sets.

| Integration | Kind | Service | What it brings |
| --- | --- | --- | --- |
| MITRE ATT&CK | Collector | `collector-mitre-attack` | Attack techniques and procedures |
| OpenAEV Datasets | Collector | `collector-openaev` | Default datasets |
| Atomic Red Team | Collector | `collector-atomic-red-team` | Atomic Red Team tests |
| CVE by NVD NIST | Collector | `collector-nvd-nist-cve` | CVE data from the National Vulnerability Database (NVD) |
| Nmap | Injector | `injector-nmap` | Network scans |
| Nuclei | Injector | `injector-nuclei` | Vulnerability scans |

The NVD collector reads an NVD API key from `COLLECTOR_NVD_NIST_CVE_API_KEY`. `.env.sample` leaves it empty and marks it as recommended.

## XTM Composer

XTM Composer deploys and runs integrations from the interface of OpenCTI and OpenAEV, instead of declaring each one in the Compose files. One XTM Composer manages both platforms: it starts once OpenCTI and OpenAEV are healthy.

| Variable | What to put |
| --- | --- |
| `XTM_COMPOSER_ID` | Its ID, a UUIDv4 that `.env.sample` already sets |

XTM Composer mounts the Docker socket of the host (`/var/run/docker.sock`) to start the integration containers. See [XTM Composer](/deploy/integrations/composer/).

## OpenCRQ

OpenCRQ quantifies cyber risk in money, from the threats of OpenCTI and the validation results of OpenAEV, and registers with XTM One, without which it does not start. It connects to the shared PostgreSQL as `POSTGRES_USER`, like OpenAEV, and needs these values in `.env`:

| Variable | What to put |
| --- | --- |
| `OPENCRQ_EXTERNAL_SCHEME`, `OPENCRQ_HOST`, `OPENCRQ_PORT` | The public URL of OpenCRQ, also its identity towards XTM One. The host publishes it on port 8082 whatever `OPENCRQ_PORT` says |

See [Finish the setup of OpenCRQ](/deploy/configure/opencrq/#finish-the-setup-after-the-install) for the first start and its first administrator.

## What's next

- [The XTM stack](/deploy/plan/architecture/): how the modules and shared services fit together.
- [Requirements](/deploy/plan/requirements/) and [Sizing](/deploy/plan/sizing/) for the whole stack.
- [Install with Docker Compose](/deploy/install/docker-compose/), or [run the platform locally](/deploy/quick-start/) to try it first.
- [Editions and licensing](/start/editions/).
