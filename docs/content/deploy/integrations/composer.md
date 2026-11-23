---
title: XTM Composer
description: XTM Composer deploys and runs the integrations of OpenCTI and OpenAEV as containers, so you manage them from the platform interface.
doc_type: explanation
products:
  - composer
  - opencti
  - openaev
personas: [platform-engineer, admin]
owner: docs-platform
last_reviewed: 2026-11-20
edition: ee
migrated_from:
  - opencti:/deployment/integration-manager/
  - openaev:/deployment/ecosystem/integration-manager/overview/
---

XTM Composer is the component that deploys integrations for OpenCTI and OpenAEV. You pick an integration in the catalog of a platform and fill its form. XTM Composer starts it as a container on your Docker, Podman or Kubernetes, reports its status and logs, and applies your changes. This page explains what it does, what it needs, and where to go next to install it.

::diagram{id=deploy-integrations-composer}

In the interface of both platforms, XTM Composer appears as the **Integration Manager**.

## What it runs

| Platform | What XTM Composer deploys | Where you deploy from |
| --- | --- | --- |
| OpenCTI | Connectors | :menu[Integrations], **Available** tab (:menu[Data > Ingestion > Connector catalog] before 7.260811) |
| OpenAEV | External collectors, injectors and executors | **Integrations** > **Catalog** |

One XTM Composer can serve OpenCTI, OpenAEV or both: each platform has its own block in its configuration (`opencti` and `openaev`), switched on or off with `enable`. The integrations built into a platform, such as the email injector of OpenAEV, do not need it. See [How integrations run](/deploy/integrations/how-they-run/).

XTM Composer runs the image the catalog names for each integration. To run the FIPS 140-3 or Red Hat Universal Base Image 9 builds of the integrations instead, `manager.image_tag_suffix` adds `-fips` or `-ubi9` to the tag of every image it deploys. The FIPS image of XTM Composer sets `-fips`. See [Configure XTM Composer](/deploy/integrations/configure-composer/).

## What it needs

- An **Enterprise Edition** license on the platform. Without it, the catalog is read-only. You can enter the license from the **Deploy** action of a catalog card, from the detail view of an integration, or from the platform settings.
- OpenCTI 6.8.17 or later. Every OpenCTI version documented here qualifies.
- A container engine XTM Composer can drive: Docker, Podman, Kubernetes, Docker Swarm, or Docker environments in Portainer.
- A 4096-bit RSA private key, which XTM Composer keeps and whose public key it gives to the platform. The platform encrypts the secrets of each integration with it.
- An API token for each platform it serves.

## XTM Composer in the XTM stack

