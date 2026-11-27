---
title: Run actions through Palo Alto Cortex
description: Let OpenAEV run attack actions on the endpoints of your Palo Alto Cortex endpoint groups, through the Agent Script Library, without installing the OpenAEV agent.
doc_type: how-to
products:
  - openaev
personas: [platform-engineer, validation-engineer]
owner: docs-openaev
last_reviewed: 2026-11-07
edition: ee
migrated_from:
  - openaev:/integrations/executors/palo-alto-cortex/
---

The Palo Alto Cortex executor runs OpenAEV implants through the Cortex XDR agent already on your endpoints. OpenAEV imports the endpoints of the endpoint groups you choose, then, for each inject, asks Cortex to run a script of your Agent Script Library that starts the implant. This page prepares Cortex, deploys the executor in OpenAEV and checks that the endpoints show up. How EDR executors work, and when to prefer the OpenAEV agent, is in [Choose between the OpenAEV agent and your EDR](/openaev/use/executors/).

::diagram{id=openaev-palo-alto-cortex-flow}

## Before you start

- OpenAEV runs with an Enterprise Edition license, and your role has the **Manage tenant settings** capability.
- In Cortex, you can add scripts to the Agent Script Library, manage endpoint groups and create API keys.
- The endpoints reach the OpenAEV URL over HTTPS: see [How an EDR executor works](/openaev/use/executors/#how-an-edr-executor-works).

## Prepare Palo Alto Cortex

### Add the two scripts

OpenAEV sends each command in Base64 to one of two Python scripts, which decode it and run it. In **Investigation & Response > Action Center > Agent Script Library**, click **+ New Script** and add both:

| Setting | Unix script | Windows script |
| --- | --- | --- |
| **Script Name** | `OpenAEV Subprocessor (Unix)` | `OpenAEV Subprocessor (Windows)` |
| **Mark as high risk script** | Checked | Checked |
| **Supported OS** | macOS and Linux | Windows |
| **Timeout** | 60 seconds | 600 seconds |
| Input | **Run by entry point** `run`, with a `command` parameter of type **String** | **Run by entry point** `run`, with a `commands_list` parameter of type **List of strings** |
| **Output Type** | **Auto Detect** | **Auto Detect** |

The Unix script:

```python
import os

def run(command):
    exit_code = os.system("echo " + command + " | base64 -d | sh")
    print("Exit code:", exit_code)

if __name__ == "__main__":
    run(command)
```

The Windows script is longer: it attaches a console the agent's process lacks, then runs the commands. Download [paloaltocortex_subprocessor_windows.py](https://github.com/OpenAEV-Platform/openaev/blob/main/docs/docs/integrations/executors/assets/paloaltocortex_subprocessor_windows.py) from the OpenAEV repository.

You can rename the scripts. Add the **Script UID** column to the list of scripts and keep the UID of each one: they go in the executor's form.

### Group the endpoints and create an API key

1. In **Inventory > Endpoints > Groups**, create a group with the endpoints your simulations target, and keep its name.
2. In **Settings > Configurations > API Keys**, create a key with the **Standard** security level and at least the **Instance Administrator** role. Keep the key, its ID, and the API URL Cortex shows next to it.

## Deploy the executor in OpenAEV

:::steps
1. Open :menu[Integrations], then the **Available** tab.
2. Open **Palo Alto Cortex Executor** and click **Deploy**. The **Create a new connector instance** panel opens.
3. Fill in the form. Each field is named after its configuration key:

   | Field | What to enter |
   | --- | --- |
   | **Display name** | The executor's name in OpenAEV, `PaloAltoCortex` by default |
   | **Executor Paloaltocortex Url** | The API URL of your tenant. OpenAEV calls its public API version 1 |
   | **Executor Paloaltocortex Api Key Id** and **Executor Paloaltocortex Api Key** | The ID of the API key and the key |
   | **Executor Paloaltocortex Group Name** | The name of your endpoint group, or several names separated by commas |
   | **Executor Paloaltocortex Windows Script Uid** and **Executor Paloaltocortex Unix Script Uid** | The UIDs of your two scripts |

4. Under **Advanced options**, the other options set how often OpenAEV imports the endpoints, cleans them and sends commands. Their defaults are in the [`executor.paloaltocortex` block](/reference/openaev/configuration/#executor-paloaltocortex).
5. Click **Create**.
6. In the **Deployed** tab, open the executor and click **Start**.
:::

::screenshot{id=openaev-executor-palo-alto-cortex-form alt="The Create a new connector instance panel of the Palo Alto Cortex executor: its display name and instance name, the settings it needs, each with its help, and the advanced options."}

## Check that it works

Within a minute of the start, open :menu[Assets]: the endpoints of your groups that Cortex saw in the last 24 hours show in the **Endpoints** list, with **PaloAltoCortex** in their **Executors** column, and :menu[Asset groups] has a group named after each endpoint group. OpenAEV imports them again every 20 minutes.

On Windows, Cortex ends the processes its script started when the script ends. So the implant starts from a scheduled task of the SYSTEM account, `OpenAEV-Inject-<inject>-Agent-<agent>`, which deletes itself when the implant ends.

Each inject is a script run in **Action Center**. When nothing runs there, follow [When an inject fails on an EDR executor](/openaev/use/executors/#when-an-inject-fails-on-an-edr-executor).

:::example[Replay Velvet Harbor on the hotels of the south]
The front-desk workstations of Grand Meridian's hotels in the south of France run the Cortex XDR agent, in the endpoint group `FD-South`. The platform engineer adds both scripts, creates a **Standard** API key with the **Instance Administrator** role, and fills in the form:

| Field | Value |
| --- | --- |
| **Display name** | `Meridian EDR - South` |
| **Executor Paloaltocortex Url** | `https://api-grandmeridian.xdr.eu.paloaltonetworks.com` |
| **Executor Paloaltocortex Api Key Id** | `14` |
| **Executor Paloaltocortex Api Key** | The key, kept in the vault as `MERIDIAN_CORTEX_API_KEY` |
| **Executor Paloaltocortex Group Name** | `FD-South` |
| **Executor Paloaltocortex Windows Script Uid** | `8a1f3c0e5b7d4e29a6c2f0b1d3e4a5c6` |
| **Executor Paloaltocortex Unix Script Uid** | `2b9d7e4f1a6c4b38b5e0c9d2f7a1e3b4` |

After **Start**, the workstations of `FD-South` show in :menu[Assets], and the Velvet Harbor simulation runs its PowerShell loader on them through Cortex, as SYSTEM.
:::

## What's next

- [Choose between the OpenAEV agent and your EDR](/openaev/use/executors/): implant cleanup and troubleshooting.
- [How collectors measure what your security tools saw](/openaev/use/collectors/): add the Palo Alto Cortex XDR collector, so its alerts fill the expectations.
- [Inject status](/openaev/use/injects/status/): understand the results and timeouts of injects.
