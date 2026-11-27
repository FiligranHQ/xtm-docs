---
title: XTM Composer architecture
description: How XTM Composer registers with OpenCTI and OpenAEV, reconciles their integrations with the containers it runs, follows new images on moving tags, and decrypts their secrets.
doc_type: explanation
products:
  - composer
  - opencti
  - openaev
personas: [platform-engineer]
owner: docs-platform
last_reviewed: 2026-11-27
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

At start, whatever platform it serves, XTM Composer loads its RSA private key and stops if the key is missing or is not a PKCS#8 PEM file. With its key loaded, it registers with each platform under `manager.id` and `manager.name`, and sends its public key: as a PKCS#1 PEM document to OpenCTI, as a SubjectPublicKeyInfo PEM document to OpenAEV.

To OpenAEV, XTM Composer also sends its own version and, when `manager.image` is set, the image it runs from, such as `filigran/xtm-composer:3.261127.0`. It sends them at registration and again at each ping, so OpenAEV keeps them current after an upgrade. OpenAEV 3.261126.1 and later keep them; earlier versions ignore them.

The platform keeps the identifier and the public key. OpenAEV keeps a single XTM Composer: each registration replaces the previous one, and the pings of a replaced XTM Composer are refused. Run one XTM Composer per OpenAEV. OpenAEV considers XTM Composer unreachable when it has not pinged for two hours.

Keep `manager.id` and the key stable: they are how the platform recognizes XTM Composer and encrypts for it.

## Reconciliation

Every `execute_schedule` seconds, XTM Composer fetches the integration instances of the platform. For each one it looks for a container of the same name, then:

