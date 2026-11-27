---
title: Run actions through CrowdStrike Falcon
description: Let OpenAEV run attack actions on the hosts of your CrowdStrike Falcon host groups, through Real Time Response scripts, without installing the OpenAEV agent.
doc_type: how-to
products:
  - openaev
personas: [platform-engineer, validation-engineer]
owner: docs-openaev
last_reviewed: 2026-11-07
edition: ee
migrated_from:
  - openaev:/integrations/executors/crowdstrike/
---

The CrowdStrike Falcon executor runs OpenAEV implants through the Falcon sensor already installed on your hosts. OpenAEV imports the hosts of the host groups you choose, then, for each inject, asks Falcon's Real Time Response to run a script of yours that starts the implant. This page prepares Falcon, deploys the executor in OpenAEV and checks that the hosts show up. How EDR executors work, and when to prefer the OpenAEV agent, is in [Choose between the OpenAEV agent and your EDR](/openaev/use/executors/).

::diagram{id=openaev-crowdstrike-flow}

## Before you start

- OpenAEV runs with an Enterprise Edition license, and your role has the **Manage tenant settings** capability.
- In Falcon, you can create response scripts, host groups, response policies and API clients.
- The hosts reach the OpenAEV URL over HTTPS: see [How an EDR executor works](/openaev/use/executors/#how-an-edr-executor-works).

## Prepare CrowdStrike Falcon

### Create the two response scripts

OpenAEV sends each command in Base64 to one of two custom scripts, which decode it and run it. In Falcon, go to **Host setup and management > Response and containment > Response scripts and files** and create both scripts. You can name them as you like: their names go in the executor's form.

| Attribute | Unix script (Linux and macOS) | Windows script |
| --- | --- | --- |
| Name | `OpenAEV Subprocessor (Unix)` | `OpenAEV Subprocessor (Windows)` |
| Shell type | bash | PowerShell |
| Script access | Users with the RTR Administrator or RTR Active Responder role | Users with the RTR Administrator or RTR Active Responder role |
| Shared with workflows | Yes | Yes |

The Unix script:

```bash
command=`echo $1 | grep -o '"command":"[^"]*' | grep -o '[^"]*$'`
echo $command | base64 -d | sh
```

The Windows script:

```powershell
$command = $args[0] | ConvertFrom-Json | Select -ExpandProperty 'command';
cmd.exe /d /c powershell.exe -ExecutionPolicy Bypass -WindowStyle Hidden -NonInteractive -NoProfile -encodedCommand $command
```

Both take the same input schema, with one required string, `command`:

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "command": {"type": "string"}
  },
  "required": ["command"]
}
```

### Group the hosts and allow custom scripts

1. In **Host setup and management > Host groups**, create a host group with the hosts your simulations target, and copy its ID.
2. In **Host setup and management > Response policies**, choose each platform of your hosts in the top-left selector, then create a policy or open an existing one.
3. Turn on **Custom Scripts**, and **Falcon Scripts** when the option exists, then click **Save**.
4. In the **Assigned host groups** tab, add your host group. The policy applies within a few minutes: the **Pending** column then shows 0.

### Create an API client

In Falcon, create an API client with the **API integrations**, **Hosts**, **Host groups** and **Real time response** scopes, and keep its client ID and secret. OpenAEV uses it to read the host groups and their hosts, open a Real Time Response session on a host and run your script there.

## Deploy the executor in OpenAEV

:::steps
1. Open :menu[Integrations], then the **Available** tab.
2. Open **CrowdStrike Executor** and click **Deploy**. The **Create a new connector instance** panel opens.
3. Fill in the form. Each field is named after its configuration key:

   | Field | What to enter |
   | --- | --- |
   | **Display name** | The executor's name in OpenAEV, `CrowdStrike` by default |
   | **Executor Crowdstrike Client Id** and **Executor Crowdstrike Client Secret** | The client ID and secret of your API client |
   | **Executor Crowdstrike Host Group** | The ID of your host group, or several IDs separated by commas |
   | **Executor Crowdstrike Windows Script Name** and **Executor Crowdstrike Unix Script Name** | The names of your two scripts, `OpenAEV Subprocessor (Windows)` and `OpenAEV Subprocessor (Unix)` by default |

4. Open **Advanced options**: **Executor Crowdstrike Api Url** points to the US-2 cloud, `https://api.us-2.crowdstrike.com`, by default. Enter the API URL of your Falcon cloud. The other options set how often OpenAEV imports the hosts, cleans them and sends commands: their defaults are in the [`executor.crowdstrike` block](/reference/openaev/configuration/#executor-crowdstrike).
5. Click **Create**.
6. In the **Deployed** tab, open the executor and click **Start**.
:::

