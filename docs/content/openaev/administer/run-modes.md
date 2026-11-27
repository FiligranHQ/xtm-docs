---
title: Start the platform in safe mode
description: Start OpenAEV with its scheduled jobs stopped, to inspect or repair the platform while no inject, purge or synchronization runs, then return to normal mode.
doc_type: how-to
products:
  - openaev
personas: [platform-engineer, admin]
owner: docs-openaev
last_reviewed: 2026-11-07
no_screenshot: "The run mode is a configuration key of the platform, set outside the interface: the diagram shows what keeps running and what stops."
migrated_from:
  - openaev:/deployment/platform/run-modes/
---

Safe mode starts OpenAEV without its scheduled jobs: the interface and the API answer, but no inject is sent, no scheduled scenario starts, and no purge or synchronization runs. Use it to look at a platform without letting it act. For example, stop a simulation that would fire against production endpoints, check the data after an upgrade, or find which job puts a load on the database. The run mode is set by `openaev.run-mode` (`OPENAEV_RUN-MODE`): `normal`, the default, or `safe`.

## Before you start

- You can change the environment of the `openaev` service and restart it. With xtm-docker, see [Change the stack without editing docker-compose.yml](/deploy/install/docker-compose/#change-the-stack-without-editing-docker-compose-yml).
- Tell the users of every tenant: in safe mode, no simulation of any tenant runs.

## What stops, what keeps running

::diagram{id=openaev-run-modes-safe}

| Stopped in safe mode | What you notice |
| --- | --- |
| Execution of injects, scheduled scenarios, atomic testings and communication checks, and the closing of finished injects | Simulations stay where they are: their next injects are not sent |
| Synchronization with XTM Composer, processing of the security coverage requests of OpenCTI, and the registration of OpenAEV in OpenCTI | Integrations deployed or changed in the interface are not started; coverage requests wait |
| Attack chaining queues, workflow timeouts and the timeouts of autonomous runs | Attack chains stop at their current step |
| Agent inactivity monitor | Agents that stop reporting keep their last status |
| Notification digests and scheduled reportings | No digest and no scheduled report is sent |
| Copy of the data to Elasticsearch or OpenSearch | Search results and lists that read the search engine miss the latest changes |
| Retention and purges: user events, execution traces, notification events, URL access tokens, and the tenants deleted more than 30 days ago | Old data stays; a deleted tenant is not purged |

The rest keeps running: the interface and the API, the health probes, the usage telemetry, the registration with XTM One and the XTM Hub checks. The expectations keep expiring, and the injectors and executors that work on their own keep working, such as the polling of an EDR executor.

:::warning
Expectations keep expiring in safe mode. A simulation that was running when you switched mode sees its detection and prevention expectations fail when their time is up, because its injects never ran. Pause the simulations that matter before you switch, and launch them again once back in normal mode.
:::

## Switch to safe mode

:::steps
1. Set the run mode in the environment of the `openaev` service:

   ```yaml
   services:
     openaev:
       environment:
         OPENAEV_RUN-MODE: safe
   ```

2. Recreate the container so that it reads the new value.

   ```bash
   docker compose up -d openaev
   ```

   :::verify
   Every page of OpenAEV, the login page included, shows the banner **Safe mode is active: background processing is disabled.**
   :::
:::

## Return to normal mode

Remove `OPENAEV_RUN-MODE` from the environment, or set it to `normal`, and recreate the container. The banner disappears and the jobs start again. Every inject whose time came while the platform was in safe mode is sent within the next minute, all at once: pause the simulations that must not catch up before you switch back. The copy to the search engine catches up with the changes made in between.

:::example[Hold a simulation that targets the wrong hotels]
On a Friday evening, the security engineer of Grand Meridian Hotels sees a mistake. The simulation replaying Velvet Harbor targets the booking database servers of every hotel, not only the Paris test site. He asks Example CERT to stop the platform from acting while he fixes the targets. The platform engineer of Example CERT adds `OPENAEV_RUN-MODE: safe` to the `openaev` service and runs `docker compose up -d openaev`. The banner shows on every page, no inject leaves the platform, and the engineer moves the simulation to the right asset group. The platform engineer then removes the variable and recreates the container: the platform is back in normal mode, and the injects of the other tenants that fell due over the evening are sent at once.
:::

## What's next

- [Troubleshoot OpenAEV](/openaev/administer/troubleshoot/), to find the cause of the problem.
- [Turn on the debug mode to find slow requests](/openaev/administer/debug-mode/), when a job slows the database down.
- The [openaev block](/reference/openaev/configuration/#openaev) of the configuration reference.
