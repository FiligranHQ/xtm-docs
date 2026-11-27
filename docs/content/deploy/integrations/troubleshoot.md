---
title: Troubleshoot XTM Composer
description: Find why XTM Composer does not start, does not register with OpenCTI or OpenAEV, or does not deploy an integration, from the messages it logs.
doc_type: how-to
products:
  - composer
  - opencti
  - openaev
personas: [platform-engineer]
owner: docs-platform
last_reviewed: 2026-10-31
migrated_from:
  - opencti:/deployment/integration-manager/troubleshooting/
  - openaev:/deployment/ecosystem/integration-manager/troubleshooting/
---

This page helps you find why XTM Composer or an integration it manages does not work. Start from the logs of XTM Composer: almost every problem shows there as one of the messages below, with what to do about it. For how XTM Composer works, see [XTM Composer architecture](/deploy/integrations/architecture/).

::diagram{id=deploy-integrations-troubleshoot}

## Read the logs

| Where XTM Composer runs | Command |
| --- | --- |
| XTM stack | `docker compose logs -f xtm-composer` |
| Docker | `docker logs -f xtm-composer` |
| Podman | `podman logs -f xtm-composer` |
| Kubernetes | `kubectl -n xtm-composer logs -f deploy/xtm-composer` |

For more detail, set `MANAGER__LOGGER__LEVEL=debug` and restart it. `MANAGER__LOGGER__FORMAT=pretty` makes the console easier to read than JSON.

A healthy start logs, in this order:

1. `Starting XTM composer`, with the version and the `image_tag_suffix` it adds to the integration images, empty when there is none.
2. `Successfully loaded RSA private key (PKCS#8 format)`.
3. `Manager registered`, once for each platform it serves.

Then, every few seconds for each integration, `Nothing to execute` when all is in place, or the action it takes: `Deploying the container`, `Starting`, `Stopping`, `Refreshing`.

## XTM Composer stops at start

When XTM Composer serves OpenAEV alone, the key messages of this table appear at its first registration instead of at start, and XTM Composer keeps running without registering.

