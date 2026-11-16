---
title: Run actions through SentinelOne
description: Let OpenAEV run attack actions on the endpoints of your SentinelOne accounts, sites or groups, through Remote Ops scripts, without installing the OpenAEV agent.
doc_type: how-to
products:
  - openaev
personas: [platform-engineer, validation-engineer]
owner: docs-openaev
last_reviewed: 2026-11-07
edition: ee
migrated_from:
  - openaev:/integrations/executors/sentinelone/
---

The SentinelOne executor runs OpenAEV implants through the SentinelOne agent already on your endpoints. OpenAEV imports the active agents of the accounts, sites or groups you choose, then, for each inject, asks SentinelOne to run a Remote Ops script of yours that starts the implant. This page prepares SentinelOne, deploys the executor in OpenAEV and checks that the endpoints show up. How EDR executors work, and when to prefer the OpenAEV agent, is in [Choose between the OpenAEV agent and your EDR](/openaev/use/executors/).

::diagram{id=openaev-sentinelone-flow}

## Before you start

- OpenAEV runs with an Enterprise Edition license, and your role has the **Manage tenant settings** capability.
- Your SentinelOne license includes the remote script orchestration add-on, which runs scripts on agents. It shows in **Settings > Configuration > Add-ons**.
- The endpoints reach the OpenAEV URL over HTTPS: see [How an EDR executor works](/openaev/use/executors/#how-an-edr-executor-works).

## Prepare SentinelOne

Create the API key and the scripts with the same user, in the same account or site.

1. In **Settings > Users > Service Users**, create a service user with at least the **IR Team** role, and keep its API token.
2. In **Automation > Remote Ops**, click **Create new** and upload the Unix script, for Linux and macOS:

   ```bash
   #!/bin/sh

   echo $1 | base64 -d | sh
   ```

3. Upload the Windows script the same way:

   ```powershell
   $command = $args[0];
   cmd.exe /d /c powershell.exe -ExecutionPolicy Bypass -WindowStyle Hidden -NonInteractive -NoProfile -encodedCommand $command
   ```

4. Set both scripts as below, then keep the ID of each one. OpenAEV passes the Base64 command as the script's input.

   | Setting | Unix script | Windows script |
   | --- | --- | --- |
   | Script name | `OpenAEV Subprocessor (Unix)` | `OpenAEV Subprocessor (Windows)` |
   | Script type | Action | Action |
   | OS type | macOS and Linux | Windows |
   | Script execution timeout | 60 seconds | 60 seconds |
   | Input | Input is required, with the instructions `$1` | Input is required, with the instructions `$command` |
5. Note the IDs of the accounts, sites or groups whose endpoints your simulations target, in **Settings > Accounts** and **Sites**.

## Deploy the executor in OpenAEV

:::steps
1. Open :menu[Integrations], then the **Available** tab.
2. Open **SentinelOne Executor** and click **Deploy**. The **Create a new connector instance** panel opens.
3. Fill in the form. Each field is named after its configuration key:

   | Field | What to enter |
   | --- | --- |
   | **Display name** | The executor's name in OpenAEV, `SentinelOne` by default |
   | **Executor Sentinelone Url** | The URL of your SentinelOne console, such as `https://euce1-meridian.sentinelone.net`. OpenAEV calls its API version 2.1 |
   | **Executor Sentinelone Api Key** | The API token of the service user |
   | **Executor Sentinelone Windows Script Id** and **Executor Sentinelone Unix Script Id** | The IDs of your two scripts |

4. Open **Advanced options** and fill in at least one of **Executor Sentinelone Account Id**, **Executor Sentinelone Site Id** and **Executor Sentinelone Group Id**, each with one ID or several separated by commas. With none, OpenAEV imports no endpoint. The other options set how often OpenAEV imports the endpoints, cleans them and sends commands: their defaults are in the [`executor.sentinelone` block](/reference/openaev/configuration/#executor-sentinelone).
5. Click **Create**.
6. In the **Deployed** tab, open the executor and click **Start**.
:::

::screenshot{id=openaev-executor-sentinelone-form alt="The Create a new connector instance panel of the SentinelOne executor: its display name and instance name, the settings it needs, each with its help, and the advanced options."}

## Check that it works

Within a minute of the start, open :menu[Assets]: the active agents of your accounts, sites and groups show as endpoints, with **SentinelOne** in their **Executors** column. :menu[Asset groups] has one group per account, per site and per group, named `<account>`, `<account>_<site>` and `<account>_<site>_<group>`. OpenAEV imports active agents only, and imports them again every 20 minutes.

Each night at 03:00, the executor asks the agents to delete the implant folders older than 24 hours. The schedule is a cron expression, **Executor Sentinelone Clean Implant Cron** in **Advanced options**.

When an inject does not run, look for the script's task in **Automation > Remote Ops**, then follow [When an inject fails on an EDR executor](/openaev/use/executors/#when-an-inject-fails-on-an-edr-executor).

:::example[Validate Meridian EDR on the booking servers]
Grand Meridian's booking database servers, where Velvet Harbor exported the guest records, run the SentinelOne agent in the site `Booking` of the account `Grand Meridian`. The platform engineer creates the service user `openaev-executor` with the **IR Team** role and uploads both scripts with it. In OpenAEV, the form gets:

| Field | Value |
| --- | --- |
| **Display name** | `Meridian EDR - Booking` |
| **Executor Sentinelone Url** | `https://euce1-meridian.sentinelone.net` |
| **Executor Sentinelone Api Key** | The API token, kept in the vault as `MERIDIAN_S1_API_TOKEN` |
| **Executor Sentinelone Windows Script Id** | `2214587703945617411` |
| **Executor Sentinelone Unix Script Id** | `2214587703945617412` |
| **Executor Sentinelone Site Id** | `1903327155824735005` |

On a new platform, the same values can come from its configuration, read at its first start only:

```bash
EXECUTOR_SENTINELONE_ENABLE=true
EXECUTOR_SENTINELONE_URL=https://euce1-meridian.sentinelone.net
EXECUTOR_SENTINELONE_API-KEY=${MERIDIAN_S1_API_TOKEN}
EXECUTOR_SENTINELONE_SITE-ID=1903327155824735005
EXECUTOR_SENTINELONE_WINDOWS-SCRIPT-ID=2214587703945617411
EXECUTOR_SENTINELONE_UNIX-SCRIPT-ID=2214587703945617412
```

After **Start**, the booking servers show in the asset group `Grand Meridian_Booking`. The simulation replays the export of the guest records on them, and SentinelOne runs each action as root or SYSTEM.
:::

## What's next

- [Choose between the OpenAEV agent and your EDR](/openaev/use/executors/): implant cleanup and troubleshooting.
- [How collectors measure what your security tools saw](/openaev/use/collectors/): add the SentinelOne collector, so its detections fill the expectations.
- [Inject status](/openaev/use/injects/status/): understand the results and timeouts of injects.
