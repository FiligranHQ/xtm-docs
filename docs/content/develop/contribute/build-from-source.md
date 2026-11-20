---
title: Build and run from source
description: Clone OpenCTI or OpenAEV, start their development services, run the back end and the front end from source, and run the checks and tests of the CI.
doc_type: how-to
products:
  - opencti
  - openaev
personas: [developer]
owner: docs-platform
last_reviewed: 2026-11-01
sidebar_position: 3
no_screenshot: "This page runs commands in a terminal: the platform it starts shows its usual screens, documented in the product spaces."
migrated_from:
  - opencti:/development/platform/
  - openaev:/development/platform/
  - openaev:/development/build-from-source/
---

This page gets OpenCTI or OpenAEV running from its source code on your machine. You change the code, see the change at once, and run the checks of the continuous integration (CI) before your pull request. The services a platform needs (search engine, broker, storage) run in containers; the platform itself runs from source. Install the tools first: [Ubuntu](/develop/contribute/environment/ubuntu/) or [Windows](/develop/contribute/environment/windows/).

::diagram{id=develop-contribute-build-from-source}

## Before you start

- The tools of your environment page: Node.js, Yarn through Corepack, a container engine with Compose, and Python 3.12 for OpenCTI or Java 21 and Maven for OpenAEV.
- A GitHub account. To propose a change, fork the repository and open the pull request from your fork.
- This page follows the default branches: `master` for OpenCTI, `main` for OpenAEV. Commands run from the root of the clone unless a step says otherwise.

## Build and run OpenCTI

### What the repository holds

The root `package.json` declares a Yarn workspace (`opencti-platform/opencti-front`, `opencti-platform/opencti-graphql` and the custom ESLint rules) and drives every part with Nx (`nx run-many`).

| Folder | What it is | Language |
| --- | --- | --- |
| `opencti-platform/opencti-graphql` | The back end: GraphQL API, managers, Python bridge | TypeScript and JavaScript on Node.js, Python 3 |
| `opencti-platform/opencti-front` | The front end, served by Vite in development | React, TypeScript, Relay |
| `opencti-platform/opencti-dev` | The Compose file of the development services, IntelliJ IDEA run configurations | YAML |
| `opencti-worker` | The worker, which imports the bundles queued in RabbitMQ | Python |
| `client-python` | The Python client `pycti`, used by the worker and the connectors | Python |

### Clone and install the dependencies

:::steps
1. Clone the repository:

   ```bash
   git clone https://github.com/OpenCTI-Platform/opencti.git
   cd opencti
   ```

2. Create the Python virtual environment next to the clone, in `../.python-venv`:

   ```bash
   python3 -m venv ../.python-venv
   ```

   The repository expects it there. The `*:venv` scripts of the root `package.json` activate `../.python-venv` (or the folder in `VENV_PATH`). The back end adds `../.python-venv/lib/python3.12/site-packages` to the import path of its embedded Python, the default of `app:python_execution_venv`. With another Python version, set that key to your `site-packages` folder.

3. Install the JavaScript workspace, then every Python dependency inside the virtual environment:

   ```bash
   yarn install
   yarn deps:venv
   ```

   `yarn deps:venv` runs the `install` target of each part: `client-python` in editable mode, the Python libraries of the back end (`src/python/requirements.txt`) and those of the worker. Yarn runs no install script except for the packages `package.json` allows (`enableScripts: false`), and ignores packages published less than three days ago (`npmMinimalAgeGate`).
:::

### Configure the back end

The back end reads `config/default.json`, then a file named after its environment over it. `yarn start` builds it in development mode, so it reads `opencti-platform/opencti-graphql/config/development.json`. That file is ignored by Git: put your local values in it.

`default.json` already points to the development services on `localhost` with their credentials. Three values have no usable default, and the platform refuses to start without them:

