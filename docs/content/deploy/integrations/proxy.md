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
last_reviewed: 2026-10-31
migrated_from:
  - opencti:/deployment/integration-manager/proxy-configuration/
  - openaev:/deployment/ecosystem/integration-manager/proxy-configuration/
---

This page sends XTM Composer and the integrations it deploys through your proxy. The settings belong to each platform block, `opencti` or `openaev`, so OpenCTI and OpenAEV integrations can use different proxies. Install XTM Composer first: see [Install XTM Composer](/deploy/integrations/install-composer/).

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

   Put in `NO_PROXY` every host the integrations reach directly: the platform and RabbitMQ at least, as the XTM stack names them here.

2. Restart XTM Composer. Integrations get the proxy when XTM Composer creates their container. For an integration already deployed, change its configuration with **Update**, or stop it and remove its container: XTM Composer then creates it again.
:::

| Key | Environment variable | Default | Effect |
| --- | --- | --- | --- |
| `opencti.with_proxy` | `OPENCTI__WITH_PROXY` | `false` | `true` turns the proxy on for this platform. With `false`, XTM Composer ignores every proxy, including the `HTTP_PROXY` variables of its own environment |
| `opencti.http_proxy` | `OPENCTI__HTTP_PROXY` | | The proxy for HTTP. Without it, XTM Composer uses `HTTP_PROXY` from its own environment |
| `opencti.https_proxy` | `OPENCTI__HTTPS_PROXY` | | The proxy for HTTPS. Without it, XTM Composer uses `HTTPS_PROXY` from its own environment |
| `opencti.no_proxy` | `OPENCTI__NO_PROXY` | | Comma-separated hosts the integrations reach without the proxy. Without it, `NO_PROXY` from the environment of XTM Composer. It applies to the integrations only: when `http_proxy` or `https_proxy` is set, XTM Composer sends its own requests to the platform through that proxy even if the platform is in this list |

With the proxy on, XTM Composer sends its own requests to the platform through it, and gives each integration container `HTTP_PROXY`, `HTTPS_PROXY` and `NO_PROXY`.

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

2. On Docker, Podman, Docker Swarm or Portainer, let the container engine see the bundle XTM Composer writes. XTM Composer writes the certificates to `/tmp/xtm-composer-proxy-ca/` and asks the engine to mount that file in each integration. The engine looks for it on the host, so when XTM Composer runs in a container, mount the same host folder at the same path:

   ```text
   -v /tmp/xtm-composer-proxy-ca:/tmp/xtm-composer-proxy-ca
   ```

   On Kubernetes, nothing to add: XTM Composer puts the bundle in a Secret named `<integration>-proxy-ca` and mounts it from there.

3. Restart XTM Composer, then recreate the integrations as in the previous procedure.
:::

Each integration then finds the bundle at `/etc/ssl/certs/xtm-proxy-ca.crt`, and the variables that point most runtimes to it: `SSL_CERT_FILE`, `REQUESTS_CA_BUNDLE`, `NODE_EXTRA_CA_CERTS` and `CURL_CA_BUNDLE`.

:::note
These settings are only for the proxy. The certificate of the platform itself is configured on the platform: see [Trust your certificate authority and proxy](/deploy/configure/certificates/). To make XTM Composer accept an invalid platform certificate while you test, set `OPENCTI__UNSECURED_CERTIFICATE=true`.
:::

## Check that it works

Read the environment of an integration container XTM Composer created after the change:

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
