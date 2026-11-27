---
title: Develop XTM Composer
description: Build XTM Composer from source in Rust, run it against an OpenCTI you run locally, test it, and open a pull request.
doc_type: how-to
products:
  - composer
  - opencti
personas: [developer]
owner: docs-platform
last_reviewed: 2026-11-27
sidebar_position: 4
no_screenshot: "This page builds and runs XTM Composer in a terminal: XTM Composer has no interface of its own."
migrated_from:
  - opencti:/development/integration-manager/
---

This page sets up XTM Composer for development. You build it from source and run it against an OpenCTI you run locally. You watch it start connectors on your container engine, and you run the tests of the continuous integration (CI). XTM Composer is written in Rust. To install it for real use, see [Install XTM Composer](/deploy/integrations/install-composer/) instead; to understand what it does, read [XTM Composer architecture](/deploy/integrations/architecture/).

::diagram{id=develop-contribute-xtm-composer}

## Before you start

- Rust, through [rustup](https://rustup.rs). The crate uses the 2024 edition, so it needs Rust 1.85 or later. The CI tests with the current stable toolchain, and the release image builds with Rust 1.96.0.
- Git, OpenSSL to generate the key, and Docker or Podman to run the connectors.
- An OpenCTI running from source, as [Build and run from source](/develop/contribute/build-from-source/) describes, and the admin token of its `development.json`.
- To deploy connectors from the OpenCTI catalog, an Enterprise Edition license on that platform: without it, the catalog is read-only.
- For the FIPS 140-3 build only (`--features fips`): Go, CMake, Perl, clang and the Linux headers, which the AWS-LC FIPS module needs to build.

## Find your way in the code

| Path | What it holds |
| --- | --- |
| `src/main.rs` | Start: logger, RSA key, one orchestration loop and one ping loop per platform |
| `src/config/settings.rs` | The configuration structures and how they load |
| `src/api/opencti`, `src/api/openaev` | The clients of the platforms: GraphQL for OpenCTI, REST for OpenAEV |
| `src/engine/opencti.rs`, `src/engine/openaev.rs` | The loops that reconcile the integrations of a platform with the containers |
| `src/orchestrator/` | One folder per engine (`docker`, `podman`, `kubernetes`, `swarm`, `portainer`), each implementing the `Orchestrator` trait |
| `src/orchestrator/registry.rs` | The registry lookup of a moving tag: a `HEAD` on the manifest of the tag gives the digest it points to, without a pull |
| `api/opencti/opencti.graphql` | The OpenCTI schema that `build.rs` registers with `cynic` to type the queries. `api/opencti/README.md` lists the queries |
| `config/default.yaml` | The defaults of the main keys. Some keys are only commented examples there, and the `manager.debug` keys take their defaults from `src/config/settings.rs` |
| `dev/` | IntelliJ IDEA run configurations, and examples for Podman and Kubernetes |

A new container engine implements the `Orchestrator` trait of `src/orchestrator/mod.rs`: `get`, `list`, `start`, `stop`, `remove`, `refresh`, `deploy`, `logs` and `state_converter`, and `pull` when XTM Composer pulls the image itself. `src/orchestrator/composer.rs` calls `pull` before every deploy and refresh: when it fails, nothing is deployed and the running workload is kept. Docker, Podman, Swarm and Portainer implement it; Kubernetes keeps the default, which does nothing, because its nodes pull the image, and reports a pod stuck on its image through the `image_pull_error` of the container `get` returns.

Since 3.261127.0, XTM Composer redeploys a workload on a moving tag (`latest`, `rolling` or no tag) when the tag points to a new image. Two more methods of the trait serve it, both with a default:

- `resolve_image_digest` returns the digest the tag points to in the registry. The default returns `None`, which turns the lookups off for the engine. Every engine calls `registry::resolve_digest`, which returns `None` for a version tag or a digest.
- `refresh_image` redeploys the workload on that image, after `pull`. The default calls `refresh`, which recreates the container from the pulled tag. Kubernetes overrides it to pin the digest in the Deployment, so that the pod rolls.

`update_moving_image`, in `composer.rs`, compares the result with the `image_digest` of the container that `get` returns: an engine that leaves it empty is never looked up. That digest also goes to OpenAEV in the health report, with `last_termination`, how the previous container ended. `composer.rs` also writes that termination in the logs of the workload, when the container restarted or ended with an exit code other than 0. To show the last lines of a replaced container or pod, `logs` records each run in a `LogHistory` and prepends the previous lines with `with_previous_logs`, as Docker and Kubernetes do. Their `remove` drops the history of the workload with `forget`.

`workload_name` returns the name the engine knows the workload by: `container_name()` by default, the instance name in lower case with every other character replaced by `-`. An engine that caps names at a Request for Comments (RFC) 1123 label overrides it with `dns_label_name()`, as Kubernetes and Swarm do. That function keeps the `container_name()` that is already a valid label of 63 characters at most. Any other name is mapped to ASCII, cut to 54 characters, and followed by `-` and 8 hexadecimal characters of the SHA-256 of the full name. `composer.rs` detects a collision from the `xtm-workload-id` label of the container `get` returns under this name, and compares the names `list` returns with it to remove a workload left under an old name. Name the workload with the same function everywhere in your engine, or every cycle removes and deploys it again.

Tag every workload with `self.labels(workload)`, which writes the `xtm-manager`, `xtm-workload-id` and `xtm-platform` labels next to their legacy keys, `opencti-manager`, `opencti-connector-id` and `opencti-platform`. The environment of the workload already holds the configuration hash under `XTM_CONFIG_HASH` and `OPENCTI_CONFIG_HASH` (`config_hash_entries`). The legacy keys let an older XTM Composer recognize the workloads after a rollback. `list` runs one query per filter of `manager_label_filters` and merges the results with `dedup_containers`.

## Build and run it

:::steps
1. Clone the repository and build it:

   ```bash
   git clone https://github.com/FiligranHQ/xtm-composer.git
   cd xtm-composer
   cargo build
   ```

2. Generate the RSA private key. XTM Composer reads it in PKCS#8 PEM (`-----BEGIN PRIVATE KEY-----`), which `openssl genpkey` writes, and refuses to start with any other format. Make it 4096 bits: a smaller key loads, but the connector secrets OpenCTI encrypts for XTM Composer then fail to decrypt.

   ```bash
   openssl genpkey -algorithm RSA -out private_key_4096.pem -pkeyopt rsa_keygen_bits:4096
   ```

   Keep the key out of your commits.

3. Write `config/development.yaml`, ignored by Git. XTM Composer reads `config/default.yaml`, then `config/<COMPOSER_ENV>.yaml` over it, then the environment variables (`MANAGER__ID`, `OPENCTI__TOKEN`...). Your file only holds what differs from the defaults:

   ```yaml
   manager:
     id: 2b7f0c4e-9a1d-4e8b-b3f6-5c2d8e1a7f90
     credentials_key_filepath: ./private_key_4096.pem
     logger:
       level: debug
       format: pretty
       directory: true
       console: true
   opencti:
     enable: true
     url: http://localhost:4000
     token: a4c97db6-ba8a-4377-b6f3-77ca5b48928f
     daemon:
       selector: docker
       docker:
         network_mode: host
   ```

   - `manager.id`: generate it once (`python3 -c 'import uuid; print(uuid.uuid4())'`) and keep it. OpenCTI accepts one XTM Composer per platform and refuses the registration of a second identifier.
   - `opencti.token`: the `app:admin:token` of your OpenCTI `development.json`.
   - `opencti.daemon.selector`: `docker` or `podman`. The default is `kubernetes`.
   - `network_mode: host`, on Linux: XTM Composer gives each connector its own `opencti.url` as `OPENCTI_URL`, and OpenCTI gives it the RabbitMQ host of its configuration, `localhost` in development. On the host network, a connector reaches both. Without it, `localhost` in a container is the container itself.

4. Run it from the root of the clone, with the development configuration:

   ```bash
   COMPOSER_ENV=development cargo run
   ```

   The run configurations of `dev/` for IntelliJ IDEA and `.vscode/launch.json` for Visual Studio Code set `COMPOSER_ENV=development` too.
:::

:::verify
The console prints `Starting XTM composer`, then `Successfully loaded RSA private key (PKCS#8 format)` and `Manager registered`. In OpenCTI, at `http://localhost:3000`, deploy a connector from the catalog: XTM Composer starts its container, and `docker ps` lists it.
:::

To see more, set `manager.logger.level` to `trace`, and `manager.debug.show_env_vars` to `true` to print the environment variables of each connector XTM Composer starts (`show_sensitive_env_vars` also prints the secrets). Run with `RUST_BACKTRACE=1` to get the backtrace of a panic. Log files go to `logs/` next to the binary, `target/debug/logs/` for `cargo run`.

## Test it

:::steps
1. Run the unit tests, in the default mode and in FIPS mode, as the CI does:

   ```bash
   cargo test --workspace
   cargo test --workspace --features fips
   ```

2. Run the integration tests of the Docker and Podman orchestrators when you change them. They are ignored by default: they need a container engine and a registry with authentication, described by environment variables at the top of `src/orchestrator/docker/integration_tests.rs`. For Docker, with a local registry holding a test image:

   ```bash
   export XTM_IT_ENGINE=docker
   export XTM_IT_REGISTRY=localhost:5000
   export XTM_IT_REGISTRY_USERNAME=xtm-it
   export XTM_IT_REGISTRY_PASSWORD="$REGISTRY_PASSWORD"
   export XTM_IT_IMAGE=xtm-composer-it/connector:1.0
   cargo test orchestrator::docker::integration_tests -- --ignored --test-threads=1 --skip delegated
   ```

   `.github/workflows/integration.yml` shows how the CI starts the registry and builds the test image, on Docker, rootful Podman and rootless Podman.

3. Format and lint before you push:

   ```bash
   cargo fmt
   cargo clippy -- -D warnings
   ```
:::

When you change a query to OpenCTI, update `api/opencti/opencti.graphql` from the OpenCTI schema: `build.rs` compiles the queries against it, so a query that does not match the schema fails the build.

## Open a pull request

- The title follows Conventional Commits and ends with the issue: `type(scope): description (#issue)`. The `validate-pr-title` workflow checks it; the old `[composer]` prefix is discontinued.
- The pull request is linked to an issue: write `Fixes #212` (your issue number) in its description. Every commit is signed. Two more workflows check both.
- Releases are cut by Filigran: the version in `Cargo.toml` and `Cargo.lock` is bumped, and a GitHub release publishes the `filigran/xtm-composer` images.

:::example[A Docker fix tested on Velvet Harbor data]
A developer at Example CERT fixes how XTM Composer restarts a crashed connector on Docker. Their OpenCTI runs from source on the same Ubuntu laptop, in `~/opencti`, with an Enterprise Edition license. They clone XTM Composer next to it and write its configuration with the values of their platform:

```bash
cd ~/xtm-composer
openssl genpkey -algorithm RSA -out private_key_4096.pem -pkeyopt rsa_keygen_bits:4096
COMPOSER_ID=$(python3 -c 'import uuid; print(uuid.uuid4())')
OPENCTI_ADMIN_TOKEN=$(python3 -c "import json; print(json.load(open('../opencti/opencti-platform/opencti-graphql/config/development.json'))['app']['admin']['token'])")
cat > config/development.yaml <<EOF
manager:
  id: $COMPOSER_ID
  credentials_key_filepath: ./private_key_4096.pem
  logger:
    level: debug
    format: pretty
    directory: true
    console: true
opencti:
  enable: true
  url: http://localhost:4000
  token: $OPENCTI_ADMIN_TOKEN
  daemon:
    selector: docker
    docker:
      network_mode: host
EOF
COMPOSER_ENV=development cargo run
```

They deploy a connector from the OpenCTI catalog, stop its container with `docker kill` to reproduce the crash, and check the behavior of their fix in the console. Then they run `cargo test --workspace` and open a pull request titled `fix(docker): restart crashed connectors once (#212)`, with `Fixes #212` in its description.
:::

## What's next

- [XTM Composer configuration keys](/reference/composer/configuration/), every key and its variable.
- [Configure XTM Composer](/deploy/integrations/configure-composer/), the settings of a real deployment.
- [Build and run from source](/develop/contribute/build-from-source/), for the OpenCTI side of your tests.
