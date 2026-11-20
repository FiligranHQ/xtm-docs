---
title: Trust your certificate authority and proxy
description: Make OpenCTI and OpenAEV trust your internal certificate authority, send their outgoing traffic through a proxy, and set how OpenAEV agents check certificates.
doc_type: how-to
products:
  - opencti
  - openaev
personas: [platform-engineer]
owner: docs-platform
last_reviewed: 2026-10-31
migrated_from:
  - openaev:/deployment/certificate-validation/
---

This page sets how the platforms of the XTM stack handle TLS certificates and proxies on their outgoing connections: to your services, to identity providers, to the tools they integrate with. You get each platform trusting your internal certificate authority (CA) without turning certificate checks off, the outgoing traffic of OpenCTI going through your proxy, and the OpenAEV agents set for your network. To publish the platforms over HTTPS, see [Publish the platforms over HTTPS](/deploy/install/reverse-proxy/).

::diagram{id=deploy-configure-certificates}

## Before you start

- The stack installed as [Install with Docker Compose](/deploy/install/docker-compose/) describes. The changes below go in `docker-compose.override.yml`: see [Change the stack without editing docker-compose.yml](/deploy/install/docker-compose/#change-the-stack-without-editing-docker-compose-yml).
- The certificate of your CA in PEM format, the full chain if there are intermediate CAs. The examples use `./certs/internal-ca.pem` on the host, next to `docker-compose.yml`.

## Trust your internal certificate authority

Prefer trusting your CA to turning certificate checks off: the platforms keep rejecting every certificate your CA did not sign.

::::tabs
:::tab[OpenCTI]
OpenCTI runs on Node.js, which reads extra CAs from the file named by `NODE_EXTRA_CA_CERTS`, in addition to its own. This covers its HTTPS calls, such as those to an OpenID Connect provider. Put several CAs in one PEM file, one after the other.

Add these lines to the `opencti` service in `docker-compose.override.yml`, then run `docker compose up -d`:

```yaml
services:
  opencti:
    volumes:
      - ./certs:/opt/opencti/certs:ro
    environment:
      - NODE_EXTRA_CA_CERTS=/opt/opencti/certs/internal-ca.pem
```

The connections to the shared services have their own settings. Each takes the path of a certificate in the container, or its PEM content. A CA set here replaces the system CAs for that connection.

| Connection | CA | Certificate check |
| --- | --- | --- |
| Elasticsearch or OpenSearch | `ELASTICSEARCH__SSL__CA`, or `ELASTICSEARCH__SSL__CA_PLAIN` for a PEM content OpenCTI uses when `ELASTICSEARCH__SSL__CA` is empty | `ELASTICSEARCH__SSL__REJECT_UNAUTHORIZED`, `true` by default |
| Redis, with `REDIS__USE_SSL=true` | `REDIS__CA`, a list | |
| RabbitMQ, with `RABBITMQ__USE_SSL=true` | `RABBITMQ__USE_SSL_CA`, a list | `RABBITMQ__USE_SSL_REJECT_UNAUTHORIZED`, off by default |
| RabbitMQ management API, with `RABBITMQ__MANAGEMENT_SSL=true` | | `RABBITMQ__MANAGEMENT_SSL_REJECT_UNAUTHORIZED`, off by default |
| Mail server | | `SMTP__REJECT_UNAUTHORIZED`, off by default |

Write a list as JSON, for example `'REDIS__CA=["/opt/opencti/certs/internal-ca.pem"]'`. The RabbitMQ and mail server checks are off by default: set them to `true` once the CA is in place.
:::
:::tab[OpenAEV]
OpenAEV adds to the CAs of Java the certificates it finds in the folder named by `OPENAEV_EXTRA-TRUSTED-CERTS-DIR`. Every outgoing HTTPS client of the platform uses them, the Elasticsearch client included.

xtm-docker already sets `OPENAEV_EXTRA-TRUSTED-CERTS-DIR=/opt/openaev/additional_certs` but mounts nothing there. Add the folder to the `openaev` service in `docker-compose.override.yml`, then run `docker compose up -d`:

```yaml
services:
  openaev:
    volumes:
      - ./certs:/opt/openaev/additional_certs:ro
```

- OpenAEV reads only the files whose name ends with `.pem`, and only the first X.509 certificate of each, PEM-armoured or DER-encoded: split a chain into one file per CA.
- It skips an expired certificate, or a file it cannot read as a certificate, and keeps the others.
- It reads the folder at start: restart OpenAEV after you add a certificate.

:::verify
```bash
docker compose exec openaev ls /opt/openaev/additional_certs
```
It lists `internal-ca.pem`.
:::
:::
::::

:::only{opencti=">=7.261109"}
## Check certificates on every outgoing connection of OpenCTI

`TLS__REJECT_UNAUTHORIZED` sets, in one place, whether OpenCTI checks the certificate of the servers it connects to. It applies to every outgoing connection that does not set its own check:

- the HTTPS proxy (`HTTPS_PROXY_REJECT_UNAUTHORIZED`), a value also passed to the integrations XTM Composer deploys;
- RabbitMQ and its management API (`RABBITMQ__USE_SSL_REJECT_UNAUTHORIZED`, `RABBITMQ__MANAGEMENT_SSL_REJECT_UNAUTHORIZED`);
- the mail server (`SMTP__REJECT_UNAUTHORIZED`);
- the remote credentials provider of a service;
- the other HTTPS calls that set no SSL verification of their own, such as a synchronization with a remote OpenCTI or a feed.

A variable of the list above, when you set it, wins over `TLS__REJECT_UNAUTHORIZED`. Elasticsearch and OpenSearch keep `ELASTICSEARCH__SSL__REJECT_UNAUTHORIZED`, `true` by default.

`TLS__REJECT_UNAUTHORIZED` is `false` by default, and OpenCTI logs a warning at each start while it is: `TLS certificate verification is disabled for the outgoing connections that do not set their own option`. A later version turns it on by default. Turn it on once your CA is trusted, so that the change of default does not cut a connection:

```yaml
services:
  opencti:
    environment:
      - NODE_EXTRA_CA_CERTS=/opt/opencti/certs/internal-ca.pem
      - TLS__REJECT_UNAUTHORIZED=true
```

After `docker compose up -d`, the warning is gone from the logs of `opencti`, and a connection to a server whose certificate your CA did not sign fails with a certificate error instead of going through.
:::

The OpenCTI workers are Python processes with settings of their own. In xtm-docker they call the platform in plain HTTP on the network of the stack, at `http://opencti:8080`; these settings matter when `OPENCTI_URL` is an `https://` address, for workers that run elsewhere. Set them on the `worker` service:

| Variable | Default | Effect |
| --- | --- | --- |
| `OPENCTI_SSL_VERIFY` | `false` | `false` accepts any certificate from the platform, so anyone between the workers and the platform can read their token. `true` checks it against the public CAs the worker ships with. The path of a PEM file in the container checks it against the CAs of that file only |
| `HTTPS_CA_CERTIFICATES` | | A CA to trust in addition to the system CAs, as the path of a PEM file in the container or as its PEM content: your internal CA, or the CA of a proxy that inspects TLS. The worker checks the certificate of the platform against both, even when `OPENCTI_SSL_VERIFY` is `false`. The same CAs serve its calls to connectors over HTTPS when `WORKER_LISTEN_API_SSL_VERIFY` is `true` |

For example, for workers that reach `https://cti-ingest.example.com`, whose certificate your internal CA signed:

```yaml
services:
  worker:
    volumes:
      - ./certs:/opt/certs:ro
    environment:
      - OPENCTI_URL=https://cti-ingest.example.com
      - OPENCTI_SSL_VERIFY=true
      - HTTPS_CA_CERTIFICATES=/opt/certs/internal-ca.pem
```

At start, each worker logs `Proxy certificates configured`, whatever the CA is for. The other settings of the workers' calls to the platform are in [Scale](/deploy/operate/scale/#shape-the-calls-of-the-workers-to-the-platform).

XTM Composer and the integrations it runs have their own settings: see [Run integrations behind a proxy](/deploy/integrations/proxy/).

## Send OpenCTI traffic through a proxy

OpenCTI sends its outgoing HTTP and HTTPS requests through the proxy you set, except to the hosts you exclude. Add the variables to the `opencti` service in `docker-compose.override.yml`:

| Variable | Default | Set it to |
| --- | --- | --- |
| `HTTP_PROXY` | | The proxy for `http://` URLs, such as `http://proxy.example.com:3128` |
| `HTTPS_PROXY` | | The proxy for `https://` URLs |
| `NO_PROXY` | | The hosts reached directly, comma-separated: host names, patterns with `*`, IP ranges in CIDR notation |
| `HTTPS_PROXY_CA` | `[]` | The CA of a proxy that inspects TLS, as a JSON list of paths or PEM contents |
| `HTTPS_PROXY_REJECT_UNAUTHORIZED` | `false` | `true` to check the certificate of the HTTPS proxy |
| `AWS__PROXY_ENABLED` | `false` | `true` to send the AWS SDK traffic (STS credentials and S3) through the same proxy |

Inside the stack, the platforms call each other on their service names: exclude them, or those calls go to the proxy.

```yaml
services:
  opencti:
    environment:
      - HTTPS_PROXY=http://proxy.example.com:3128
      - HTTP_PROXY=http://proxy.example.com:3128
      - NO_PROXY=localhost,opencti,openaev,xtm-one,elasticsearch,redis,rabbitmq,minio
```

OpenCTI applies these values with its own HTTP agents, and clears the proxy variables of its process so that its libraries do not apply them a second time.

## Set how OpenAEV agents check certificates and proxies

Two settings of the OpenAEV platform do not act on the platform: OpenAEV writes them into the installation commands of its agents and into the commands its agents run. They decide how the agents on your endpoints reach the platform.

| Variable on `openaev` | Default | When `true` |
| --- | --- | --- |
| `OPENAEV_UNSECURED-CERTIFICATE` | `false` | The agents accept the certificate of the platform without checking it, for a platform with a self-signed certificate |
| `OPENAEV_WITH-PROXY` | `false` | The agents use the proxy set on the endpoint (`HTTP_PROXY`, `HTTPS_PROXY`). When `false`, they connect directly |

:::warning
`OPENAEV_UNSECURED-CERTIFICATE=true` lets anyone between the endpoints and the platform impersonate it. Prefer a certificate your endpoints trust.
:::

The installation command carries the values of the moment: reinstall an agent to change them on it.

## Serve HTTPS from a platform

A reverse proxy in front of the stack is the simpler way to serve HTTPS: see [Publish the platforms over HTTPS](/deploy/install/reverse-proxy/). OpenCTI and OpenAEV can also serve HTTPS themselves. In xtm-docker, the health checks and the internal URLs (`http://opencti:8080`, `http://openaev:8080`) then all have to change to `https`.

| | OpenCTI | OpenAEV |
| --- | --- | --- |
| Certificate and key | `APP__HTTPS_CERT__CRT`, `APP__HTTPS_CERT__KEY`: paths or PEM contents. `APP__HTTPS_CERT__PASSPHRASE` when the key is encrypted | `SERVER_SSL_ENABLED=true`, with a keystore: `SERVER_SSL_KEY-STORE`, `SERVER_SSL_KEY-STORE-TYPE` (`PKCS12` by default), `SERVER_SSL_KEY-STORE-PASSWORD`, `SERVER_SSL_KEY-ALIAS` |
| Secure cookies | `APP__HTTPS_CERT__COOKIE_SECURE=true` | `OPENAEV_COOKIE-SECURE=true` |

Behind a reverse proxy that serves HTTPS, set the secure cookies too: the browser then sends the session cookie over HTTPS only.

OpenCTI needs to serve HTTPS itself for the **Client Certificate** login, which reads the user from the certificate the browser presents (see [Authentications](/opencti/administer/authentications/)). When that login is on at start, OpenCTI asks every client for a certificate:

| Variable | Default | Effect |
| --- | --- | --- |
| `APP__HTTPS_CERT__CA` | `[]` | The CAs that sign the client certificates, as a JSON list of paths or PEM contents |
| `APP__HTTPS_CERT__REJECT_UNAUTHORIZED` | `true` | OpenCTI refuses the TLS connection of a client without a certificate signed by one of those CAs. Workers, connectors, scripts and health probes then need a certificate too: set `false` to let them, and the users of other logins, connect without one. The **Client Certificate** login still accepts only a certificate signed by one of those CAs |

OpenCTI reads whether **Client Certificate** is on when it starts: restart it after you turn that login on or off. All settings: [Network and security](/reference/opencti/configuration/#network-and-security).

## What's next

- [Publish the platforms over HTTPS](/deploy/install/reverse-proxy/).
- [Configure the shared services](/deploy/configure/shared-services/), for the TLS settings of each service.
- [Run integrations behind a proxy](/deploy/integrations/proxy/), for XTM Composer and the integrations.
- [Harden the deployment](/deploy/operate/harden/).
