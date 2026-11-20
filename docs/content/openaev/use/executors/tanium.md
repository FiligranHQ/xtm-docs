---
title: Run actions through Tanium
description: Let OpenAEV run attack actions on the endpoints of your Tanium computer groups, through two Tanium packages, without installing the OpenAEV agent.
doc_type: how-to
products:
  - openaev
personas: [platform-engineer, validation-engineer]
owner: docs-openaev
last_reviewed: 2026-11-07
edition: ee
migrated_from:
  - openaev:/integrations/executors/tanium/
---

The Tanium executor runs OpenAEV implants through the Tanium client already on your endpoints. OpenAEV imports the endpoints of the computer groups you choose, then, for each inject, asks Tanium to deploy one of two OpenAEV packages, which starts the implant. This page imports the packages, deploys the executor in OpenAEV and checks that the endpoints show up. How EDR executors work, and when to prefer the OpenAEV agent, is in [Choose between the OpenAEV agent and your EDR](/openaev/use/executors/).

::diagram{id=openaev-tanium-flow}

## Before you start

- OpenAEV runs with an Enterprise Edition license, and your role has the **Manage tenant settings** capability.
- In Tanium, you can import packages and create an API token that reads endpoints through the Tanium Gateway (GraphQL) and deploys packages.
- The endpoints reach the OpenAEV URL over HTTPS: see [How an EDR executor works](/openaev/use/executors/#how-an-edr-executor-works).

## Prepare Tanium

1. Choose the packages. Most endpoints take the standard packages. Endpoints that run Tanium Threat Response (TTR) need the TTR packages, which work everywhere but do more on the endpoint and raise more alerts.

   | Packages | When | Files |
   | --- | --- | --- |
   | Standard | The Tanium client only | [openaev-tanium-packages.json](https://github.com/OpenAEV-Platform/openaev/blob/main/openaev-api/src/main/java/io/openaev/executors/tanium/openaev-tanium-packages.json) |
   | TTR | Tanium client with Threat Response | [openaev-tanium-packages-TTR.json](https://github.com/OpenAEV-Platform/openaev/blob/main/openaev-api/src/main/java/io/openaev/executors/tanium/openaev-tanium-packages-TTR.json), with the scripts [openaev-ttr.ps1](https://github.com/OpenAEV-Platform/openaev/blob/main/openaev-api/src/main/java/io/openaev/executors/tanium/openaev-ttr.ps1) and [openaev-ttr.sh](https://github.com/OpenAEV-Platform/openaev/blob/main/openaev-api/src/main/java/io/openaev/executors/tanium/openaev-ttr.sh) attached in the files of the package |

2. Import the file in Tanium. It creates two packages, `OpenAEV Subprocessor (Unix)` and `OpenAEV Subprocessor (Windows)`, which decode the Base64 command OpenAEV passes as their parameter and run it.
3. Open each package and check that **Launch this package command in a process group** is not checked: the implant runs as a detached process, which a process group would end with the package.
4. Keep the ID of each package: it is in the address of its page, `ui/console/packages/<ID>/preview`.
5. Keep the IDs of the computer group whose endpoints your simulations target, and of the action group allowed to run the packages on them.
6. Create an API token, and keep it.

## Deploy the executor in OpenAEV

:::steps
1. Open :menu[Integrations], then the **Available** tab.
2. Open **Tanium Executor** and click **Deploy**. The **Create a new connector instance** panel opens.
3. Fill in the form. Each field is named after its configuration key:

   | Field | What to enter |
   | --- | --- |
   | **Display name** | The executor's name in OpenAEV, `Tanium` by default |
   | **Executor Tanium Api Url** | The URL of your Tanium server. OpenAEV calls its Gateway at `/plugin/products/gateway/graphql` |
   | **Executor Tanium Api Key** | The API token |
   | **Executor Tanium Windows Package Id** and **Executor Tanium Unix Package Id** | The IDs of your two packages |

4. Open **Advanced options** and set **Executor Tanium Computer Group Id**, one ID or several separated by commas, and **Executor Tanium Action Group Id**. They are `1` and `4` by default: check them against your Tanium. The other options set how often OpenAEV imports the endpoints, cleans them and sends commands: their defaults are in the [`executor.tanium` block](/reference/openaev/configuration/#executor-tanium).
5. Click **Create**.
6. In the **Deployed** tab, open the executor and click **Start**.
:::

::screenshot{id=openaev-executor-tanium-form alt="The Create a new connector instance panel of the Tanium executor: its display name and instance name, the settings it needs, each with its help, and the advanced options."}

## Check that it works

Within a minute of the start, open :menu[Assets]: the endpoints of your computer groups show in the **Endpoints** list, with **Tanium** in their **Executors** column, and :menu[Asset groups] has a group named after each computer group. OpenAEV imports them again every 20 minutes.

Each inject is a deployment of one of your packages, in Tanium's actions. When nothing runs there, follow [When an inject fails on an EDR executor](/openaev/use/executors/#when-an-inject-fails-on-an-edr-executor).

:::example[Run the Velvet Harbor simulation on the Paris workstations through Tanium]
Grand Meridian manages the workstations of its Paris hotel with Tanium, without Threat Response. The platform engineer imports the standard packages, which get the IDs `1874` (Windows) and `1875` (Unix), and creates an API token. The front desks are the computer group `231`, and actions on them go through the action group `12`. In OpenAEV, the form gets:

| Field | Value |
| --- | --- |
| **Display name** | `Tanium - Paris` |
| **Executor Tanium Api Url** | `https://tanium.grand-meridian.example` |
| **Executor Tanium Api Key** | The token, kept in the vault as `MERIDIAN_TANIUM_API_TOKEN` |
| **Executor Tanium Windows Package Id** | `1874` |
| **Executor Tanium Unix Package Id** | `1875` |
| **Executor Tanium Computer Group Id** | `231` |
| **Executor Tanium Action Group Id** | `12` |

On a new platform, the same values can come from its configuration, read at its first start only:

```bash
EXECUTOR_TANIUM_ENABLE=true
EXECUTOR_TANIUM_URL=https://tanium.grand-meridian.example
EXECUTOR_TANIUM_API-KEY=${MERIDIAN_TANIUM_API_TOKEN}
EXECUTOR_TANIUM_WINDOWS-PACKAGE-ID=1874
EXECUTOR_TANIUM_UNIX-PACKAGE-ID=1875
EXECUTOR_TANIUM_COMPUTER-GROUP-ID=231
EXECUTOR_TANIUM_ACTION-GROUP-ID=12
```

After **Start**, the front-desk workstations show in :menu[Assets] with the **Tanium - Paris** executor, and the Velvet Harbor simulation runs on them as SYSTEM.
:::

## What's next

- [Choose between the OpenAEV agent and your EDR](/openaev/use/executors/): implant cleanup and troubleshooting.
- [How collectors measure what your security tools saw](/openaev/use/collectors/): add the Tanium Threat Response collector, so its detections fill the expectations.
- [Inject status](/openaev/use/injects/status/): understand the results and timeouts of injects.