| Key | Variable | What it needs |
| --- | --- | --- |
| `app:admin:password` | `APP__ADMIN__PASSWORD` | Any value except `ChangeMe` |
| `app:admin:token` | `APP__ADMIN__TOKEN` | A UUID, and not `ChangeMe`: it is the API token of the admin user |
| `app:encryption_key` | `APP__ENCRYPTION_KEY` | At least 32 bytes, base64-encoded: `openssl rand -base64 32` |

Write them with a short `development.json`: the other keys keep the values of `default.json`.

```bash
OPENCTI_ADMIN_TOKEN=$(python3 -c 'import uuid; print(uuid.uuid4())')
OPENCTI_ENCRYPTION_KEY=$(openssl rand -base64 32)
cat > opencti-platform/opencti-graphql/config/development.json <<EOF
{
  "app": {
    "admin": {
      "email": "admin@opencti.io",
      "password": "VelvetHarbor-dev-2026",
      "token": "$OPENCTI_ADMIN_TOKEN"
    },
    "encryption_key": "$OPENCTI_ENCRYPTION_KEY"
  }
}
EOF
```

Keep the token: the worker and your API calls use it. Every other key and its default is in the [OpenCTI configuration reference](/reference/opencti/configuration/).

### Start the development services

:::steps
1. Start the containers:

   ```bash
   cd opencti-platform/opencti-dev
   docker compose up -d
   cd ../..
   ```

   With Podman, run `podman compose up -d`.

2. Check what started. The ports match the defaults of `default.json`:

   | Service | Image | Ports |
   | --- | --- | --- |
   | Elasticsearch | `elasticsearch:8.19.23` | 9200 |
   | Kibana | `kibana:8.19.23` | 5601 |
   | Redis | `redis:8.6.7` | 6379 |
   | Redis Insight | `redislabs/redisinsight:3.8.0` | 5540 |
   | RabbitMQ, with its management interface | `rabbitmq:4.3.6-management` | 5672, 15672 |
   | Silo, S3-compatible storage | `pgsty/silo` | 9000, console 9001 |
   | MailDev, a fake SMTP server | `maildev/maildev:3.0.0` | SMTP 1025, web 1080 |
:::

Profiles start more services when you need them:

- `--profile opensearch`: OpenSearch on port 9201, instead of Elasticsearch.
- `--profile keycloak`: an OpenID Connect and SAML provider on port 9999.
- `--profile mokapi`: LDAP and CyberArk mocks.
- `--profile telemetry`: Jaeger, Prometheus, Grafana and Pyroscope.

`opencti-platform/opencti-dev/README.md` gives the configuration each one needs.

### Start the platform

The worker reads `opencti-worker/src/config.yml`, ignored by Git. Create it from the sample, pointed at your back end and its admin token:

```bash
cat > opencti-worker/src/config.yml <<EOF
opencti:
  url: 'http://localhost:4000'
  token: '$OPENCTI_ADMIN_TOKEN'
  json_logging: true
worker:
  log_level: 'info'
  telemetry_enabled: false
EOF
```

Then start everything from the root, inside the virtual environment. Pick how the back end reacts to your changes:

| Command | Back end | Front end | Worker |
| --- | --- | --- | --- |
| `yarn dev:venv` | Builds once (`yarn start`) and runs | Vite, reloads on change | Runs |
| `yarn dev:watch:venv` | Rebuilds and restarts on each change (`yarn dev`) | Vite, reloads on change | Runs |
| `yarn dev:watch-with-graphql:venv` | Same, and regenerates the GraphQL schema and types when a `.graphql` file changes | Same, and recompiles the Relay artifacts | Runs |

To run one part alone, activate the virtual environment with `. ../.python-venv/bin/activate`, then:

- Back end: `cd opencti-platform/opencti-graphql && yarn start`. In development, the back end also starts `python` child processes, a command the virtual environment provides.
- Front end: `cd opencti-platform/opencti-front && yarn start`. It listens on port 3000 (`FRONT_END_PORT`) and proxies `/graphql`, `/auth`, `/storage`, `/stream` and the other API paths to `http://localhost:4000` (`BACK_END_URL`).
- Worker: `cd opencti-worker/src && python worker.py`.

