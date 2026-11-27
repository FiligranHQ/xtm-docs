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
last_reviewed: 2026-11-27
migrated_from:
  - opencti:/deployment/integration-manager/registry-authentication/
  - openaev:/deployment/ecosystem/integration-manager/registry-authentification/
---

This page makes XTM Composer pull the images of the integrations from your own registry, such as a mirror of Docker Hub in an isolated network, with or without credentials. Install XTM Composer first: see [Install XTM Composer](/deploy/integrations/install-composer/).

::diagram{id=deploy-integrations-private-registry}

## Before you start

- A registry that holds the integration images under the same names as Docker Hub, such as `opencti/connector-mitre` or `openaev/collector-mitre-attack`, with the same tags: the `-fips` or `-ubi9` tags when XTM Composer adds a tag suffix.
- The host name of the registry, and a user and password or token if it requires them.
- For integrations on a moving tag, such as the `rolling` images of the OpenAEV catalog: HTTPS access from XTM Composer itself to the registry, as in [Follow new images in your registry](#follow-new-images-in-your-registry).

## How XTM Composer names the image

Without a registry, XTM Composer pulls the image the platform asks for, such as `opencti/connector-mitre:<version>`, from Docker Hub. With a registry `server`, it puts the server in front of every image name: `registry.example.com/opencti/connector-mitre:<version>`. It does not check whether the name already has a registry, so mirror the images under their Docker Hub names.

When `manager.image_tag_suffix` is set, XTM Composer also adds it to the tag: `registry.example.com/opencti/connector-mitre:6.9.0-fips`, or `latest-fips` for an image without a tag. The FIPS image of XTM Composer sets `-fips` by default. Mirror the suffixed tags too, or the pulls fail. See [Configure XTM Composer](/deploy/integrations/configure-composer/) for the values it accepts.

## Set the registry

:::steps
1. Add the registry to the environment of XTM Composer, in the `daemon` block of each platform. For OpenAEV, replace `OPENCTI__` with `OPENAEV__`.

   ```text
   OPENCTI__DAEMON__REGISTRY__SERVER=registry.example.com
   OPENCTI__DAEMON__REGISTRY__USERNAME=your-registry-user
   OPENCTI__DAEMON__REGISTRY__PASSWORD=your-registry-password
   ```

   `OPENCTI__DAEMON__REGISTRY__EMAIL` is optional. Leave out the user and password for a registry that needs no credentials: XTM Composer then only adds the server to the image names.

2. Restart XTM Composer. It uses the registry for every image it pulls from then on: when it deploys an integration, when it recreates one after a configuration change, and when it redeploys one whose moving tag points to a new image.
:::

| Key | Environment variable | Effect |
| --- | --- | --- |
| `opencti.daemon.registry.server` | `OPENCTI__DAEMON__REGISTRY__SERVER` | The registry host, put in front of every image name. Required with credentials |
| `opencti.daemon.registry.username` | `OPENCTI__DAEMON__REGISTRY__USERNAME` | The user. Credentials are used only when both the user and the password are set |
| `opencti.daemon.registry.password` | `OPENCTI__DAEMON__REGISTRY__PASSWORD` | The password or token |
| `opencti.daemon.registry.email` | `OPENCTI__DAEMON__REGISTRY__EMAIL` | Optional |

Each platform pulls with the registry of its own block, on every engine. Up to XTM Composer 3.261021.1, Kubernetes, Docker Swarm and Portainer read the `opencti` block for both platforms.

:::warning
If XTM Composer serves OpenAEV on Kubernetes, Docker Swarm or Portainer, and you set its registry in `OPENCTI__DAEMON__REGISTRY__*` only, set it in `OPENAEV__DAEMON__REGISTRY__*` before you upgrade. Otherwise XTM Composer pulls the OpenAEV images from Docker Hub.
:::

## Follow new images in your registry

From XTM Composer 3.261127.0, an integration on a moving tag (`latest`, `rolling` or no tag) follows the image that tag points to in your registry. A tag with a version, such as `7.261030.1`, never moves. XTM Composer itself, not the container engine, asks your registry which image the tag points to: a `HEAD` request on the manifest, such as `https://registry.example.com/v2/openaev/collector-mitre-attack/manifests/rolling`, with the tag suffix applied. This request is not a pull.

It asks once the integration runs, then every 600 seconds by default (`manager.image_update_interval`). When the tag points to another image than the one the integration runs, XTM Composer redeploys the integration on it. On Docker, Podman, Docker Swarm and Portainer, it pulls the new image from your registry first: when the pull fails, the integration keeps running on its image. On Kubernetes, it points the Deployment to the new digest and the cluster pulls: the previous pod keeps running until the new one starts. See [How integrations get new images](/deploy/integrations/how-they-run/#how-integrations-get-new-images).

With a mirror, your integrations change when the tag of the mirror changes: the schedule that syncs your mirror decides when they update.

For the request to succeed:

- The registry answers over HTTPS, on the port of `server` if it has one. XTM Composer cannot read the tags of a registry served over plain HTTP. It waits 15 seconds at most for each request to the registry, so up to 45 seconds when the registry asks for a token.
- XTM Composer sends the user and password of the platform block, as Basic authentication or to the token service the registry names. Without them, a registry that requires credentials refuses the request. On Podman, credentials kept only in the auth file of the host do not reach XTM Composer: set them in its environment too.
- XTM Composer trusts the certificate authorities of the system it runs on. It reaches the registry directly, or through the proxy of the `HTTPS_PROXY` variable of its own environment: the proxy of the platform block does not apply. See [Run integrations behind a proxy](/deploy/integrations/proxy/#reach-the-registry).

When the request fails, the integration keeps running on its image, and XTM Composer logs the warning `Cannot read the digest of the workload image in its registry, the running workload is kept`, with the error. To stop these requests, set `MANAGER__IMAGE_UPDATE_INTERVAL=0`: XTM Composer then no longer redeploys an integration because its tag moved.

## What each engine does with it

::::tabs
:::tab[Docker and Podman]
XTM Composer sends the credentials with each pull. With Podman, you can also leave the credentials out of XTM Composer, keep only the server, and log in on the Podman host with the auth file the Podman service reads:

```bash
sudo podman login --authfile /root/.config/containers/auth.json registry.example.com
```

For a rootless Podman, run `podman login --authfile ~/.config/containers/auth.json registry.example.com` as the user that runs the integrations.

The auth file serves the pulls only. When your registry requires credentials and your integrations run on a moving tag, keep the credentials in XTM Composer too, so that it can read the tag: see [Follow new images in your registry](#follow-new-images-in-your-registry).
:::
:::tab[Kubernetes]
When a user and a password are set, XTM Composer creates at start an image pull Secret of type `kubernetes.io/dockerconfigjson` in its namespace for each platform, and attaches it to the Deployment of each integration of that platform. It deletes and recreates the Secrets at every start, so a changed password takes effect on restart.

Each Secret is named after the server and the platform, in the Kubernetes naming format: `registry.example.com` gives `registry-example-com-opencti` and `registry-example-com-openaev`. Two platforms in the same namespace can then use the same server with different credentials.

XTM Composer also keeps the Secret of its earlier versions, named after the server only (`registry-example-com`), up to date from the `opencti` block: the Deployments created before the upgrade use it until XTM Composer updates them: after a configuration change, or when their moving tag points to a new image. A later version of XTM Composer will stop maintaining it.

The role of XTM Composer must allow `create` and `delete` on `secrets`, as in [Install XTM Composer](/deploy/integrations/install-composer/).

When a moving tag points to a new image, XTM Composer sets the image of the Deployment to the tag followed by the new digest, such as `registry.example.com/openaev/collector-mitre-attack:rolling@sha256:` and the digest. The pod rolls whatever the `imagePullPolicy`, and the cluster pulls with the same Secret.
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
kubectl -n xtm-composer get secret registry-example-com-opencti
```

:::verify
The logs say `Kubernetes registry secret created` with the name of each Secret, and the Secret exists. Replace `registry-example-com` with the name of your server, dots and colons replaced by dashes, and `opencti` with `openaev` for OpenAEV.
:::
::::
:::::

When the image cannot be pulled, XTM Composer deploys nothing, or keeps the previous deployment running after a configuration change or a new image under its tag. The **Logs** of the integration on the platform say why, such as `[XTM Composer] Connector not deployed: the image "registry.example.com/opencti/connector-mitre:6.9.0" cannot be pulled:` followed by the error of the engine, or `Connector not updated, the previous deployment is kept` for an update. After a redeployment on a new image, they say `[XTM Composer] Image updated:` with the old and the new digest. On Kubernetes, the cluster pulls the image: the logs say `Connector waiting for its image` while the pod waits. Check the name and the tag in the registry, and the credentials. See [Troubleshoot XTM Composer](/deploy/integrations/troubleshoot/).

## What's next

- [Run integrations behind a proxy](/deploy/integrations/proxy/).
- [Install offline or with FIPS 140-3](/deploy/install/air-gapped/), to mirror the images of the whole stack.
- [Troubleshoot XTM Composer](/deploy/integrations/troubleshoot/).
