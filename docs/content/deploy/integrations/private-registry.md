---
title: Pull integrations from a private registry
description: Make XTM Composer pull integration images from a private or mirrored container registry, with credentials, on Docker, Podman or Kubernetes.
doc_type: how-to
products:
  - composer
  - opencti
  - openaev
personas: [platform-engineer]
owner: docs-platform
last_reviewed: 2026-10-31
migrated_from:
  - opencti:/deployment/integration-manager/registry-authentication/
  - openaev:/deployment/ecosystem/integration-manager/registry-authentification/
---

This page makes XTM Composer pull the images of the integrations from your own registry, such as a mirror of Docker Hub in an isolated network, with or without credentials. Install XTM Composer first: see [Install XTM Composer](/deploy/integrations/install-composer/).

::diagram{id=deploy-integrations-private-registry}

## Before you start

- A registry that holds the integration images under the same names as Docker Hub, such as `opencti/connector-mitre` or `openaev/collector-mitre-attack`, with the same tags.
- The host name of the registry, and a user and password or token if it requires them.

## How XTM Composer names the image

Without a registry, XTM Composer pulls the image the platform asks for, such as `opencti/connector-mitre:<version>`, from Docker Hub. With a registry `server`, it puts the server in front of every image name: `registry.example.com/opencti/connector-mitre:<version>`. It does not check whether the name already has a registry, so mirror the images under their Docker Hub names.

## Set the registry

:::steps
1. Add the registry to the environment of XTM Composer, in the `daemon` block of each platform. For OpenAEV, replace `OPENCTI__` with `OPENAEV__`.

   ```text
   OPENCTI__DAEMON__REGISTRY__SERVER=registry.example.com
   OPENCTI__DAEMON__REGISTRY__USERNAME=your-registry-user
   OPENCTI__DAEMON__REGISTRY__PASSWORD=your-registry-password
   ```

   `OPENCTI__DAEMON__REGISTRY__EMAIL` is optional. Leave out the user and password for a registry that needs no credentials: XTM Composer then only adds the server to the image names.

2. Restart XTM Composer. It uses the registry for every image it pulls from then on: when it deploys an integration, and when it recreates one after a configuration change.
:::

| Key | Environment variable | Effect |
| --- | --- | --- |
| `opencti.daemon.registry.server` | `OPENCTI__DAEMON__REGISTRY__SERVER` | The registry host, put in front of every image name. Required with credentials |
| `opencti.daemon.registry.username` | `OPENCTI__DAEMON__REGISTRY__USERNAME` | The user. Credentials are used only when both the user and the password are set |
| `opencti.daemon.registry.password` | `OPENCTI__DAEMON__REGISTRY__PASSWORD` | The password or token |
| `opencti.daemon.registry.email` | `OPENCTI__DAEMON__REGISTRY__EMAIL` | Optional |

:::warning
On Kubernetes, Docker Swarm and Portainer, XTM Composer reads the registry of the `opencti` block for both platforms. When it serves OpenAEV with one of these engines, set the registry in `OPENCTI__DAEMON__REGISTRY__*`. On Docker and Podman, each platform uses its own block.
:::

## What each engine does with it

::::tabs
:::tab[Docker and Podman]
XTM Composer sends the credentials with each pull. With Podman, you can also leave the credentials out of XTM Composer, keep only the server, and log in on the Podman host with the auth file the Podman service reads:

```bash
sudo podman login --authfile /root/.config/containers/auth.json registry.example.com
```

For a rootless Podman, run `podman login --authfile ~/.config/containers/auth.json registry.example.com` as the user that runs the integrations.
:::
:::tab[Kubernetes]
When a user and a password are set, XTM Composer creates at start an image pull Secret of type `kubernetes.io/dockerconfigjson` in its namespace, and attaches it to the Deployment of each integration. It deletes and recreates the Secret at every start, so a changed password takes effect on restart.

The Secret is named after the server, in the Kubernetes naming format: `registry.example.com` gives `registry-example-com`. The role of XTM Composer must allow `create` and `delete` on `secrets`, as in [Install XTM Composer](/deploy/integrations/install-composer/).
:::
::::

## Check that it works

Deploy an integration from the catalog, then check where its image came from.

:::::tabs
::::tab[Docker and Podman]
```bash
docker ps --filter label=opencti-manager --format '{{.Names}} {{.Image}}'
```

:::verify
Each image starts with your registry, such as `registry.example.com/opencti/connector-mitre:<version>`.
:::
::::
::::tab[Kubernetes]
```bash
kubectl -n xtm-composer logs deploy/xtm-composer | grep 'Kubernetes registry secret'
kubectl -n xtm-composer get secret registry-example-com
```

:::verify
The logs say `Kubernetes registry secret created`, and the Secret exists. Replace `registry-example-com` with the name of your server, dots replaced by dashes.
:::
::::
:::::

On Docker and Podman, when the pull fails, XTM Composer logs `Error fetching container image` with the image name: check the name in the registry and the credentials.

## What's next

- [Run integrations behind a proxy](/deploy/integrations/proxy/).
- [Install offline or with FIPS 140-3](/deploy/install/air-gapped/), to mirror the images of the whole stack.
- [Troubleshoot XTM Composer](/deploy/integrations/troubleshoot/).