:::verify
The back end logs `[OPENCTI] API ready on port 4000`. Open `http://localhost:3000` and sign in with the email and password of `development.json`.
:::

### Check your changes

The CI runs these checks on each pull request. Run them before you push.

| Part | Command, from its folder | What the CI checks |
| --- | --- | --- |
| Back end | `yarn check-ts`, `yarn lint`, `yarn knip --dependencies` | Types, ESLint rules, declared dependencies |
| Front end | `yarn check-ts`, `yarn lint`, `yarn knip --dependencies`, `yarn verify-translation`, `yarn check:utility-classes` | Types, ESLint rules, dependencies, translation files, utility classes |
| Front end | `yarn test` | Unit tests (Vitest) |

The root `yarn lint` runs the `lint` target of every part: `check-ts`, `lint` and `verify-translation` for the front end, `lint` for the back end. Front-end changes also go through the design system gates of `AGENTS.md`: run `node fds-migration/scripts/check-mui-regression.mjs --explain` before writing interface code.

### Run the back-end tests

The back-end tests use Vitest and `opencti-platform/opencti-graphql/config/test.json`, which is in the repository. It runs the test platform on port 4010 and isolates its data with the `test` prefix: Elasticsearch indices, Redis namespace, RabbitMQ queues and the S3 bucket. The test suites use the same development services as your platform.

Run the commands from `opencti-platform/opencti-graphql`, with the virtual environment active:

| Command | What it runs | Needs the services |
| --- | --- | --- |
| `yarn test:ci-unit` | Unit tests, `tests/01-unit` | No |
| `yarn test:ci-integration` | Data injection and integration tests, `tests/02-*` and `tests/03-*` | Yes |
| `yarn test:ci-rules-and-others` | Data injection, rules, `tests/21-integration`, TAXII and deprecated tests | Yes |
| `yarn test:ci-integration-sync` | Integration and stream tests, and sync tests that need second platforms on port 4200 | Yes, and the CI stack |
| `yarn test:dev` | Every suite, after wiping and seeding the test platform: the longest run | Yes |

To work on one integration test, initialize the test platform and load its sample data once, then run only your file as often as you need. `test:dev:resume` skips the cleanup and the initialization, so it starts in seconds:

```bash
yarn test:dev:init
yarn test:dev:resume tests/03-integration/01-database/sorting-test.ts
```

Each `test:ci-*` command also takes one file, for example `yarn test:ci-unit tests/01-unit/utils/sorting-test.ts`.

## Build and run OpenAEV

OpenAEV is a multi-module Maven project (`openaev-api`, `openaev-model`, `openaev-framework` and others) with a React front end in `openaev-front`.

:::steps
1. Clone the repository:

   ```bash
   git clone https://github.com/OpenAEV-Platform/openaev.git
   cd openaev
   ```

2. Create the configuration of the `dev` Spring profile from its example. The copy is ignored by Git:

   ```bash
   cp openaev-api/src/main/resources/application-dev.properties.example \
      openaev-api/src/main/resources/application-dev.properties
   ```

3. Edit `application-dev.properties`:

   - Set `openaev.admin.token` to a UUID: `python3 -c 'import uuid; print(uuid.uuid4())'`.
   - The example turns on integrations that need external accounts (SAML, Caldera, Tanium, CrowdStrike, IMAP, OVH SMS). Set the `enable` or `enabled` keys of those you do not have to `false`, and set `openaev.listener.smtp.enabled=false`.
   - `openaev.rabbitmq.hostname` is `host.docker.internal`, so that injectors running in Docker reach RabbitMQ. Add `127.0.0.1 host.docker.internal` to `/etc/hosts` (Docker Desktop usually adds it on Windows), or set `localhost` if you run no injector in Docker. Keep the value alone on its line: Java properties read a trailing `# comment` as part of the value.