| Message | Cause | Fix |
| --- | --- | --- |
| `No credentials key provided!` | No key configured | Set `MANAGER__CREDENTIALS_KEY_FILEPATH` to the path of the key in the container, or `MANAGER__CREDENTIALS_KEY` |
| `Failed to read credentials key file` | The file is not at that path | Check the volume that mounts the key and the path. In xtm-docker, the key is in the `rsakeys` volume, made by `rsa-key-generator` |
| `Invalid private key format. Expected PKCS#8 PEM format` | The key is PKCS#1 (`BEGIN RSA PRIVATE KEY`), as older `openssl genrsa` versions write | Convert it, as below |
| `Invalid daemon configuration` | `DAEMON__SELECTOR` has another value | Use `docker`, `podman`, `kubernetes`, `swarm` or `portainer` |
| `Missing swarm configuration`, `Missing portainer configuration` | The block of the selector is absent | Add the `daemon.swarm` or `daemon.portainer` block: see [Configure XTM Composer](/deploy/integrations/configure-composer/#the-container-engine) |
| `Invalid portainer type configuration` | `daemon.portainer.env_type` is not `docker` | Set it to `docker`: XTM Composer deploys through Portainer on Docker environments only |
| `Invalid log level`, `Invalid log format` | A logger value is misspelled | `trace`, `debug`, `info`, `warn` or `error`; `json` or `pretty` |
| `invalid image_tag_suffix "-alpine", expected one of: -fips, -ubi9` | `MANAGER__IMAGE_TAG_SUFFIX` has another value | Set `-fips` or `-ubi9`, or leave it empty for no suffix: see [Configure XTM Composer](/deploy/integrations/configure-composer/) |

To convert a PKCS#1 key without changing it, which keeps the integrations the platform already encrypted for it:

```bash
head -n 1 private_key.pem
openssl pkcs8 -topk8 -inform PEM -outform PEM -nocrypt -in private_key.pem -out private_key_pkcs8.pem
```

The first command shows `-----BEGIN RSA PRIVATE KEY-----` for PKCS#1. Point `MANAGER__CREDENTIALS_KEY_FILEPATH` to the new file, or replace the old one with it, and restart XTM Composer.

## XTM Composer does not register

XTM Composer retries every `ping_alive_schedule` seconds, 60 by default, until the platform answers.

OpenCTI shows no list of the XTM Composer instances. It counts one as active while it has reported in the last 5 minutes: until then, and again 5 minutes after XTM Composer stops, the catalog of an Enterprise Edition platform says that deploying requires the **Integration Manager**.

| Message | Cause | Fix |
| --- | --- | --- |
| `Fail to fetch version, check your configuration` (OpenCTI), `Failed to fetch version, check your configuration` (OpenAEV) | XTM Composer cannot reach the platform URL | Check `OPENCTI__URL` or `OPENAEV__URL` from where XTM Composer runs: the host name, the port, the network. Behind a [proxy](/deploy/integrations/proxy/), put the platform host in `OPENCTI__NO_PROXY` or `OPENAEV__NO_PROXY` when XTM Composer must reach it directly |
| `GraphQL operation failed` (OpenCTI), `Failed to fetch version: non-success status code` (OpenAEV) | The platform answers but refuses the request | Check the token: `OPENCTI__TOKEN` or `OPENAEV__TOKEN` |
| `OpenCTI backend does not support XTM composer manager registration` | OpenCTI is older than 6.8.17 | Upgrade OpenCTI |
| `OpenCTI connectors orchestration disabled`, `OpenAEV connectors orchestration disabled` | The platform block has `enable: false` | Set `OPENCTI__ENABLE` or `OPENAEV__ENABLE` to `true`. xtm-docker sets both to `true` |

A certificate the platform presents that XTM Composer does not trust also fails the connection. Fix the certificate: see [Trust your certificate authority and proxy](/deploy/configure/certificates/). To test with a self-signed certificate only, set `OPENCTI__UNSECURED_CERTIFICATE=true`.

Test the URL and the token from the network of XTM Composer, here the network of the XTM stack:

```bash
set -a; . ./.env; set +a
docker run --rm --network xtm_default curlimages/curl:8.11.1 -s -H "Authorization: Bearer $OPENCTI_ADMIN_TOKEN" -H 'Content-Type: application/json' -d '{"query":"{ about { version } }"}' http://opencti:8080/graphql
docker run --rm --network xtm_default curlimages/curl:8.11.1 -s -H "Authorization: Bearer $OPENAEV_ADMIN_TOKEN" http://openaev:8080/api/settings/version
```

:::verify
Each command prints the version of the platform: in a small JSON document for OpenCTI, as text for OpenAEV. An error or nothing means XTM Composer gets the same answer.
:::

### What OpenAEV says

OpenAEV shows its own messages when you deploy from **Integrations** > **Catalog**:

| Message | Cause | Fix |
| --- | --- | --- |
| `XTM Composer is not configured in the platform settings` | No XTM Composer ever registered | Start XTM Composer with `OPENAEV__ENABLE=true` and check it logs `Manager registered` |
| `XTM Composer is not reachable` | XTM Composer has not pinged OpenAEV for two hours | Check that XTM Composer runs and reaches OpenAEV |
| `Invalid xtm-composer identifier` | Another XTM Composer, or the same one with a new `manager.id`, registered since | Run a single XTM Composer per OpenAEV, with a stable `MANAGER__ID` |

## An integration is not deployed or not updated

| Message in the logs of XTM Composer | Cause | Fix |
| --- | --- | --- |
| `Refusing to deploy connector: some encrypted configuration values cannot be decrypted` (or `refresh`) | The key of XTM Composer is not the one the platform encrypted the secrets with, for example after a new key | Put the previous key back. Otherwise, enter the secrets of the integration again with **Update** |
| `Connector name collision detected` | A container, Kubernetes Deployment or Swarm service of the same name belongs to another integration, or was created by hand | Rename the instance, or remove the other workload. The **Logs** tab of the instance names it: `[XTM Composer] Connector not deployed: a deployment named "<name>" already exists and belongs to another owner (<id>).` |
| `Kubernetes creation api error`, `Error creating swarm service` | The engine refused the workload, for the reason in the `error` field. XTM Composer 3.261120.0 and earlier named it after the instance as is, so the engine refused a name longer than 63 characters, as many OpenAEV integrations have | Upgrade XTM Composer to 3.261121.0 or later, which shortens such names. Otherwise, fix what the `error` field names |
| `Cannot pull the workload image, deployment canceled` | The engine cannot pull the image in the `image` field, for the reason in the `error` field. XTM Composer deploys nothing and tries again at its next pass | See [The image cannot be pulled](#the-image-cannot-be-pulled) |
| `Cannot pull the workload image, refresh canceled` | The same, after a change of the configuration of the integration. The previous container keeps running, with the previous configuration | See [The image cannot be pulled](#the-image-cannot-be-pulled). XTM Composer applies the change once the pull works |
| `Error creating container` | Docker refused the container, for the reason in the `error` field, such as a network that does not exist | Set `DAEMON__DOCKER__NETWORK_MODE` to a network that `docker network ls` lists, such as `xtm_default` in xtm-docker |
| `Cannot query the Docker API` | XTM Composer cannot use the Docker socket | Mount `/var/run/docker.sock` in its container, and check that the Docker daemon runs |
| `Portainer error fetching containers`, `Error deploying the container` (Portainer) | Portainer refused the request: wrong `api`, `api_key`, `env_id` or `api_version`. The `error` field gives the answer of Portainer | Test the same request from where XTM Composer runs, as below |
| `No Podman API socket found` | No Podman socket exists at the paths it checks | Enable the socket, then mount it at `/run/podman/podman.sock`: see [Install XTM Composer](/deploy/integrations/install-composer/) |

With the `portainer` selector, XTM Composer calls the Docker API that Portainer exposes for the environment, with the API key in the `X-API-KEY` header. It does not check the certificate of Portainer. Run the same call with the values of your `daemon.portainer` block:

```bash
curl -sk -H "X-API-KEY: $PORTAINER_API_KEY" https://portainer.example.com:9443/api/endpoints/3/docker/v1.44/containers/json
```

:::verify
It prints a JSON list of the containers of the environment. An error message names what Portainer refused: the key, or an environment ID that does not exist.
:::

On Kubernetes, a `forbidden` error means the role of XTM Composer misses a permission. Check it:

```bash
kubectl -n xtm-composer auth can-i create deployments --as=system:serviceaccount:xtm-composer:xtm-composer
kubectl -n xtm-composer auth can-i get pods --subresource=log --as=system:serviceaccount:xtm-composer:xtm-composer
```

:::verify
Both commands answer `yes`. Otherwise, apply the role of [Install XTM Composer](/deploy/integrations/install-composer/).
:::

### The image cannot be pulled

With Docker, Podman, Docker Swarm and Portainer, XTM Composer pulls the image of the integration before it deploys or updates it. When the pull fails, it changes nothing, and writes the reason in the **Logs** tab of the integration in the platform:

- `[XTM Composer] Connector not deployed: the image "<image>" cannot be pulled: <error>`: nothing runs. An integration the platform showed as started is set to stopped.
- `[XTM Composer] Connector not updated, the previous deployment is kept: the image "<image>" cannot be pulled: <error>`: the previous container keeps running.

XTM Composer 3.261021.1 and earlier logged `Error fetching container image` with Docker, and removed the running container before a failed update.

On Kubernetes, the nodes pull the image. A pod that cannot get it waits in `ErrImagePull`, `ImagePullBackOff`, `InvalidImageName` or `ErrImageNeverPull`, and XTM Composer reports it in the **Logs** tab:

- `[XTM Composer] Connector waiting for its image: ...`: no pod of the integration runs, and its status is stopped. XTM Composer does not start it again while the image is missing, but stopping the integration in the platform still scales the Deployment down to zero.
- `[XTM Composer] Connector not updated, the previous deployment is kept: ...`: the pod of the previous version keeps running during the update.

The answer of the registry is in the events of the pod:

```bash
kubectl -n xtm-composer get pods
kubectl -n xtm-composer describe pod <pod name>
```

The usual causes:

- The image name or its tag is wrong. The `image` field, or the message, gives the name XTM Composer asked for.
- `MANAGER__IMAGE_TAG_SUFFIX` adds `-fips` or `-ubi9` to the tag, and the integration is not published with that tag. OpenAEV publishes no FIPS images. See [Configure XTM Composer](/deploy/integrations/configure-composer/).
- The registry refuses the credentials, or XTM Composer pulls from Docker Hub when your images are in a private registry. Each platform pulls with the registry of its own block: since XTM Composer 3.261120.0, an OpenAEV integration on Kubernetes, Docker Swarm or Portainer no longer uses `OPENCTI__DAEMON__REGISTRY__*`. On Kubernetes, the pull secret is named after the server and the platform, such as `registry-example-com-openaev`. See [Pull integrations from a private registry](/deploy/integrations/private-registry/).
- A proxy blocks the registry. See [Run integrations behind a proxy](/deploy/integrations/proxy/).

## An integration starts, then fails

XTM Composer reports `Reboot loop detected` when a container has restarted more than three times and has been up for less than five minutes. The cause is in the logs of the integration itself: its **Logs** tab in the platform, or the container.

```bash
docker ps -a --filter label=opencti-manager --format '{{.Names}} {{.Status}}'
docker logs --tail 50 <container name>
```

XTM Composer labels each container with `xtm-manager` and, until a future version drops it, with `opencti-manager`. Containers deployed before XTM Composer 3.261120.0 carry `opencti-manager` only, so the filter above lists them all.

On Kubernetes, the integration is a Deployment named after the instance: `kubectl -n xtm-composer logs deploy/<name>`. The name is the instance name in lower case, with `-` for every symbol and space. Since XTM Composer 3.261121.0, when that name is not a valid Kubernetes name, XTM Composer shortens it. This happens when the name is longer than 63 characters, has an accented letter, or starts or ends with a symbol. XTM Composer then keeps up to 54 characters and adds `-` and 8 hexadecimal characters. For example, an OpenAEV integration, named after its title and the ID of the instance, becomes `dns-enumeration--subdomains-and-dns-records--7653e749-` followed by those 8 characters. A name that was already valid does not change. To find the Deployment of an integration, list them with the ID of their instance, in `opencti-connector-id` for the Deployments made before XTM Composer 3.261120.0:

```bash
kubectl -n xtm-composer get deployments -L xtm-workload-id,opencti-connector-id
```

On Docker Swarm, the service follows the same rule: `docker service logs <name>`.

The usual causes:

- The integration cannot reach the platform. It uses the URL XTM Composer uses, `OPENCTI__URL` or `OPENAEV__URL`, so it must run on a network where that URL resolves: set `DAEMON__DOCKER__NETWORK_MODE` to the network of the platform, `xtm_default` in xtm-docker.
- The integration cannot reach RabbitMQ at the address the platform gives it. See [Networking](/deploy/integrations/manual/#networking).
- A required setting of the integration is empty or wrong. Fix it with **Update**.

## Logs do not show in the platform

- Only the instances XTM Composer manages show logs in the interface.
- XTM Composer sends the last 100 lines of the integration every `logs_schedule` seconds, 10 by default: wait a few seconds.
- On Kubernetes, XTM Composer reads the container of the integration, in the pod where that container runs before a pod still waiting or left over from an update, the newest first. The restart count and the start time come from the same pod. When it cannot read any pod, the **Logs** tab shows `[XTM Composer] Cannot read the connector logs:` with the answer of Kubernetes. A `forbidden` answer means the role must allow reading `pods/log`.

## The configuration file is not read

- XTM Composer reads `/opt/xtm-composer/config/<COMPOSER_ENV>.yaml`, `production.yaml` by default. A file mounted at `/config` is ignored.
- Mount the file, not the `config` folder, which would hide `default.yaml`.
- Environment variables win over the file. They use `__` between levels: `MANAGER__LOGGER__LEVEL`, not `MANAGER.LOGGER.LEVEL`.

## What's next

- [Configure XTM Composer](/deploy/integrations/configure-composer/).
- [Troubleshoot the XTM stack](/deploy/operate/troubleshoot/), for the rest of the stack.
