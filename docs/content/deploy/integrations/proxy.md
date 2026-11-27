---
title: Run integrations behind a proxy
description: Send XTM Composer and the integrations it runs through an HTTP or HTTPS proxy, including a TLS-intercepting proxy with its own certificate authority.
doc_type: how-to
products:
  - composer
  - opencti
  - openaev
personas: [platform-engineer]
owner: docs-platform
last_reviewed: 2026-11-27
migrated_from:
  - opencti:/deployment/integration-manager/proxy-configuration/
  - openaev:/deployment/ecosystem/integration-manager/proxy-configuration/
---

This page sends XTM Composer and the integrations it deploys through your proxy. The settings belong to each platform block, `opencti` or `openaev`, so OpenCTI and OpenAEV integrations can use different proxies. The requests of XTM Composer to the image registry follow the proxy variables of its own environment instead: see [Reach the registry](#reach-the-registry). Install XTM Composer first: see [Install XTM Composer](/deploy/integrations/install-composer/).

::diagram{id=deploy-integrations-proxy}

## Before you start

- The URL of your proxy, such as `http://proxy.example.com:8080`, and the hosts that must not go through it.
- For a proxy that intercepts TLS, its certificate authority as a PEM file.

## Turn on the proxy

:::steps
1. Add the proxy to the environment of XTM Composer: its env file, such as `composer.env`, or the `environment` of its service, in `docker-compose.override.yml` for xtm-docker. For OpenAEV, replace `OPENCTI__` with `OPENAEV__`.

   ```text
   OPENCTI__WITH_PROXY=true
   OPENCTI__HTTP_PROXY=http://proxy.example.com:8080
   OPENCTI__HTTPS_PROXY=http://proxy.example.com:8080
   OPENCTI__NO_PROXY=localhost,127.0.0.1,opencti,openaev,rabbitmq
   ```

   Put in `NO_PROXY` every host XTM Composer and the integrations reach directly: the platform and RabbitMQ at least, as the XTM stack names them here.

2. Restart XTM Composer. Integrations get the proxy when XTM Composer creates their container. For an integration already deployed, change its configuration with **Update**, or stop it and remove its container: XTM Composer then creates it again.
:::

| Key | Environment variable | Default | Effect |
| --- | --- | --- | --- |
| `opencti.with_proxy` | `OPENCTI__WITH_PROXY` | `false` | `true` turns the proxy on for this platform. With `false`, XTM Composer reaches the platform without a proxy and adds no proxy variable to its integrations, not even the `HTTP_PROXY` variables of its own environment. OpenCTI still gives its integrations the proxy of its own configuration, when it has one: see [Trust your certificate authority and proxy](/deploy/configure/certificates/) |
| `opencti.http_proxy` | `OPENCTI__HTTP_PROXY` | | The proxy for HTTP. Without it, the integrations get `HTTP_PROXY` from the environment of XTM Composer, and so do its own requests to the platform unless `https_proxy` is set |
| `opencti.https_proxy` | `OPENCTI__HTTPS_PROXY` | | The proxy for HTTPS. Without it, the integrations get `HTTPS_PROXY` from the environment of XTM Composer, and so do its own requests to the platform unless `http_proxy` is set |
| `opencti.no_proxy` | `OPENCTI__NO_PROXY` | | Comma-separated hosts, domains or CIDR ranges reached without the proxy, such as `opencti,.example.com,10.0.0.0/8`. Without it, `NO_PROXY` from the environment of XTM Composer. It applies to the integrations and to the requests of XTM Composer to the platform |

With the proxy on, XTM Composer sends its own requests to the platform through it, except to the hosts of `no_proxy`, and gives each integration container `HTTP_PROXY`, `HTTPS_PROXY` and `NO_PROXY`. When you set `no_proxy` but neither `http_proxy` nor `https_proxy`, XTM Composer takes the proxy for its own requests from `HTTP_PROXY` and `HTTPS_PROXY` of its environment, else from `ALL_PROXY`.

Up to XTM Composer 3.261021.1, `no_proxy` applied to the integrations only: with `http_proxy` or `https_proxy` set, XTM Composer reached the platform through the proxy even when the platform was in the list.

## Trust a TLS-intercepting proxy

A proxy that decrypts HTTPS presents certificates signed by its own authority. Give that authority to the integrations:

| Key | Environment variable | Default | Effect |
| --- | --- | --- | --- |
| `opencti.https_proxy_ca` | `OPENCTI__HTTPS_PROXY_CA` | | Certificates to trust: a list of file paths or PEM blocks. As an environment variable, a JSON array such as `["/certs/proxy-ca.pem"]` |
| `opencti.https_proxy_reject_unauthorized` | `OPENCTI__HTTPS_PROXY_REJECT_UNAUTHORIZED` | `true` | `false` gives the integrations `NODE_TLS_REJECT_UNAUTHORIZED=0`, which turns off certificate checks in Node.js integrations. Prefer `https_proxy_ca` |

:::steps
1. Make the PEM file readable by XTM Composer, which reads the paths: mount it in its container, for example at `/certs/proxy-ca.pem`, and add the variable to its environment. In a shell, put the value in single quotes.

   ```text
   OPENCTI__HTTPS_PROXY_CA=["/certs/proxy-ca.pem"]
   ```

2. On Docker, Podman, Docker Swarm or Portainer, let the container engine see the bundle XTM Composer writes. XTM Composer writes the certificates to `/tmp/xtm-composer-proxy-ca/` and asks the engine to mount that file in each integration. The engine looks for it on the host that runs the integration, so when XTM Composer runs in a container, mount the same host folder at the same path:

   ```text
   -v /tmp/xtm-composer-proxy-ca:/tmp/xtm-composer-proxy-ca
   ```

   On Docker Swarm with several nodes, or with Portainer on another host, the file must also exist at that path on the host of each integration: share the folder, or keep the integrations on the host of XTM Composer.

   On Kubernetes, nothing to add: XTM Composer puts the bundle in a Secret named after the integration's Deployment, `<deployment>-proxy-ca` cut to 63 characters, and mounts it from there. From XTM Composer 3.261121.0, when the integration's name is not a valid Kubernetes name, such as one longer than 63 characters, the Deployment name ends with 8 characters of a hash. The Secret stays `<deployment>-proxy-ca` cut to 63 characters: for a name longer than 63 characters, it has the name of the Deployment. See the naming rule in [XTM Composer architecture](/deploy/integrations/architecture/).

3. Restart XTM Composer, then recreate the integrations as in the previous procedure.
:::

Each integration then finds the bundle at `/etc/ssl/certs/xtm-proxy-ca.crt`, and the variables that point most runtimes to it: `SSL_CERT_FILE`, `REQUESTS_CA_BUNDLE`, `NODE_EXTRA_CA_CERTS` and `CURL_CA_BUNDLE`.

:::note
These settings are only for the proxy. The certificate of the platform itself is configured on the platform: see [Trust your certificate authority and proxy](/deploy/configure/certificates/). To make XTM Composer accept an invalid platform certificate while you test, set `OPENCTI__UNSECURED_CERTIFICATE=true`.
:::

## Reach the registry

From XTM Composer 3.261127.0, XTM Composer itself asks the image registry which image a moving tag (`latest`, `rolling` or no tag) points to, and redeploys the integrations on that tag when it changes. The OpenAEV catalog deploys its integrations on `rolling`. See [Follow new images in your registry](/deploy/integrations/private-registry/#follow-new-images-in-your-registry).

These requests do not use the settings of the platform blocks: `with_proxy`, `http_proxy`, `https_proxy`, `no_proxy`, `https_proxy_ca` and `unsecured_certificate` apply only to the requests to the platform and to the integrations. The requests to the registry go through the proxy of the `HTTPS_PROXY` variable of the environment of XTM Composer, except to the hosts of its `NO_PROXY` variable. They trust the certificate authorities of the system XTM Composer runs on.

:::steps
1. Add the proxy variables to the environment of XTM Composer, next to those of the platform blocks.

   ```text
   HTTPS_PROXY=http://proxy.example.com:8080
   NO_PROXY=localhost,127.0.0.1,opencti,openaev,rabbitmq
   ```

   For images from Docker Hub, the proxy must let through `registry-1.docker.io` and `auth.docker.io`. For your own registry, its host: see [Pull integrations from a private registry](/deploy/integrations/private-registry/).

2. Restart XTM Composer.
:::

A proxy that intercepts TLS breaks these requests unless the system of XTM Composer trusts its authority: `https_proxy_ca` does not apply to them. When a request fails, the integration keeps running on its image, and XTM Composer logs the warning `Cannot read the digest of the workload image in its registry, the running workload is kept`, with the error. To stop these requests, set `MANAGER__IMAGE_UPDATE_INTERVAL=0`: XTM Composer then no longer redeploys an integration because its tag moved.

## Check that it works

Check the proxy XTM Composer uses for its own requests, in its logs at start. From the xtm-docker folder:

```bash
docker compose logs xtm-composer 2>&1 | grep -E 'Using HTTPS? proxy'
```

Elsewhere, use the name of the XTM Composer container.

:::verify
A line `Using HTTP proxy` or `Using HTTPS proxy` per platform, with its `no_proxy` list and its `source`: `explicit` for the URL of the platform block, `environment` for a URL read from the environment. When the block sets no proxy URL and no `no_proxy`, XTM Composer uses the proxy variables of its environment and logs no such line.
:::

Then read the environment of an integration container XTM Composer created after the change:

```bash
docker ps --filter label=opencti-manager --format '{{.Names}}'
docker inspect --format '{{range .Config.Env}}{{println .}}{{end}}' <container name> | grep -E '^(HTTP_PROXY|HTTPS_PROXY|NO_PROXY|SSL_CERT_FILE)='
```

:::verify
It prints `HTTP_PROXY`, `HTTPS_PROXY` and `NO_PROXY` with your values, and `SSL_CERT_FILE=/etc/ssl/certs/xtm-proxy-ca.crt` when you set a certificate authority. With `MANAGER__DEBUG__SHOW_ENV_VARS=true`, XTM Composer also logs these variables when it starts a container, with the proxy URLs redacted.
:::

## What's next

- [Pull integrations from a private registry](/deploy/integrations/private-registry/), when the images come from your registry.
- [Configure XTM Composer](/deploy/integrations/configure-composer/).