4. Start the four services OpenAEV needs. `.env.example` works as it is:

   ```bash
   cd openaev-dev
   cp .env.example .env
   docker compose up -d openaev-dev-pgsql openaev-dev-minio openaev-dev-elasticsearch openaev-dev-rabbitmq
   cd ..
   ```

   | Service | Ports |
   | --- | --- |
   | PostgreSQL | 5432 |
   | Silo, S3-compatible storage | 10000, console 10001 |
   | Elasticsearch | 9200, 9300 |
   | RabbitMQ | 5672, 15672 |

5. Build every module, then start the back end with the `dev` profile. It listens on port 8080:

   ```bash
   mvn clean install -DskipTests
   mvn spring-boot:run -pl openaev-api -Dspring-boot.run.profiles=dev
   ```

6. In another terminal, install and start the front end. It listens on port 3001 and proxies the API calls to port 8080:

   ```bash
   cd openaev-front
   yarn install
   yarn start
   ```
:::

:::verify
Open `http://localhost:3001` and sign in with `openaev.admin.email` and `openaev.admin.password` of `application-dev.properties` (`admin@openaev.io` and `admin` in the example).
:::

OpenAEV and OpenCTI both use port 9200 for Elasticsearch: run one development stack at a time, or change a port. Before you push, run the checks of the CI:

| Part | Commands |
| --- | --- |
| Back end | `mvn spotless:apply` (Google Java Format), `mvn test` with the services running, or `mvn test -pl openaev-api -Dtest=MyTestClass` for one class |
| Front end, from `openaev-front` | `yarn lint` (no warning allowed), `yarn check-ts`, `yarn depcruise`, `yarn knip --dependencies`, `yarn test` |

`openaev-dev/README.md` lists the optional services (test database, Kibana, pgAdmin, OpenSearch) and the `setup-auto-db` script, which starts a PostgreSQL container per branch.

## Open a pull request

OpenCTI and OpenAEV check the same conventions on every pull request:

- The title follows Conventional Commits and ends with the issue: `type(scope): description (#issue)`. Types: `feat`, `fix`, `chore`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `revert`. The description starts with a lowercase letter and has no final period. The old `[backend]` and `[frontend]` prefixes are discontinued.
- The pull request is linked to an issue.
- Every commit is signed. See [Signing commits](https://docs.github.com/en/authentication/managing-commit-signature-verification/signing-commits) in the GitHub documentation.

:::example[Fix the report list for Velvet Harbor]
A developer at Example CERT fixes a bug in how OpenCTI sorts the reports on Velvet Harbor, tracked in issue 14210. On Ubuntu, with the tools installed:

```bash
git clone https://github.com/OpenCTI-Platform/opencti.git && cd opencti
python3 -m venv ../.python-venv
yarn install && yarn deps:venv
OPENCTI_ADMIN_TOKEN=$(python3 -c 'import uuid; print(uuid.uuid4())')
OPENCTI_ENCRYPTION_KEY=$(openssl rand -base64 32)
cat > opencti-platform/opencti-graphql/config/development.json <<EOF
{"app": {"admin": {"email": "admin@opencti.io", "password": "VelvetHarbor-dev-2026", "token": "$OPENCTI_ADMIN_TOKEN"}, "encryption_key": "$OPENCTI_ENCRYPTION_KEY"}}
EOF
(cd opencti-platform/opencti-dev && docker compose up -d)
yarn dev:watch:venv
```

They change the sort in the back end, which restarts at each save, and check the list at `http://localhost:3000`. They run `yarn check-ts`, `yarn lint` and `yarn test:ci-unit` in `opencti-platform/opencti-graphql`. Then they open a pull request titled `fix(backend): sort reports by publication date (#14210)`, with signed commits.
:::

## What's next

- [Build a connector](/develop/build/connector/), to test an integration against the platform you run.
- [The Python client](/develop/build/python-client/), the `pycti` library of `client-python`.
- [Develop XTM Composer](/develop/contribute/xtm-composer/), for the component that runs integrations as containers.
- [Translations](/develop/contribute/translations/), to add or fix the labels of the interface.
