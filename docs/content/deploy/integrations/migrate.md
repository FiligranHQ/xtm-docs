---
title: Migrate to XTM Composer
description: Move connectors, collectors and injectors you deployed by hand under the management of XTM Composer.
doc_type: how-to
products:
  - composer
  - opencti
  - openaev
personas: [platform-engineer, admin]
owner: docs-platform
last_reviewed: 2026-10-31
edition: ee
migrated_from:
  - opencti:/deployment/integration-manager/migration/
---

This page moves an integration you run yourself, as a Compose service or a Python program, under XTM Composer, so you start, stop, update and read its logs from the platform interface. OpenCTI replaces the connector with a new managed instance; OpenAEV takes over the existing collector or injector. Read [XTM Composer](/deploy/integrations/composer/) first if you are new to it.

The [default integrations of xtm-docker](/deploy/integrations/how-they-run/#the-default-integrations-of-xtm-docker) are part of the stack and always start with it, so this page does not apply to them: XTM Composer would run a second copy next to them.

::diagram{id=deploy-integrations-migrate}

## Before you start

- XTM Composer running and registered with the platform: see [Deploy a first integration with XTM Composer](/deploy/integrations/quick-start/).
- An Enterprise Edition license on the platform.
- In OpenCTI, the **Manage ingestion** capability. In OpenAEV, the permission to manage the platform settings.
- The current configuration of the integration: its environment variables in your Compose file, or its `config.yml`. You need its settings and credentials again.

## Move the integration

::::::tabs
:::::tab[OpenCTI]
:::warning
OpenCTI creates a new connector instance. The state of the old one, such as its last run and its work in progress, is not carried over: the new instance starts from the beginning.
:::

::::steps
1. Open :menu[Integrations], **Available** tab (:menu[Data > Ingestion > Connector catalog] before 7.260811) and find the connector. A connector that is not in the catalog, or that the catalog shows without **Deploy**, cannot be managed by XTM Composer: keep running it by hand.
2. Find the settings the connector needs. The connector page links to its code: its `__metadata__` folder has a `CONNECTOR_CONFIG_DOC.md` file that lists them. Match them with the values of your current configuration.
3. Click **Deploy**, fill the form with those values and a new **Display name**, which gives the **Instance name**, then click **Create**.
4. Click **Start** and wait for the status to change.

   :::verify
   The new instance appears in :menu[Integrations], **Deployed** tab (:menu[Data > Ingestion > Connectors] before 7.260811) and its **Logs** tab fills.
   :::

5. Stop and remove the old connector, as described in [Remove the old container](#remove-the-old-container), so it does not run twice.
::::
:::::
:::::tab[OpenAEV]
OpenAEV registers the collector or injector again under XTM Composer, and keeps its identifier.

::::steps
1. Open **Integrations** > **Collectors** or **Injectors**, or the detail page of the collector or injector. A **Migrate** button shows when the integration is external, has no managed instance yet, has an entry in the catalog, and XTM Composer runs. On the detail page, it also needs the permission to manage the platform settings. Without Enterprise Edition, the button asks for a license.
2. Click **Migrate**. A drawer opens with the same form as a new deployment.
3. Fill the settings with the values of your current configuration, then click **Migrate**.

   :::verify
   OpenAEV opens the new managed instance. You can start and stop it and read its logs like any deployed instance.
   :::

4. Stop and remove the old container, as described in [Remove the old container](#remove-the-old-container). If a service provider runs your OpenAEV, ask them to do it.
::::
:::::
::::::

## Remove the old container

Remove the integration where you declared it, or it starts again at the next `docker compose up -d`.

::::tabs
:::tab[Your own Compose file]
Delete the service of the integration from your Compose file, your own `docker-compose.yml` or the `docker-compose.override.yml` of xtm-docker, then remove its container:

```bash
docker compose up -d --remove-orphans
```
:::
:::tab[A Python program]
Stop the process and remove it from whatever starts it, such as a systemd unit.
:::
::::

:::verify
```bash
docker compose ps --format '{{.Service}}' | grep -E '^(connector|collector|injector)-'
```
The integrations you moved are no longer listed. The default integrations of xtm-docker still are.
:::

## What's next

- [XTM Composer](/deploy/integrations/composer/), what you manage from the interface.
- [Troubleshoot XTM Composer](/deploy/integrations/troubleshoot/).
