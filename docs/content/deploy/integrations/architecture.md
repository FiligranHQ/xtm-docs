---
title: XTM Composer architecture
description: How XTM Composer registers with OpenCTI and OpenAEV, reconciles their integrations with the containers it runs, and decrypts their secrets.
doc_type: explanation
products:
  - composer
  - opencti
  - openaev
personas: [platform-engineer]
owner: docs-platform
last_reviewed: 2026-10-31
migrated_from:
  - opencti:/deployment/integration-manager/architecture/
---

XTM Composer is a small Rust program that sits between the platforms and a container engine. It never receives orders: it asks each platform, every few seconds, which integrations should exist and in which state, then makes the container engine match. This page explains that loop, how it registers, and how the secrets of an integration reach its container, so you can reason about its behavior when you install or troubleshoot it.

::diagram{id=deploy-integrations-architecture}

## Two loops per platform

For each platform whose block has `enable: true`, XTM Composer runs two independent loops.

| Loop | Interval | What it does |
| --- | --- | --- |
| Registration and ping | `manager.ping_alive_schedule`, 60 seconds by default | Reads the platform version, registers, then pings. It registers again when the platform version changes, and starts over when a ping fails |
| Reconciliation | `manager.execute_schedule`, 10 seconds by default | Compares the integrations the platform lists with the containers that run, and acts on the difference |

Both platforms are reached with the URL and token of their block: the GraphQL API at `<url>/graphql` for OpenCTI, the REST API at `<url>/api` for OpenAEV.

## Registration

At start, when it serves OpenCTI, XTM Composer loads its RSA private key and stops if the key is missing or is not a PKCS#8 PEM file. When it serves OpenAEV alone, it reads the key only when it first registers: a bad key then shows as an error in its logs, and XTM Composer does not register. With its key loaded, it registers with each platform under `manager.id` and `manager.name`, and sends its public key: as a PKCS#1 PEM document to OpenCTI, as a SubjectPublicKeyInfo PEM document to OpenAEV.

The platform keeps the identifier and the public key. OpenAEV keeps a single XTM Composer: each registration replaces the previous one, and the pings of a replaced XTM Composer are refused. Run one XTM Composer per OpenAEV. OpenAEV considers XTM Composer unreachable when it has not pinged for two hours.

Keep `manager.id` and the key stable: they are how the platform recognizes XTM Composer and encrypts for it.

## Reconciliation

Every `execute_schedule` seconds, XTM Composer fetches the integration instances of the platform. For each one it looks for a container of the same name, then:

| Situation | What XTM Composer does |
| --- | --- |
| No container | Pulls the image and creates the container, stopped, then reports it as stopped |
| The requested state is started and the container is stopped | Starts it |
| The requested state is stopped and the container runs | Stops it |
| The configuration hash of the instance differs from the one of the container | Removes the container and creates it again with the new configuration |
| The container runs | Reports its health, its restart count and its start date: when it starts and every 30 seconds |
| `logs_schedule` seconds have passed, 10 by default | Sends the latest lines of its logs to the platform |
| A container of that name belongs to another instance | Leaves it alone and writes in the logs of the instance that the name is taken |

Then it removes the containers it manages for this platform that no longer match an instance, or whose name changed.

A container counts as in a reboot loop when it has restarted more than three times and has been up for less than five minutes. XTM Composer reports it to the platform with its health.

### How it marks its containers

XTM Composer lists and cleans up only the containers whose `opencti-manager` label is its own `manager.id`. Every container it creates has:

| Label or variable | Value |
| --- | --- |
| Label `opencti-manager` | `manager.id` |
| Label `opencti-connector-id` | The ID of the instance in the platform |
| Label `opencti-platform` | `opencti` or `openaev` |
| Variable `OPENCTI_CONFIG_HASH` | The configuration hash of the instance |

The container is named after the instance: its name in lower case, every character other than a letter or a digit replaced by `-`. On Kubernetes, the instance is a Deployment of that name with one replica when started and zero when stopped.

### What the container receives

The container gets the configuration of the instance as environment variables, plus the URL of each platform XTM Composer serves, `OPENCTI_URL` and `OPENAEV_URL`, taken from its own configuration. The integration therefore reaches the platform at the same URL as XTM Composer: in xtm-docker, `http://opencti:8080` and `http://openaev:8080` on the network of the deployment. When the proxy is on, it also gets the proxy variables: see [Run integrations behind a proxy](/deploy/integrations/proxy/).

## Secrets

The platform encrypts every secret setting of an instance, such as an API key, with the public key XTM Composer registered. Only XTM Composer can decrypt it, right before it creates the container.

Each encrypted value is Base64 text made of:

| Bytes | Content |
| --- | --- |
| 1 | Version: `1` for RSA PKCS#1 v1.5, `2` for RSA-OAEP with SHA-256 |
| 512 | A random AES-256 key and its 12-byte nonce, encrypted with the RSA public key |
| The rest | The value, encrypted with AES-256-GCM |

The 512 bytes are the size of a 4096-bit RSA block: XTM Composer needs a 4096-bit key. The FIPS build of XTM Composer refuses version `1`.

A value may fail to decrypt, for example when the key of XTM Composer changed after the platform encrypted it. XTM Composer then refuses to deploy or update the instance rather than start it with a blank secret. Its logs then say `Refusing to deploy connector`.

## What's next

- [Install XTM Composer](/deploy/integrations/install-composer/).
- [Configure XTM Composer](/deploy/integrations/configure-composer/), for the intervals and the container engine.
- [Troubleshoot XTM Composer](/deploy/integrations/troubleshoot/).