::screenshot{id=openaev-executor-crowdstrike-form alt="The Create a new connector instance panel of the CrowdStrike executor: its display name and instance name, the settings it needs, each with its help, and the advanced options."}

## Check that it works

Within a minute of the start, open :menu[Assets]: the hosts of your host groups show in the **Endpoints** list, with **CrowdStrike** in their **Executors** column. :menu[Asset groups] has a group named after each host group. The executor imports the hosts again every 20 minutes, so a host you add to the group in Falcon shows in OpenAEV at the next import.

Then run an atomic test on one host, for example a harmless command, and look for the session of your script in Falcon's Real Time Response audit. When nothing runs, follow [When an inject fails on an EDR executor](/openaev/use/executors/#when-an-inject-fails-on-an-edr-executor).

:::example[Replay Velvet Harbor through Meridian EDR]
Meridian EDR, the endpoint detection of Grand Meridian Hotels, is a CrowdStrike Falcon tenant in the EU-1 cloud. The validation engineer wants to replay Velvet Harbor's PowerShell loader on the Paris front-desk workstations without installing anything new on them.

In Falcon, the platform engineer creates the two scripts with their default names, a host group `Front desk - Paris` with ID `5f2c8e7a91d44b6c8a3e0d17b9c2f461`, assigns it to a Windows response policy that allows custom scripts, and creates an API client. In OpenAEV, the form gets:

| Field | Value |
| --- | --- |
| **Display name** | `Meridian EDR` |
| **Executor Crowdstrike Client Id** | The client ID, kept in the vault as `MERIDIAN_FALCON_CLIENT_ID` |
| **Executor Crowdstrike Client Secret** | The secret, kept as `MERIDIAN_FALCON_CLIENT_SECRET` |
| **Executor Crowdstrike Host Group** | `5f2c8e7a91d44b6c8a3e0d17b9c2f461` |
| **Executor Crowdstrike Api Url** | `https://api.eu-1.crowdstrike.com` |

On a new platform, the same values can come from its configuration instead. The platform reads them at its first start only:

```bash
EXECUTOR_CROWDSTRIKE_ENABLE=true
EXECUTOR_CROWDSTRIKE_API-URL=https://api.eu-1.crowdstrike.com
EXECUTOR_CROWDSTRIKE_CLIENT-ID=${MERIDIAN_FALCON_CLIENT_ID}
EXECUTOR_CROWDSTRIKE_CLIENT-SECRET=${MERIDIAN_FALCON_CLIENT_SECRET}
EXECUTOR_CROWDSTRIKE_HOST-GROUP=5f2c8e7a91d44b6c8a3e0d17b9c2f461
```

After **Start**, the front-desk workstations of the Paris hotel show in :menu[Assets] with the **Meridian EDR** executor, and the asset group `Front desk - Paris` holds them. The Velvet Harbor simulation targets this group, and Falcon runs each action as SYSTEM.
:::

## What's next

- [Choose between the OpenAEV agent and your EDR](/openaev/use/executors/): implant cleanup and troubleshooting.
- [How collectors measure what your security tools saw](/openaev/use/collectors/): add the CrowdStrike collector, so Falcon's detections fill the expectations.
- [Inject status](/openaev/use/injects/status/): understand the results and timeouts of injects.
