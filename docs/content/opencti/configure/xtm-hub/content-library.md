---
title: XTM Hub content library
description: What the XTM Hub library offers to OpenCTI, how one-click deployment reaches your platform, and what to check before you use the content.
sidebar_position: 1
doc_type: explanation
products:
  - opencti
personas: [admin, cti-lead]
owner: docs-opencti
last_reviewed: 2026-11-01
---

The XTM Hub library holds ready-made OpenCTI content, published by Filigran and the community: dashboards, playbooks, feeds and connectors. Once OpenCTI is [connected to the XTM Hub](/opencti/configure/xtm-hub/), you deploy an item in one click and it lands in the matching screen of the platform. A CTI team lead starts from a dashboard or a playbook someone already built; an administrator adds a feed or a connector without writing its configuration. This page explains what the library offers, how deployment works, and what to check before the content goes live.

## What it is good for

::::only{opencti=">=7.260811"}
| Goal | Content | Lands in |
| --- | --- | --- |
| Start with a useful home view | Custom dashboards | The imported dashboard, in :menu[Dashboards] |
| Automate a common task | Playbooks | The imported playbook, stopped, in :menu[Data > Processing > Automation] |
| Change how an entity type looks | Custom views | :menu[Settings > Customization] |
| Ingest a source without code | CSV, RSS and Trusted Automated Exchange of Intelligence Information (TAXII) feeds, OpenCTI-to-OpenCTI synchronizers | A prefilled creation form; once closed, the deployed integrations of :menu[Integrations] |
| Add a connector | Connectors of the catalog | The catalog page of the connector, with the deployment dialog open |
::::

::::only{opencti="<7.260811"}
| Goal | Content | Lands in |
| --- | --- | --- |
| Start with a useful home view | Custom dashboards | The imported dashboard, in :menu[Dashboards] |
| Ingest a source without code | CSV and Trusted Automated Exchange of Intelligence Information (TAXII) feeds, OpenCTI-to-OpenCTI synchronizers | A prefilled creation form in :menu[Data > Ingestion] |
| Add a connector | Connectors of the catalog | The catalog page of the connector, with the deployment dialog open |
::::

## How deployment works

::diagram{id=admin-xtm-hub-content-library}

There are two ways in:

- **From the XTM Hub.** Click deploy on an item. The XTM Hub opens OpenCTI on a deployment link. OpenCTI downloads the file from the XTM Hub, imports it and takes you to the result.
- **From OpenCTI.** Screens that accept library content have an **Import from Hub** button, which opens the matching part of the library. You find it on the dashboards list and on the feed and synchronizer screens, for example. The button only shows when your browser reaches the XTM Hub.

The import runs as you, so you need the right to create that kind of content: the capability for each kind is listed in [Who can deploy content](/opencti/configure/xtm-hub/#who-can-deploy-content). Without it, the import stops with a message saying you do not have the rights to create that content.

When the platform is not connected, or has lost its connection, a **Connectivity lost** dialog says the XTM Hub connection is unavailable. With **Manage XTM Hub**, it offers **Reconnect**; otherwise it asks you to contact your administrator. When the download or the import fails, a message says so, such as **An error occurred while importing dashboard** or **An error occurred while importing playbook**, and you land back on the home page or on the list of that content.

## Check content before you use it

Content from the library is a starting point, not a finished setup.

- **Playbooks arrive stopped.** Check that the labels, markings, connectors, notifiers and templates they refer to exist on your platform, then start them. See [Create a playbook](/opencti/use/automate/playbooks/create/).
- **Feeds need your settings.** Review the schedule, the author and the markings of a feed in its creation form before you save it.
- **Dashboards follow access rights.** A dashboard shows only the data its viewer may see, so a widget can look empty to a user with restricted access. Share it with [authorized members](/opencti/administer/sharing/).
- **Connectors deploy through XTM Composer.** Without it, the catalog says so. See [XTM Composer](/deploy/integrations/composer/).

:::only{opencti=">=7.260811"}
- **Community connectors are not supported by Filigran.** The catalog page of a connector reads **Supported by Filigran** or **Supported by Community**. Deploying a community connector opens **Deploy a Community Connector**, where you acknowledge that you deploy it at your own risk. Read its source code before you give it credentials.
:::

::::only{opencti=">=7.260811"}
## Stay informed of new content

Connected platforms receive the [Follow what XTM Hub publishes](/opencti/configure/xtm-hub/news-feed/): a notification each time the library publishes new content, such as a dashboard or a playbook.
::::

## What's next

- [Connect to XTM Hub](/opencti/configure/xtm-hub/) if your platform is not connected yet.
- [Reuse content from XTM Hub](/workflows/reuse-from-hub/), the end-to-end workflow.
- [Follow what XTM Hub publishes](/opencti/configure/xtm-hub/news-feed/) to follow new content.