In [FiligranHQ/xtm-docker](https://github.com/FiligranHQ/xtm-docker), XTM Composer is part of the stack and always starts with it. `modules/xtm-composer.yml` defines two services: `xtm-composer`, and `rsa-key-generator`, which creates the RSA key once in the `rsakeys` volume. XTM Composer always serves both OpenCTI and OpenAEV: the service sets `OPENCTI__ENABLE` and `OPENAEV__ENABLE` to `true`. It starts once the key exists and OpenCTI, OpenAEV and RabbitMQ are healthy.

A `.env` written for an earlier version of xtm-docker may still set `XTM_COMPOSER_OPENCTI_ENABLE` or `XTM_COMPOSER_OPENAEV_ENABLE`. They no longer do anything: remove them.

The service connects to `http://opencti:8080` and `http://openaev:8080` with the admin token of each platform, drives the Docker daemon through `/var/run/docker.sock`, and attaches the integrations it starts to the network of the deployment. [Deploy a first integration with XTM Composer](/deploy/integrations/quick-start/) checks that it runs and deploys a first integration.

## Deploy an integration from the interface

Once XTM Composer is running, the catalog no longer warns that deployment requires the Integration Manager.

:::::tabs
::::tab[OpenCTI]
:::steps
1. Open :menu[Integrations], **Available** tab (:menu[Data > Ingestion > Connector catalog] before 7.260811) and find the connector. A badge on **Deploy** shows that it is already deployed.
2. Click **Deploy** and fill the form: a **Display name**, from which OpenCTI derives the **Instance name** that names the container, the account the connector writes as, with **Automatically create a service account** or an existing **Service account responsible for data creation**, and the settings of the connector under **Configuration**. **Advanced options** shows the optional ones. The header of the form counts the **instances deployed** by XTM Composer for this connector; click it to list them.
3. Click **Create**. OpenCTI opens the new instance. It is not started yet.
4. Click **Start**. The **Logs** tab shows what the connector writes, at the log level you chose.
:::

OpenCTI refuses an instance name or a service account name that is already used by another instance or user. You can deploy several instances of the same connector with different names, but each one adds load to the platform: avoid it unless you need it.
::::
::::tab[OpenAEV]
:::steps
1. Open **Integrations** > **Catalog** and find the collector, injector or executor.
2. Click **Deploy** and fill the form: a unique **Instance name**, the log level and the settings of the integration. **Advanced options** shows the optional ones.
3. Click **Create**. OpenAEV opens the new instance. It is not started yet.
4. Click **Start**. The **Logs** tab shows what the integration writes.
:::

The deployed instances are listed in **Integrations** > **Collectors**, **Injectors** and **Executors**.
::::
:::::

## What you manage from the interface

- Only the instances XTM Composer manages can be started and stopped from the interface, and only they show their logs there.
- You can update the configuration of a managed instance with **Update**. XTM Composer applies it within a few seconds by recreating the container, or by updating the Deployment on Kubernetes.
- When the image of an integration cannot be pulled, XTM Composer deploys nothing, or keeps the container that runs with its previous configuration, and says so in the **Logs** tab: `[XTM Composer] Connector not deployed` or `[XTM Composer] Connector not updated, the previous deployment is kept`, with the image and the error. It tries again at each pass. See [Troubleshoot XTM Composer](/deploy/integrations/troubleshoot/).
- Secrets such as API keys are encrypted when you save them and never shown again.
- An integration you deployed by hand can be taken over by XTM Composer: see [Migrate to XTM Composer](/deploy/integrations/migrate/).

## Add your own connectors to the OpenCTI catalog

The OpenCTI catalog lists the connectors of Filigran. To offer connectors you build yourself, list more catalogs in `app:custom_catalogs` (`APP__CUSTOM_CATALOGS`), as a JSON array of file paths the platform can read, inside its container in Docker. Each file is a JSON catalog in the format of the built-in one, with a `contracts` list. XTM Composer can deploy a contract that has `manager_supported` set to `true` and a `config_schema`.

:::only{opencti=">=7.261030"}
An entry can also be an `http://` or `https://` URL, which OpenCTI downloads. The catalog manager loads the catalogs: its keys are in the [Engines, Schedules and Managers](/reference/opencti/configuration/#engines-schedules-and-managers) section of the configuration reference.
:::

```yaml
services:
  opencti:
    environment:
      - 'APP__CUSTOM_CATALOGS=["/opt/opencti/catalogs/example-cert.json"]'
    volumes:
      - ./catalogs:/opt/opencti/catalogs:ro
```

Filigran may remove this key in a later version: check with your Filigran contact before you build on it.

## What's next

- [Deploy a first integration with XTM Composer](/deploy/integrations/quick-start/), to turn it on in the XTM stack.
- [Install XTM Composer](/deploy/integrations/install-composer/), on Docker, Podman or Kubernetes outside xtm-docker.
- [XTM Composer architecture](/deploy/integrations/architecture/), how it registers, reconciles and decrypts.
- [Configure XTM Composer](/deploy/integrations/configure-composer/), [Run integrations behind a proxy](/deploy/integrations/proxy/) and [Pull integrations from a private registry](/deploy/integrations/private-registry/).
- [Troubleshoot XTM Composer](/deploy/integrations/troubleshoot/).
