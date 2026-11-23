---
title: Install offline or with FIPS 140-3
description: Install the XTM stack on a host without internet access, and run OpenCTI with its FIPS 140-3 images and compliant shared services.
doc_type: how-to
products:
  - suite
  - opencti
personas: [platform-engineer]
owner: docs-platform
last_reviewed: 2026-11-20
migrated_from:
  - opencti:/reference/fips/
---

This page installs the XTM stack on a host that cannot reach the internet, by carrying the images over from a connected machine, and lists what still needs an outside source. It then switches OpenCTI to its FIPS 140-3 images and says what the shared services need for a compliant stack. Read [Install with Docker Compose](/deploy/install/docker-compose/) first: this page changes only where the images come from and which ones run.

::diagram{id=deploy-install-air-gapped}

## Before you start

- A connected machine with Docker Compose v2 and Git, and a way to copy files from it to the target host.
- The same processor architecture on both machines: Intel or AMD (amd64). XTM One is published for amd64 only, and so are the FIPS images of OpenCTI.
- On the target host, Docker Engine 20.10 or later with Docker Compose v2, and the space for the images: about as much as `docker image ls` shows on the connected machine after the download.

## Install without internet access

:::steps
1. On the connected machine, get the deployment and configure it as [Install with Docker Compose](/deploy/install/docker-compose/) describes, up to the pinned versions: `docker-compose.override.yml` pins the images you will copy. For FIPS images, also follow [Switch OpenCTI to its FIPS images](#switch-opencti-to-its-fips-images) now.

   ```bash
   git clone https://github.com/FiligranHQ/xtm-docker.git
   cd xtm-docker
   ```

2. Download the images and save them in one archive.

   ```bash
   docker compose pull
   docker save -o xtm-images.tar $(docker compose config --images | sort -u)
   ```

   :::verify
   ```bash
   docker compose config --images | sort -u
   ```
   It lists every image of the stack, with the versions you pinned, and `xtm-images.tar` exists next to `docker-compose.yml`.
   :::

3. Copy the whole `xtm-docker` folder, with `.env`, `docker-compose.override.yml`, the `infra` and `modules` folders and `xtm-images.tar`, to the target host. `.env` holds every secret of the stack: copy it the way you copy secrets.

4. On the target host, load the images.

   ```bash
   cd xtm-docker
   docker load -i xtm-images.tar
   ```

5. Start the stack, without `docker compose pull`: Docker Compose uses the images it finds on the host.

   ```bash
   docker compose up -d
   ```

   :::verify
   ```bash
   docker compose ps --format '{{.Service}} {{.Health}}' | grep -E '^(opencti|openaev|xtm-one|opencrq) '
   ```
   `opencti`, `openaev`, `xtm-one` and `opencrq` say `healthy` after a few minutes. If `up` tries to download an image, an image is missing from the archive: repeat steps 2 to 4.
   :::
:::

To upgrade, repeat these steps with the new versions pinned in `docker-compose.override.yml`.

## What still needs an outside source

Once the images are on the host, the stack starts from them. Some features read from or deploy outside sources, and need a mirror, an on-premise service or a proxy instead:

| Feature | What it needs | Where to set it up |
| --- | --- | --- |
| Integrations deployed by XTM Composer | Pulls the image of each integration from a registry | [Pull integrations from a private registry](/deploy/integrations/private-registry/) |
| Integrations that read public sources, such as the NVD CVE collector | Access to their source | [Run integrations behind a proxy](/deploy/integrations/proxy/) |
| AI document extraction in OpenCTI | The Import Document AI connector calls a service hosted by Filigran by default | [Extract intelligence from documents on premises](/deploy/configure/ai-extraction/) |
| Certificates of internal services | Your internal certificate authority | [Trust your certificate authority and proxy](/deploy/configure/certificates/) |

:::only{opencti=">=7.260811"}
The OpenCTI map works offline: the platform ships its map tiles in the image. See [Map tiles](/deploy/configure/maps/).
:::

:::only{opencti="<7.260811"}
The OpenCTI map loads its tiles in the browser from `map.opencti.io`. On a network without internet access, run a tile server of your own and point OpenCTI to it, as [Map tiles](/deploy/configure/maps/) describes.
:::

## Switch OpenCTI to its FIPS images

The FIPS 140-3 images of OpenCTI are built on Alpine Linux with the OpenSSL FIPS provider enabled, from the [base image that Filigran maintains](https://github.com/FiligranHQ/docker-python-nodejs-fips). They exist for the platform, the worker and the connectors, with the tag of the version followed by `-fips`: `7.261030.1-fips`, or `7.260811.0-lts.1-fips` for a long-term support release. XTM Composer has a FIPS image too, which deploys the FIPS images of the connectors. OpenAEV and XTM One have no FIPS images.

:::steps
1. Pin the versions as [Pin the image versions](/deploy/install/docker-compose/#pin-the-image-versions) describes.

2. Switch every OpenCTI image to its `-fips` tag in `docker-compose.override.yml`. The Import External Reference connector has no FIPS image and keeps its tag. Run this command again each time you run the pinning commands: they write the tags without `-fips`.

   ```bash
   OPENCTI_VERSION=7.261030.1
   sed -i.bak -E \
     -e "s#image: (opencti/[a-z0-9-]+):${OPENCTI_VERSION}\$#image: \1:${OPENCTI_VERSION}-fips#" \
     -e "s#(connector-import-external-reference):${OPENCTI_VERSION}-fips#\1:${OPENCTI_VERSION}#" \
     docker-compose.override.yml
   ```

   :::verify
   ```bash
   docker compose config --images | grep '^opencti/'
   ```
   Every OpenCTI image ends with `-fips`, except `opencti/connector-import-external-reference`.
   :::

3. Switch XTM Composer to its FIPS image, so that the connectors you deploy from the catalog run their FIPS images. Add these lines under the `services:` key of `docker-compose.override.yml`, above the `# Pinned versions` line. The pinning commands do not change the image of XTM Composer, so this tag stays.

   ```yaml
     xtm-composer:
       image: filigran/xtm-composer:3.261120.0-fips
   ```

   The FIPS image of XTM Composer sets `manager.image_tag_suffix` (`MANAGER__IMAGE_TAG_SUFFIX`) to `-fips`: it adds `-fips` to the tag of every integration it deploys. `opencti/connector-mitre:7.261030.1` runs as `opencti/connector-mitre:7.261030.1-fips`, and an image without a tag as `latest-fips`. A tag that already ends with `-fips`, or an image pinned by digest, stays as it is. On a host without internet access, the registry XTM Composer pulls from must hold these `-fips` tags: see [Pull integrations from a private registry](/deploy/integrations/private-registry/).

   :::verify
   ```bash
   docker compose config --images | grep xtm-composer
   ```
   It prints `filigran/xtm-composer:3.261120.0-fips`. Once the stack runs, `docker compose logs xtm-composer | grep 'Starting XTM composer'` shows `fips` set to `true` and `image_tag_suffix` set to `-fips`.
   :::

   :::warning[Integrations without a FIPS image]
   XTM Composer adds the suffix to every integration it deploys, for OpenAEV as for OpenCTI. An integration with no image under the `-fips` tag, such as the Import External Reference connector, is not deployed: its **Logs** tab says `[XTM Composer] Connector not deployed`, then that the image cannot be pulled, with the error of the registry. XTM Composer tries again at each pass.
   :::

4. For each connector you run outside XTM Composer, use its `-fips` tag when Docker Hub lists one, such as [the FIPS tags of the STIX file export connector](https://hub.docker.com/r/opencti/connector-export-file-stix/tags?name=fips).
:::

### What the images guarantee

All cryptography in the OpenCTI FIPS images goes through the OpenSSL FIPS provider 3.1.2, the module validated under CMVP certificate #4985, valid until 10 March 2030. FIPS mode is on by default: no flag, variable or configuration step is needed.

- The module is built from the validated source distribution, unmodified, by the procedure of its Security Policy, with integrity verification and the required self-tests enabled.
- FIPS mode is enforced: non-approved algorithms are refused, not substituted.

The full posture is in [FIPS.md](https://github.com/FiligranHQ/docker-python-nodejs-fips/blob/main/FIPS.md), in the base image repository.

## Make the shared services compliant

The FIPS images cover OpenCTI. For a compliant stack, the shared services must use FIPS 140-3 cryptography too.

| Service | How |
| --- | --- |
| Elasticsearch | FIPS 140-3 from version 9.4, and from 8.19.15 in the 8.x line, with the Bouncy Castle FIPS 2.x libraries as Java security provider. Earlier versions support FIPS 140-2 with Bouncy Castle 1.x. Follow the [Elastic FIPS guide](https://www.elastic.co/docs/deploy-manage/security/fips-es), or use the [Elastic FedRAMP offering](https://www.elastic.co/industries/public-sector/fedramp). |
| Redis | No FIPS images, but [custom TLS cipher suites](https://github.com/redis/redis/issues/7802) that can use the system OpenSSL in FIPS mode. Or put a [Stunnel](https://www.stunnel.org/) TLS endpoint between OpenCTI and Redis. |
| RabbitMQ | No FIPS images, but [custom cipher suites](https://www.rabbitmq.com/docs/ssl#cipher-suites); since RabbitMQ 3.12.5, its Erlang build (26.1 and later) [supports FIPS mode on OpenSSL 3](https://www.rabbitmq.com/docs/which-erlang). Or use a Stunnel TLS endpoint. |
| S3 object storage | Prefer an endpoint that is already validated, such as the [AWS S3 FIPS endpoints](https://aws.amazon.com/compliance/fips/). xtm-docker runs RustFS: check its builds against your FIPS requirements before you rely on them. The last MinIO FIPS images, on [Quay.io](https://quay.io/repository/minio/minio?tab=tags) with tags ending in `.fips`, are no longer maintained: check them against your requirements first. |

The Elasticsearch of xtm-docker, 8.19.18, is in the FIPS 140-3 range, but runs with security off: enable FIPS mode as the Elastic guide describes.

:::tip[AWS native services]
OpenCTI works with AWS native services, available in the [FedRAMP Moderate and FedRAMP High scopes](https://aws.amazon.com/compliance/services-in-scope/FedRAMP/): Amazon OpenSearch Service, Amazon ElastiCache for Redis, Amazon MQ for RabbitMQ and Amazon S3.
:::

## Check what the OpenCTI images contain

Each OpenCTI release publishes a software bill of materials (SBOM) for each variant of the platform and worker images, in the assets of its [GitHub release](https://github.com/OpenCTI-Platform/opencti/releases). Use it in the security review of the images you carry over, or to check whether a vulnerable library is in the version you run.

| Image variant | Image tag | SBOM asset |
| --- | --- | --- |
| Standard | `7.261030.1` | `sbom-opencti-platform-7.261030.1.cdx.json` |
| FIPS 140-3 | `7.261030.1-fips` | `sbom-opencti-platform_fips-7.261030.1.cdx.json` |
| Red Hat Universal Base Image 9 | `7.261030.1-ubi9` | `sbom-opencti-platform_ubi9-7.261030.1.cdx.json` |

Each file is in the CycloneDX JSON format. Its top component is OpenCTI, with one child component for each part, each with its own dependencies: `filigran/platform`, the platform image, with its Alpine and Python packages; `opencti-front`, the web interface; `opencti-graphql`, the GraphQL API; and `filigran/worker`, the worker image. The connectors are not in it: each has its own image.

::::only{opencti=">=7.261030"}
:::warning[Incomplete SBOM in 7.261030.1]
In the SBOMs of 7.261030.1, `opencti-front` and `opencti-graphql` list no dependencies: the JavaScript libraries of the platform are missing. The SBOMs of the long-term support releases list them.
:::
::::

To count the dependencies of each part, then find the version of a library, here the Python library `stix2`:

```bash
OPENCTI_VERSION=7.261030.1
curl -fsSLO "https://github.com/OpenCTI-Platform/opencti/releases/download/${OPENCTI_VERSION}/sbom-opencti-platform-${OPENCTI_VERSION}.cdx.json"
jq -r '.components[] | "\(.name): \((.components // []) | length)"' "sbom-opencti-platform-${OPENCTI_VERSION}.cdx.json"
jq -r '[.. | objects | select(.name? == "stix2") | "\(.name) \(.version)"] | unique[]' "sbom-opencti-platform-${OPENCTI_VERSION}.cdx.json"
```

:::verify
The first command prints one line for each part, such as `filigran/worker: 122`. The second prints `stix2 3.0.2` for 7.261030.1.
:::

## What's next

- [Publish the platforms over HTTPS](/deploy/install/reverse-proxy/), to publish the platforms over HTTPS.
- [Pull integrations from a private registry](/deploy/integrations/private-registry/), to deploy integrations from your own registry.
- [Upgrade](/deploy/upgrade/), to plan the next versions you will carry over.
