---
title: Install XTM Composer
description: Install XTM Composer with xtm-docker, Docker, Podman or Kubernetes, with its RSA key, its platform tokens and access to the container engine.
doc_type: how-to
products:
  - composer
  - opencti
  - openaev
personas: [platform-engineer]
owner: docs-platform
last_reviewed: 2026-10-31
edition: ee
migrated_from:
  - opencti:/deployment/integration-manager/installation/
  - openaev:/deployment/ecosystem/integration-manager/installation/
---

This page installs XTM Composer next to OpenCTI, OpenAEV or both, on the container engine that will run your integrations. In the XTM stack, xtm-docker already runs XTM Composer for both platforms: see [Deploy a first integration with XTM Composer](/deploy/integrations/quick-start/). Read [XTM Composer](/deploy/integrations/composer/) first if you are new to it.

::diagram{id=deploy-integrations-install-composer}

## Before you start

- OpenCTI, OpenAEV or both, running and reachable from the machine or cluster where XTM Composer and its integrations will run.
- An API token for each platform. xtm-docker uses the admin token of each platform (`OPENCTI_ADMIN_TOKEN`, `OPENAEV_ADMIN_TOKEN`).
- The container engine that will run the integrations: Docker, Podman or Kubernetes. For Docker Swarm and Portainer, install with Docker below, then set the selector in [Configure XTM Composer](/deploy/integrations/configure-composer/#the-container-engine).
- OpenSSL, to generate the key, and Python 3, to generate an identifier.

## What every installation has

| Part | Value |
| --- | --- |
| Image | `filigran/xtm-composer`, for `linux/amd64` and `linux/arm64`. Tags: `latest`, each release such as `3.261121.0`, and a FIPS build tagged `latest-fips` or `<release>-fips`. The FIPS build deploys the `-fips` images of the integrations: see `manager.image_tag_suffix` in [Configure XTM Composer](/deploy/integrations/configure-composer/#xtm-composer-itself) |
| RSA private key | 4096 bits, PKCS#8 PEM (`-----BEGIN PRIVATE KEY-----`), given with `MANAGER__CREDENTIALS_KEY_FILEPATH` or `MANAGER__CREDENTIALS_KEY` |
| Identifier | `MANAGER__ID`, the same at every restart. xtm-docker uses a UUID |
| Platforms | `OPENCTI__ENABLE`, `OPENCTI__URL`, `OPENCTI__TOKEN`, and the same with `OPENAEV__`. OpenCTI is enabled by default, OpenAEV is not |
| Container engine | `OPENCTI__DAEMON__SELECTOR` and `OPENAEV__DAEMON__SELECTOR`: `docker`, `podman`, `kubernetes`, `swarm` or `portainer`. The default is `kubernetes` |

Generate the key with `openssl genpkey`: it writes PKCS#8. Older OpenSSL versions of `openssl genrsa` write PKCS#1 (`BEGIN RSA PRIVATE KEY`), which XTM Composer refuses.

```bash
openssl genpkey -algorithm RSA -out private_key.pem -pkeyopt rsa_keygen_bits:4096
```

XTM Composer reads its settings from environment variables, which this page uses, or from a YAML file: see [Configure XTM Composer](/deploy/integrations/configure-composer/).

## Install

::::::tabs
:::::tab[XTM stack]
XTM Composer is part of the stack: there is nothing to add. `docker compose pull` and `docker compose up -d` start it with the rest, from `modules/xtm-composer.yml`, and it serves both OpenCTI and OpenAEV. The `rsa-key-generator` service creates the key in the `rsakeys` volume and `XTM_COMPOSER_ID` in `.env` is the identifier.

[Deploy a first integration with XTM Composer](/deploy/integrations/quick-start/) gives every command and its check.
:::::
:::::tab[Docker]
XTM Composer drives the Docker daemon through its socket and starts the integrations as sibling containers, attached to the network you name.

::::steps
1. Create a folder with the key.

   ```bash
   mkdir -p xtm-composer/keys && cd xtm-composer
   openssl genpkey -algorithm RSA -out keys/private_key.pem -pkeyopt rsa_keygen_bits:4096
   ```

2. Set the platforms to serve and the Docker network the integrations must join to reach them. Leave the URL of a platform empty to leave it out.

   ```bash
   export OPENCTI_URL=https://opencti.example.com OPENCTI_TOKEN=your-opencti-token
   export OPENAEV_URL= OPENAEV_TOKEN=
   export PLATFORM_NETWORK=xtm_default
   ```

   :::verify
   ```bash
   docker network inspect "$PLATFORM_NETWORK" --format '{{.Name}}'
   ```
   It prints the network name. `docker network ls` lists the networks if it does not exist.
   :::

3. Write the configuration once, with a new identifier. Keep `composer.env`: the identifier must not change.

   ```bash
   on() { [ -n "$1" ] && echo true || echo false; }
   cat > composer.env <<EOF
   MANAGER__ID=$(python3 -c 'import uuid; print(uuid.uuid4())')
   MANAGER__NAME=XTM Composer
   MANAGER__CREDENTIALS_KEY_FILEPATH=/keys/private_key.pem
   OPENCTI__ENABLE=$(on "$OPENCTI_URL")
   OPENCTI__URL=$OPENCTI_URL
   OPENCTI__TOKEN=$OPENCTI_TOKEN
   OPENCTI__DAEMON__SELECTOR=docker
   OPENCTI__DAEMON__DOCKER__NETWORK_MODE=$PLATFORM_NETWORK
   OPENAEV__ENABLE=$(on "$OPENAEV_URL")
   OPENAEV__URL=$OPENAEV_URL
   OPENAEV__TOKEN=$OPENAEV_TOKEN
   OPENAEV__DAEMON__SELECTOR=docker
   OPENAEV__DAEMON__DOCKER__NETWORK_MODE=$PLATFORM_NETWORK
   EOF
   chmod 600 composer.env
   ```

4. Start XTM Composer.

   ```bash
   docker run -d --name xtm-composer --restart always \
     --env-file composer.env \
     --network "$PLATFORM_NETWORK" \
     -v /var/run/docker.sock:/var/run/docker.sock \
     -v "$PWD/keys:/keys:ro" \
     filigran/xtm-composer:latest
   ```

   :::verify
   ```bash
   docker logs xtm-composer 2>&1 | grep -E 'Successfully loaded RSA private key|Manager registered'
   ```
   It shows the key loaded and one `Manager registered` line per platform. Registration is retried every minute.
   :::
::::
:::::
:::::tab[Podman]
XTM Composer drives Podman through its API socket, rootful or rootless, with the `podman` selector.

::::steps
1. Turn on the Podman socket. Rootful, as root:

   ```bash
   sudo systemctl enable --now podman.socket
   ```

   Rootless, as the user that will own the integrations, and keep its containers running after logout:

   ```bash
   systemctl --user enable --now podman.socket
   sudo loginctl enable-linger "$USER"
   ```

2. Create the key and the configuration as in the Docker tab, steps 1 to 3, with three changes. Write `podman` instead of `docker` in the `OPENCTI__DAEMON__SELECTOR` and `OPENAEV__DAEMON__SELECTOR` lines, and `PODMAN` instead of `DOCKER` in the two `NETWORK_MODE` lines. Check the network with `podman network inspect`.

3. Start XTM Composer with the socket mounted at `/run/podman/podman.sock`, where it finds it without configuration. For a rootless Podman, run this as the same user and mount `/run/user/$(id -u)/podman/podman.sock` instead.

   ```bash
   podman run -d --name xtm-composer --restart always \
     --env-file composer.env \
     --security-opt label=disable \
     -v /run/podman/podman.sock:/run/podman/podman.sock \
     -v "$PWD/keys:/keys:ro" \
     docker.io/filigran/xtm-composer:latest
   ```

   `label=disable` lets the container use the socket on SELinux hosts.

   :::verify
   ```bash
   podman logs xtm-composer 2>&1 | grep -E 'Connected to Podman|Manager registered'
   ```
   It shows `Connected to Podman`, with the socket and the rootful or rootless mode, and one `Manager registered` line per platform.
   :::
::::
:::::
:::::tab[Kubernetes]
XTM Composer creates one Deployment per integration, in its own namespace, with a service account that may manage Deployments there. The integrations must reach the platforms at the URLs you give, from that namespace.

::::steps
1. Create the namespace, the key and the token secrets. Set `OPENCTI_URL` to the address of the OpenCTI service as it resolves from the `xtm-composer` namespace, such as `http://<service>.<namespace>.svc:8080`.

   ```bash
   kubectl create namespace xtm-composer
   openssl genpkey -algorithm RSA -out private_key.pem -pkeyopt rsa_keygen_bits:4096
   kubectl -n xtm-composer create secret generic xtm-composer-key --from-file=private_key.pem=private_key.pem
   export OPENCTI_URL=http://opencti.opencti.svc:8080 OPENCTI_TOKEN=your-opencti-token
   kubectl -n xtm-composer create secret generic xtm-composer-tokens --from-literal=opencti-token="$OPENCTI_TOKEN"
   ```

2. Write the manifest once, with a new identifier, and keep `xtm-composer.yaml`. The role covers what XTM Composer does: manage Deployments, read Pods and their logs, and create or delete the Secrets of a private registry or a proxy CA.

   ```bash
   cat > xtm-composer.yaml <<EOF
   apiVersion: v1
   kind: ServiceAccount
   metadata:
     name: xtm-composer
   ---
   apiVersion: rbac.authorization.k8s.io/v1
   kind: Role
   metadata:
     name: xtm-composer
   rules:
     - apiGroups: ["apps"]
       resources: ["deployments"]
       verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
     - apiGroups: [""]
       resources: ["pods", "pods/log"]
       verbs: ["get", "list", "watch"]
     - apiGroups: [""]
       resources: ["secrets"]
       verbs: ["create", "delete"]
   ---
   apiVersion: rbac.authorization.k8s.io/v1
   kind: RoleBinding
   metadata:
     name: xtm-composer
   roleRef:
     apiGroup: rbac.authorization.k8s.io
     kind: Role
     name: xtm-composer
   subjects:
     - kind: ServiceAccount
       name: xtm-composer
   ---
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: xtm-composer
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: xtm-composer
     template:
       metadata:
         labels:
           app: xtm-composer
       spec:
         serviceAccountName: xtm-composer
         containers:
           - name: xtm-composer
             image: filigran/xtm-composer:latest
             env:
               - {name: MANAGER__ID, value: "$(python3 -c 'import uuid; print(uuid.uuid4())')"}
               - {name: MANAGER__NAME, value: "XTM Composer"}
               - {name: MANAGER__CREDENTIALS_KEY_FILEPATH, value: /keys/private_key.pem}
               - {name: OPENCTI__URL, value: "$OPENCTI_URL"}
               - name: OPENCTI__TOKEN
                 valueFrom: {secretKeyRef: {name: xtm-composer-tokens, key: opencti-token}}
               - {name: OPENCTI__DAEMON__SELECTOR, value: kubernetes}
             volumeMounts:
               - {name: key, mountPath: /keys, readOnly: true}
         volumes:
           - name: key
             secret: {secretName: xtm-composer-key}
   EOF
   kubectl -n xtm-composer apply -f xtm-composer.yaml
   ```

   For OpenAEV, add `OPENAEV__ENABLE` set to `"true"`, `OPENAEV__URL`, `OPENAEV__TOKEN` and `OPENAEV__DAEMON__SELECTOR` the same way, and set `OPENCTI__ENABLE` to `"false"` if you leave OpenCTI out.

   :::verify
   ```bash
   kubectl -n xtm-composer auth can-i create deployments --as=system:serviceaccount:xtm-composer:xtm-composer
   kubectl -n xtm-composer logs deploy/xtm-composer | grep -E 'Successfully loaded RSA private key|Manager registered'
   ```
   The first command answers `yes`. The logs show the key loaded and one `Manager registered` line per platform.
   :::
::::
:::::
::::::

## Build from source

XTM Composer is a Rust program. To build it, for development or a custom build:

```bash
git clone https://github.com/FiligranHQ/xtm-composer.git
cd xtm-composer
cargo build --release
```

Run `./target/release/xtm-composer` from the repository folder: it reads `config/default.yaml` there, then your environment variables. It writes its log files in a `logs` folder next to the executable.

## Check that it works

Open the catalog of each platform: :menu[Integrations], **Available** tab (:menu[Data > Ingestion > Connector catalog] before 7.260811) in OpenCTI, **Integrations** > **Catalog** in OpenAEV. It no longer says that deploying requires the Integration Manager. Deploy an integration and start it, as in [Deploy a first integration with XTM Composer](/deploy/integrations/quick-start/#step-4-deploy-a-first-integration).

## What's next

- [Configure XTM Composer](/deploy/integrations/configure-composer/), for every setting and the other container engines.
- [Run integrations behind a proxy](/deploy/integrations/proxy/) and [Pull integrations from a private registry](/deploy/integrations/private-registry/).
- [Troubleshoot XTM Composer](/deploy/integrations/troubleshoot/).
