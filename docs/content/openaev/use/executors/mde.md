---
title: Run actions through Microsoft Defender for Endpoint
description: Let OpenAEV run attack actions on the devices Microsoft Defender for Endpoint protects, through Live Response, without installing the OpenAEV agent.
doc_type: how-to
products:
  - openaev
personas: [platform-engineer, validation-engineer]
owner: docs-openaev
last_reviewed: 2026-11-07
edition: ee
migrated_from:
  - openaev:/integrations/executors/mde/
---

The Microsoft Defender for Endpoint (MDE) executor runs OpenAEV implants through the Defender sensor already on your devices. OpenAEV installs nothing: it imports the onboarded devices of the device groups you choose, then, for each inject, opens a Live Response session on the device and runs a script of your Live Response library that starts the implant. This page prepares Microsoft Entra ID and Defender, deploys the executor in OpenAEV and checks that the devices show up. How EDR executors work, and when to prefer the OpenAEV agent, is in [Choose between the OpenAEV agent and your EDR](/openaev/use/executors/).

::diagram{id=openaev-mde-flow}

## Before you start

- OpenAEV runs with an Enterprise Edition license, and your role has the **Manage tenant settings** capability.
- In Microsoft Entra ID, you can register an application and grant admin consent. In the Microsoft Defender portal, you can change the advanced features and upload to the Live Response library.
- The devices reach the OpenAEV URL over HTTPS: see [How an EDR executor works](/openaev/use/executors/#how-an-edr-executor-works).

## Prepare Microsoft Entra ID and Defender

### Register an application

OpenAEV signs in to the Defender API as an application, with the client credentials flow.

1. In **Microsoft Entra ID > App registrations**, click **New registration** and register the application.
2. Keep its **Directory (tenant) ID** and **Application (client) ID**.
3. In **Certificates & secrets**, create a client secret and keep its value.
4. In **API permissions**, add these application permissions of **WindowsDefenderATP**, then click **Grant admin consent**:

| Permission | Why OpenAEV needs it |
| --- | --- |
| `Machine.Read.All` | List the devices and their device group, to import them |
| `Machine.LiveResponse.All` | Run the Live Response script that starts the implant |
| `AdvancedQuery.Read.All` | Read the recent activity of each device, to tell which devices are up |

They must be application permissions, not delegated ones: OpenAEV works without a signed-in user. Without `AdvancedQuery.Read.All`, OpenAEV falls back on the sensor's health status, which tells less reliably whether a device is reachable.

### Turn on Live Response

In the Microsoft Defender portal, go to **Settings > Endpoints > Advanced features** and turn on:

- **Live Response**;
- **Live Response for servers**, when you target Windows Server or Linux servers;
- **Live Response unsigned script execution**: the OpenAEV scripts are not signed, and without this option every inject fails at the Live Response step.

### Upload the two scripts

OpenAEV passes the whole command as one Base64 argument to one of two scripts of the Live Response library, which decode it and run it. Upload both to the library, and turn on the parameters option in the upload dialog for each one, so that both show **Has parameters: Yes**. Without it, MDE runs the script without the command and every inject fails.

`openaev-subprocessor.ps1`, for Windows:

```powershell
if ($args.Count -lt 1 -or [string]::IsNullOrEmpty($args[0])) {
    Write-Error "OpenAEV subprocessor: no argument received (expected Base64 command as first parameter)"
    exit 1
}
$decoded = [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($args[0]))
Invoke-Expression $decoded
```

`openaev-subprocessor.sh`, for Linux and macOS:

```bash
#!/bin/sh

if [ -z "$1" ]; then
    echo "OpenAEV subprocessor: no argument received (expected Base64 command as first parameter)" >&2
    exit 1
fi
echo "$1" | base64 -d | sh
```

Keep them as they are: in PowerShell, the command must come through `$args[0]`, and a parameter named `$Args` breaks it.

### Find the IDs of your device groups

To import only some devices, OpenAEV needs the numeric ID (`rbacGroupId`) of their device groups. The Defender portal shows the group names only, under **Settings > Endpoints > Permissions > Device groups**, so read the IDs from the devices with the same application:

```powershell
$tenant = $env:MDE_TENANT_ID
$client = $env:MDE_CLIENT_ID
$secret = $env:MDE_CLIENT_SECRET

$token = (Invoke-RestMethod -Method Post `
  -Uri "https://login.microsoftonline.com/$tenant/oauth2/v2.0/token" `
  -Body @{
    grant_type    = 'client_credentials'
    client_id     = $client
    client_secret = $secret
    scope         = 'https://api.securitycenter.microsoft.com/.default'
  }).access_token

$machines = Invoke-RestMethod `
  -Uri 'https://api.securitycenter.microsoft.com/api/machines?$select=rbacGroupId,rbacGroupName' `
  -Headers @{ Authorization = "Bearer $token" }

$machines.value | Select-Object rbacGroupName, rbacGroupId -Unique | Sort-Object rbacGroupName | Format-Table -AutoSize
```

Keep the URL in single quotes, so that PowerShell does not read `$select` as a variable. A group shows only once it holds a device that MDE has seen. `UnassignedGroup` holds the devices of no custom group.

## Deploy the executor in OpenAEV

:::steps
1. Open :menu[Integrations], then the **Available** tab.
2. Open **Microsoft Defender for Endpoint Executor** and click **Deploy**. The **Create a new connector instance** panel opens.
3. Fill in the form. Each field is named after its configuration key:

   | Field | What to enter |
   | --- | --- |
   | **Display name** | The executor's name in OpenAEV, `Microsoft Defender for Endpoint` by default |
   | **Executor Mde Azure Tenant Id** | The **Directory (tenant) ID** of the application |
   | **Executor Mde Client Id** and **Executor Mde Client Secret** | Its **Application (client) ID** and its client secret |
   | **Executor Mde Windows Script Name** and **Executor Mde Unix Script Name** | The names of the two scripts in the library, `openaev-subprocessor.ps1` and `openaev-subprocessor.sh` by default |

4. Open **Advanced options**. In **Executor Mde Device Group**, enter the IDs of your device groups, separated by commas: left empty, OpenAEV imports every onboarded device of the tenant. The other options keep their defaults, from the [`executor.mde` block](/reference/openaev/configuration/#executor-mde): the Microsoft URLs, an import every 20 minutes, and 10 devices per batch of 5 seconds, since the Live Response API limits calls more strictly than other EDRs.
5. Click **Create**.
6. In the **Deployed** tab, open the executor and click **Start**.
:::

The Microsoft Defender for Endpoint executor is set up in the interface only: the keys of its configuration block never create it.

::screenshot{id=openaev-executor-mde-form alt="The Create a new connector instance panel of the Microsoft Defender for Endpoint executor: its display name and instance name, the settings it needs, each with its help, and the advanced options."}

## Check that it works

Within a minute of the start, open :menu[Assets]: the devices of your groups show in the **Endpoints** list, with **Microsoft Defender for Endpoint** in their **Executors** column, and :menu[Asset groups] has a group named after each device group. Only devices whose onboarding status is **Onboarded** are imported: devices that Defender discovered but does not protect have no sensor to run Live Response.

An agent is active when its device showed activity in Advanced Hunting recently: within 70 minutes plus one import interval, so 90 minutes with the defaults. A device that goes offline shows as inactive within about two hours. Keep the import interval under one hour, or agents age out between two imports.

Each inject is a Live Response action, which you find in the Defender portal:

- in **Action center > History**, with **Live response** as the action type and the API as its source;
- on the device's **Timeline** and in its Live Response session log;
- in **Advanced hunting**, on `DeviceProcessEvents`, with the scheduled task, the implant and the commands of the actions.

On Windows, a Live Response session ends every process it started. So the implant starts from a scheduled task of the SYSTEM account, `OpenAEV-Inject-<inject>-Agent-<agent>`, which deletes itself when the implant ends.

## Troubleshoot

| What you see | What to check |
| --- | --- |
| Every inject fails at the Live Response step | **Live Response unsigned script execution** is on, both scripts are in the library under the names of the form with **Has parameters: Yes**, and `Machine.LiveResponse.All` has admin consent |
| Devices are missing | The device group IDs, the **Onboarded** status of the devices, and that they report to Defender |
| Devices show as inactive | `AdvancedQuery.Read.All` has admin consent; the platform's log warns when it falls back on the health status |
| An inject times out on an active device | Live Response runs only while the sensor is connected: on an offline device, the action stays **Pending** in **Action center > History**. MDE allows one Live Response session per device, so OpenAEV cancels an action pending for more than 30 minutes before it sends the next one |
| The inject is in error but has output | The status comes from the action itself: a command that exits with an error, or writes to the error output, marks the trace in error even though the executor ran it. Read the exit code and the output in the trace |

:::example[Run the Velvet Harbor simulation on the Spanish hotels]
The front-desk workstations of Grand Meridian's Spanish hotels are protected by Microsoft Defender for Endpoint, in a device group named `Front desk - Spain`. The platform engineer registers the application `OpenAEV executor`, grants the three permissions, turns on the three Live Response features and uploads both scripts. The script above lists the groups:

```text
rbacGroupName        rbacGroupId
-------------        -----------
Front desk - Spain           412
UnassignedGroup              366
```

In OpenAEV, the form gets:

| Field | Value |
| --- | --- |
| **Display name** | `Meridian EDR - Spain` |
| **Executor Mde Azure Tenant Id** | The directory ID, kept in the vault as `MDE_TENANT_ID` |
| **Executor Mde Client Id** | The application ID, kept as `MDE_CLIENT_ID` |
| **Executor Mde Client Secret** | The secret, kept as `MDE_CLIENT_SECRET` |
| **Executor Mde Device Group** | `412` |

After **Start**, the workstations of `Front desk - Spain` show in :menu[Assets] with the **Meridian EDR - Spain** executor. The Velvet Harbor simulation targets the asset group of the same name, and each PowerShell action shows in Advanced Hunting next to the alert Defender raised.
:::

## What's next

- [Choose between the OpenAEV agent and your EDR](/openaev/use/executors/): implant cleanup and troubleshooting.
- [How collectors measure what your security tools saw](/openaev/use/collectors/): add the Microsoft Defender collector, so Defender's detections fill the expectations.
- [Inject status](/openaev/use/injects/status/): understand the results and timeouts of injects.