| Situation | What XTM Composer does |
| --- | --- |
| No container | Pulls the image and creates the container, and starts it when the requested state is started. It reports the instance as stopped, then as started at the next pass once the container runs |
| The requested state is started and the container is stopped | Starts it |
| The requested state is stopped and the container runs | Stops it |
| The configuration hash of the instance differs from the one of the container | Pulls the image, then removes the container and creates it again with the new configuration |
| The container runs | Reports its health, its restart count and its start date. Each instance reports every 30 seconds, and at once when it was never reported, has just started, was recreated or restarted. To OpenAEV, it also reports the `sha256:` digest of the image the container runs, and how the previous container ended |
| The container runs on a moving tag, and `manager.image_update_interval` seconds have passed, 600 by default | Reads the digest the tag points to in the registry, and redeploys the instance when it changed: see [When the tag points to a new image](#when-the-tag-points-to-a-new-image) |
| `logs_schedule` seconds have passed, 10 by default | Sends the latest lines of its logs to the platform: see [What the logs show](#what-the-logs-show) |
| A container of that name belongs to another instance | Leaves it alone and writes in the logs of the instance that the name is taken |

Then it removes the containers it manages for this platform that no longer match an instance, or whose name changed.

A container counts as in a reboot loop when it has restarted more than three times and has been up for less than five minutes. XTM Composer reports it to the platform with its health.

### When the image cannot be pulled

XTM Composer pulls the image of an instance before it deploys it or applies a new configuration. On Kubernetes, the nodes pull the image: XTM Composer reads the pull failures from the pods instead. A missing image never replaces a running integration.

| Situation | What XTM Composer does |
| --- | --- |
| The pull fails before a first deployment | Deploys nothing and reports the instance as stopped. The logs of the instance say `[XTM Composer] Connector not deployed`, followed by the image and the error |
| The pull fails before a new configuration is applied | Keeps the container that runs, with its previous configuration. The logs of the instance say `[XTM Composer] Connector not updated, the previous deployment is kept`, followed by the image and the error |
| On Kubernetes, a pod waits for its image (`ErrImagePull`, `ImagePullBackOff`, `InvalidImageName`, `ErrImageNeverPull`) | With no pod running, reports the instance as stopped, does not start it again and writes `Connector waiting for its image` in its logs. When the previous pod still runs, writes `Connector not updated, the previous deployment is kept`. Stopping the instance still scales the Deployment down |

XTM Composer tries again at each pass, so the instance deploys or updates once the image is in the registry. Its own logs say `Cannot pull the workload image`. See [Troubleshoot XTM Composer](/deploy/integrations/troubleshoot/) for the usual causes.

### When the tag points to a new image

The configuration hash changes only with the configuration. An image republished under the same tag would never reach a running instance, so XTM Composer watches the instances on a moving tag: `latest`, `rolling` or no tag at all. A `-fips` or `-ubi9` suffix does not change that: `rolling-fips` moves. An image with a version tag or an `@sha256:` digest never moves, and XTM Composer never looks it up.

OpenAEV deploys its catalog images on the `rolling` tag, so OpenAEV integrations follow each new image. The OpenCTI catalog names images on the platform version, such as `opencti/connector-mitre:7.261030.1`: they stay pinned.

XTM Composer looks up an instance when it runs, its configuration is unchanged, its last pull did not fail, every secret decrypted, and the digest of the image it runs is known. The first lookup happens at once, then at most every `manager.image_update_interval` seconds, 600 by default. `0` turns the lookups off.

A lookup is not a pull, and Docker Hub does not count it as one. The XTM Composer process itself sends a `HEAD` request for the manifest of the tag, over HTTPS, with a 15-second timeout:

| Registry | Where the request goes |
| --- | --- |
| Docker Hub | `registry-1.docker.io`, with a token from `auth.docker.io` |
| The registry of `daemon.registry` | That registry, with its credentials and the image tag suffix |

The XTM Composer container or pod therefore needs HTTPS access to the registry, not only the container engine. A registry served over plain HTTP cannot be looked up. The request follows the `HTTPS_PROXY` and `NO_PROXY` variables of the XTM Composer process and trusts the system's certificate authorities. The proxy settings of the `opencti` and `openaev` blocks apply to the platforms only: see [Run integrations behind a proxy](/deploy/integrations/proxy/).

When the tag points to a digest other than the running one, XTM Composer writes `New image pushed under the workload tag, redeploying` in its own logs, then:

| Engine | What XTM Composer does |
| --- | --- |
| Docker, Podman, Docker Swarm, Portainer | Pulls the tag first. When the pull fails, the running instance stays and its logs say `[XTM Composer] Connector not updated, the previous deployment is kept`. Otherwise it creates the container or the service again from the pulled image |
| Kubernetes | Patches the Deployment: the image becomes `<image>@<digest>`, and the pod template gets the annotation `xtm-image-digest`. The pod rolls whatever its `imagePullPolicy`. A node that cannot pull the image shows as described in [When the image cannot be pulled](#when-the-image-cannot-be-pulled) |

The logs of the instance then say `[XTM Composer] Image updated:`, followed by the previous digest and the new one. When the lookup fails, because the registry is unreachable, refuses the credentials or gives no digest, the instance keeps running its image. The logs of XTM Composer then say `Cannot read the digest of the workload image in its registry, the running workload is kept`, with the error.

### What the logs show

Every `logs_schedule` seconds, XTM Composer sends the platform the last 100 lines of the integration container, shown in the **Logs** tab of the instance. It adds lines of its own, prefixed with `[XTM Composer]`, after them.

When the previous container crashed or restarted, the logs end with how it ended: `[XTM Composer] The previous container ended: reason OOMKilled, exit code 137, at 2026-11-26T10:00:00Z`, for example. XTM Composer writes it when the container restarted at least once, or when its exit code is not `0`. A clean stop without a restart is not reported.

| Engine | Where the end of the previous container comes from |
| --- | --- |
| Kubernetes | The last termination of the integration container in its pod |
| Docker, Podman, Portainer | The state of the container, while it does not run: Docker clears it when the container starts again. The reason is `OOMKilled`, the error of the engine, `Error` or `Completed` |
| Docker Swarm | Not read |

On Kubernetes, the logs of a restarted container or of a replaced pod start with up to the last 50 lines of the previous one. They come under `[XTM Composer] Last lines of the previous container:` or `pod:`, and the current lines under `[XTM Composer] Logs of the current container:` or `pod:`. XTM Composer keeps the lines of a replaced pod in memory, from the time it started.

### Runtime and logs on Kubernetes

During a rolling update, the Deployment of an instance may have a new pod next to the one that runs. XTM Composer reads the restart count, the start date, the image digest, the last termination and the logs from a pod whose integration container runs, the newest one when there are several. It sends the last 100 lines of the integration container only, not those of its sidecars. When no pod answers, the logs of the instance say `[XTM Composer] Cannot read the connector logs`, followed by the error.

### How it marks its containers

XTM Composer lists and cleans up only the containers whose `xtm-manager` or `opencti-manager` label is its own `manager.id`. Every container it creates carries each key twice, under its `xtm-` name and under its former `opencti-` name:

| Label or variable | Former name, still written | Value |
| --- | --- | --- |
| Label `xtm-manager` | `opencti-manager` | `manager.id` |
| Label `xtm-workload-id` | `opencti-connector-id` | The ID of the instance in the platform |
| Label `xtm-platform` | `opencti-platform` | `opencti` or `openaev` |
| Variable `XTM_CONFIG_HASH` | `OPENCTI_CONFIG_HASH` | The configuration hash of the instance. On Kubernetes, an annotation of the Deployment |

The former names keep a rollback to an earlier XTM Composer working, and keep the containers deployed by earlier versions recognized: XTM Composer reads the `xtm-` labels first, then the former ones. A later version will stop writing the former names: filter on `xtm-manager` in new scripts and monitoring. A command such as `docker ps --filter label=opencti-manager` still works in this version.

The container is named after the instance: its name in lower case, every character other than a letter or a digit replaced by `-`. On Docker, Podman and Portainer, that is the name of the container, whatever its length.

On Kubernetes and Docker Swarm, the name must be a DNS label (RFC 1123): lowercase ASCII letters, digits and `-`, no `-` at either end, 63 characters at most. XTM Composer keeps the container name when it already is one. Otherwise, for example when the name is longer than 63 characters or holds an accented letter, it builds a label from the instance name:

1. It puts the name in lower case, replaces every character other than an ASCII letter or digit with `-`, and removes the `-` at both ends.
2. It keeps the first 54 characters, without a final `-`.
3. It appends `-` and the first 8 hexadecimal characters of the SHA-256 hash of the full instance name.

The label is the same at every pass, and two long names that share their first characters still get two labels. OpenAEV names its instances after the catalog title and the instance ID, so their names often take the suffix: `DNS Enumeration (subdomains and DNS records)-7653e749-8c9d-4723-a656-84f3b9fda3ab` becomes `dns-enumeration--subdomains-and-dns-records--7653e749-` followed by 8 hexadecimal characters. A name such as `Import File (STIX) v2` stays `import-file--stix--v2`. This rule arrives with XTM Composer 3.261121.0. It does not rename or redeploy the instances that already run: their names were valid labels, since Kubernetes and Docker Swarm refuse any other. An instance whose name was refused before now deploys at the next pass.

On Kubernetes, the instance is a Deployment of that name, whose integration container has the same name, with one replica when started and zero when stopped. On Docker Swarm, it is a service of that name. To find the Deployment or the service of an instance without computing its name, filter on its `xtm-workload-id` label.

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

A value may fail to decrypt, for example when the key of XTM Composer changed after the platform encrypted it. XTM Composer then refuses to deploy or update the instance rather than start it with a blank secret. Its logs then say `Refusing to deploy connector`, or `Refusing to refresh connector` for an update, and the running container is kept.

## What's next

- [Install XTM Composer](/deploy/integrations/install-composer/).
- [Configure XTM Composer](/deploy/integrations/configure-composer/), for the intervals and the container engine.
- [Troubleshoot XTM Composer](/deploy/integrations/troubleshoot/).
