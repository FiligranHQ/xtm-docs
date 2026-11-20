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

1. `Starting XTM composer`, with the version.
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
| `Fail to fetch version, check your configuration` (OpenCTI), `Failed to fetch version, check your configuration` (OpenAEV) | XTM Composer cannot reach the platform URL | Check `OPENCTI__URL` or `OPENAEV__URL` from where XTM Composer runs: the host name, the port, the network, a proxy |
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
| `Connector name collision detected` | A container of the same name belongs to another integration | Rename the instance, or remove the other container. The platform logs of the instance say which one |
| `Error fetching container image` | The image cannot be pulled | Check the image name, the [private registry](/deploy/integrations/private-registry/) and the [proxy](/deploy/integrations/proxy/) |
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

## An integration starts, then fails

XTM Composer reports `Reboot loop detected` when a container has restarted more than three times and has been up for less than five minutes. The cause is in the logs of the integration itself: its **Logs** tab in the platform, or the container.

```bash
docker ps -a --filter label=opencti-manager --format '{{.Names}} {{.Status}}'
docker logs --tail 50 <container name>
```

On Kubernetes, the integration is the Deployment of the same name: `kubectl -n xtm-composer logs deploy/<name>`.

The usual causes:

- The integration cannot reach the platform. It uses the URL XTM Composer uses, `OPENCTI__URL` or `OPENAEV__URL`, so it must run on a network where that URL resolves: set `DAEMON__DOCKER__NETWORK_MODE` to the network of the platform, `xtm_default` in xtm-docker.
- The integration cannot reach RabbitMQ at the address the platform gives it. See [Networking](/deploy/integrations/manual/#networking).
- A required setting of the integration is empty or wrong. Fix it with **Update**.

## Logs do not show in the platform

- Only the instances XTM Composer manages show logs in the interface.
- XTM Composer sends them every `logs_schedule` seconds, 10 by default: wait a few seconds.
- On Kubernetes, the role must allow reading `pods/log`.

## The configuration file is not read

- XTM Composer reads `/opt/xtm-composer/config/<COMPOSER_ENV>.yaml`, `production.yaml` by default. A file mounted at `/config` is ignored.
- Mount the file, not the `config` folder, which would hide `default.yaml`.
- Environment variables win over the file. They use `__` between levels: `MANAGER__LOGGER__LEVEL`, not `MANAGER.LOGGER.LEVEL`.

## What's next

- [Configure XTM Composer](/deploy/integrations/configure-composer/).
- [Troubleshoot the XTM stack](/deploy/operate/troubleshoot/), for the rest of the stack.
