---
title: Deploy and operate
description: Deploy the whole XTM platform as one stack, then run, scale and upgrade
  it.
doc_type: landing
hide_table_of_contents: true
sidebar_label: Overview
products:
  - suite
sidebar_position: 0
---

:::product-hero{eyebrow="One deployment for the whole platform" title="Deploy the whole XTM platform" primary-label="Run it locally" primary-to="/deploy/quick-start/" secondary-label="Deploy with an AI agent" secondary-to="/deploy/quick-start/with-an-ai-agent/" diagram=deploy-overview}
XTM is deployed as one stack around XTM One: OpenCTI, OpenAEV, OpenCRQ and XTM Composer always run with it and share Elasticsearch, Redis, RabbitMQ, S3 storage and PostgreSQL. No module is optional. These pages follow the life of a deployment, from a first try on your machine to production, operations and upgrades.
:::

## Start here

Pick where you are: each tab gives your goal and the pages to read first, in order.

::::tabs
:::tab[Try it]
You want the platform running on one machine today, to see what it does.

1. [Run the XTM platform locally](/deploy/quick-start/): XTM One, OpenCTI, OpenAEV and XTM Composer on one machine with Docker Compose.
2. [Deploy with an AI agent](/deploy/quick-start/with-an-ai-agent/): let a coding agent do it, from a prompt and steps it can check.
3. [What each XTM module brings](/deploy/plan/modules/): the services each module runs and the settings it needs.
:::
:::tab[Go to production]
You prepare a deployment your teams will rely on.

1. [Requirements](/deploy/plan/requirements/): what the host, the shared services and the runtimes must provide.
2. [Sizing](/deploy/plan/sizing/): how much CPU, memory and disk the stack needs.
3. [Topologies: one host, a cluster, Kubernetes](/deploy/plan/topologies/): where the modules and the shared services can run.
4. [Install with Docker Compose](/deploy/install/docker-compose/): secrets, public URLs, persistent data and pinned versions.
5. [Publish the platforms over HTTPS](/deploy/install/reverse-proxy/): the three platforms behind your reverse proxy.
6. [Sign in the first time and keep an emergency account](/deploy/configure/first-access/): the first administrator, and an account that works when single sign-on fails.
:::
:::tab[Add integrations]
You connect the platforms to your sources and your security tools.

1. [How integrations run](/deploy/integrations/how-they-run/): connectors, collectors, injectors and executors, and how they reach their platform.
2. [XTM Composer](/deploy/integrations/composer/): deploy and run them as containers, from the platform interface.
3. [Deploy a first integration with XTM Composer](/deploy/integrations/quick-start/): check that the platforms see it, then deploy one.
:::
:::tab[Run it]
You keep the deployment healthy, safe and recoverable.

1. [Check that the platforms are healthy](/deploy/operate/health-probes/): the health endpoints and what each answer means.
2. [Back up and restore](/deploy/operate/backup/): every store of the stack together, and a restore on an empty deployment.
3. [Harden the deployment](/deploy/operate/harden/): close the shared services to the network and tighten the settings.
4. [Troubleshoot the XTM stack](/deploy/operate/troubleshoot/): where the logs are, and the common startup and ingestion errors.
:::
:::tab[Upgrade]
You move to a new release without surprise.

1. [Upgrade](/deploy/upgrade/): new images after a backup, and what each platform migrates on its own.
2. [Compatibility matrix](/deploy/upgrade/compatibility/): the versions documented together and the constraints between them.
3. [Breaking changes](/deploy/upgrade/breaking-changes/): the changes that need an action, per version.
4. [Versioning and LTS](/deploy/upgrade/lifecycle/): how releases are numbered and which lines are supported.
:::
::::

## What the stack runs

::stack-builder

## Integrations, run by XTM Composer

::::split
:::column
Connectors, collectors, injectors and executors are programs that run beside the platforms. XTM Composer, part of the stack, deploys them as containers: you pick an integration in the OpenCTI or OpenAEV interface, fill in its settings, and follow its status and logs there.
:::
:::column
::screenshot{id=opencti-ingest-connectors-catalog alt="The connector catalog of OpenCTI, where an administrator picks an integration that XTM Composer deploys."}
:::
::::

## Browse Deploy

::section-overview{id=deploy}
