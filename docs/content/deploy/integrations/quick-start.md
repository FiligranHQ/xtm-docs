---
title: Deploy a first integration with XTM Composer
description: Check that XTM Composer runs in the XTM stack and that OpenCTI and OpenAEV see it, then deploy a first integration from the interface.
doc_type: tutorial
products:
  - composer
  - opencti
  - openaev
personas: [platform-engineer, evaluator]
owner: docs-platform
last_reviewed: 2026-10-31
edition: ee
migrated_from:
  - opencti:/deployment/integration-manager/quick-start/
  - openaev:/deployment/ecosystem/integration-manager/quick-start/
---

This tutorial checks XTM Composer in a running XTM stack and deploys a first integration with it, in about ten minutes. At the end, OpenCTI and OpenAEV see XTM Composer, and you have deployed a first integration from the interface of a platform, which XTM Composer runs as a container.

::diagram{id=deploy-integrations-quick-start}

It uses [FiligranHQ/xtm-docker](https://github.com/FiligranHQ/xtm-docker), where XTM Composer is part of the stack: it starts with the rest, generates its own RSA key and serves both OpenCTI and OpenAEV. To run XTM Composer outside xtm-docker, see [Install XTM Composer](/deploy/integrations/install-composer/).

## What you need

- The XTM stack running from xtm-docker. If you do not have it yet, follow [Run the XTM platform locally](/deploy/quick-start/): it starts XTM Composer with the rest of the stack.
- An Enterprise Edition license for each platform you deploy integrations on.
- A terminal in the `xtm-docker` folder.

## Step 1: Check that XTM Composer is in the stack

::::steps
1. List the services of XTM Composer in the stack.

   ```bash
   docker compose config --services | grep -E '^(xtm-composer|rsa-key-generator)$'
   ```

   :::verify
   It lists `rsa-key-generator` and `xtm-composer`. If it does not, your copy of xtm-docker is older: get the current one with `git pull`, then run the command again.
   :::
::::

## Step 2: Start XTM Composer

::::steps
1. Download the current images, then start what is not running yet.

   ```bash
   docker compose pull
   docker compose up -d
   ```

   `rsa-key-generator` creates a 4096-bit key in the `rsakeys` volume the first time, then XTM Composer starts once OpenCTI, OpenAEV and RabbitMQ are healthy.

2. Check that XTM Composer loaded its key and registered with both platforms.

   ```bash
   docker compose logs xtm-composer | grep -E 'Successfully loaded RSA private key|Manager registered'
   ```

   :::verify
   One line says `Successfully loaded RSA private key (PKCS#8 format)`, and one `Manager registered` line appears for each platform, OpenCTI and OpenAEV. Registration is retried every minute: if a line is missing, wait a minute and run the command again, then see [Troubleshoot XTM Composer](/deploy/integrations/troubleshoot/).
   :::
::::

## Step 3: Check the platforms see it

Sign in to each platform and open its catalog: :menu[Integrations], **Available** tab (:menu[Data > Ingestion > Connector catalog] before 7.260811) in OpenCTI, **Integrations** > **Catalog** in OpenAEV.

:::verify
The catalog no longer says that deploying requires the installation of the Integration Manager. If it says that deployment requires an Enterprise Edition license, enter your license first.
:::

## Step 4: Deploy a first integration

::::steps
1. In the catalog, click **Deploy** on an integration whose settings you have, for example one that needs no API key.
2. Fill the form and click **Create**. The instance opens, stopped.
   - In OpenCTI: a **Display name**, from which OpenCTI derives the **Instance name**, and the account the connector writes as, with **Automatically create a service account** or an existing **Service account responsible for data creation**.
   - In OpenAEV: a unique **Instance name**.
3. Click **Start**.

   :::verify
   On the host, XTM Composer has created a container for the instance, named after the instance name in lower case, with the label `xtm-manager` it puts on every container it manages:

   ```bash
   docker ps --filter label=xtm-manager --format '{{.Names}} {{.Status}}'
   ```
   The instance is `Up`, and its **Logs** tab in the platform fills within a few seconds. XTM Composer 3.261021.1 and earlier label the container `opencti-manager` only. If no container appears and the **Logs** tab says that the image cannot be pulled, see [The image cannot be pulled](/deploy/integrations/troubleshoot/#the-image-cannot-be-pulled).
   :::
::::

## What you learned

- [XTM Composer](/deploy/integrations/composer/) runs the integrations you deploy from the interface of OpenCTI and OpenAEV.
- [XTM Composer architecture](/deploy/integrations/architecture/): it registers with a public key, then reconciles every few seconds what the platform asks with what runs.
- [Configure XTM Composer](/deploy/integrations/configure-composer/) lists the settings you can change, such as the network its integrations join.
