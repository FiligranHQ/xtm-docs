---
title: Collect logs for support
description: Generate and download an OpenCTI support package, a ZIP file with the recent application logs and usage statistics of every node, to troubleshoot or to send to support.
doc_type: how-to
products:
  - opencti
personas: [platform-engineer, admin]
owner: docs-platform
last_reviewed: 2026-10-31
migrated_from:
  - opencti:/administration/support-package/
---

A support package is a ZIP file with the recent application logs and usage statistics of every OpenCTI node of your platform. Each node adds its support log, a copy of its application log from the `info` level up (as far as `APP__APP_LOGS__LOGS_LEVEL` lets it through), kept in up to three files of 10 MB, and its usage statistics. Generate one to investigate a problem, or to send to Filigran support or to the community with your question: it gathers in one download what you would otherwise collect node by node.

::diagram{id=deploy-operate-support-package}

## Before you start

- The capability **Access to support data**, which opens :menu[Settings > Filigran Experience].
- Leave any draft first: a support package cannot be generated in draft mode.

:::warning[Check the content before you share it]
OpenCTI avoids writing sensitive data to these logs, but a support package may still contain information you do not want to share with everyone. Open it and check it against your security policy before you attach it to a ticket.
:::

## Generate and download a package

:::steps
1. Open :menu[Settings > Filigran Experience] and, under **Support packages**, click **Generate Support Package**.

   Every OpenCTI node receives the request and uploads its files. The package shows the status `IN_PROGRESS`.

2. Wait for the status `READY`: once every node has answered, OpenCTI gathers their files in one ZIP file. Then click **Download this support package**.

3. Delete the package when you no longer need it, with **Delete this support package**.
:::

## When a node does not answer

If the platform is unstable, some nodes may not send their files, and the package stays `IN_PROGRESS`.

- After one minute, **Force download on this support package** turns on: it builds the ZIP file with the files received so far. This partial package lacks the logs of the nodes that did not answer.
- After five minutes without reaching `READY`, the package shows `TIMEOUT`.

A package that failed shows `IN_ERROR`: generate a new one.

## What's next

- [Troubleshoot the XTM stack](/deploy/operate/troubleshoot/), to read the logs yourself first.
- [Check that the platforms are healthy](/deploy/operate/health-probes/), to find a node or a dependency that is down.
