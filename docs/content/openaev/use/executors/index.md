---
title: Choose between the OpenAEV agent and your EDR
description: Compare the executors that run attack actions on your endpoints, the OpenAEV agent or the EDR you already run, and learn how they enroll endpoints and run implants.
doc_type: explanation
products:
  - openaev
personas: [validation-engineer, platform-engineer]
owner: docs-openaev
last_reviewed: 2026-11-07
migrated_from:
  - openaev:/integrations/executors/executors/
---

To run attack actions on an endpoint, OpenAEV needs an executor there: a program that starts an implant for each inject, and the implant runs the actions. You can install the OpenAEV agent, or reuse the endpoint detection and response (EDR) agent already on your endpoints, driven through its vendor's API, with nothing new to deploy. This page compares the executors and explains how an EDR executor enrolls endpoints and runs implants, so you can choose one and know what to expect from it. The parts of OpenAEV and the path of an inject are in [How OpenAEV works](/openaev/concepts/architecture/).

::diagram{id=openaev-executors-overview}

## The executors

| Executor | Runs implants through | Enterprise Edition | Set up |
| --- | --- | --- | --- |
| OpenAEV agent | The OpenAEV agent you install | No | [Install the OpenAEV agent on your endpoints](/openaev/use/agents/) |
| CrowdStrike Falcon | Real Time Response custom scripts | Yes | [Run actions through CrowdStrike Falcon](/openaev/use/executors/crowdstrike/) |
| Microsoft Defender for Endpoint | Live Response library scripts | Yes | [Run actions through Microsoft Defender for Endpoint](/openaev/use/executors/mde/) |
| SentinelOne | Remote Ops scripts | Yes | [Run actions through SentinelOne](/openaev/use/executors/sentinelone/) |
| Palo Alto Cortex | Agent Script Library scripts | Yes | [Run actions through Palo Alto Cortex](/openaev/use/executors/palo-alto-cortex/) |
| Tanium | Tanium packages | Yes | [Run actions through Tanium](/openaev/use/executors/tanium/) |
| Caldera | MITRE Caldera agents | No | [Run actions through Caldera](/openaev/use/executors/caldera/) |

Every executor works on Windows, Linux and macOS. Without an Enterprise Edition license, the agents of an EDR executor show an **EE** chip on their endpoint's page, and the injects that target them fail with a `LICENSE RESTRICTION` trace.

How the actions run differs between the two families:

| | OpenAEV agent | EDR executors |
| --- | --- | --- |
| Rights of the actions | Those of the installation mode you choose: a standard user, an administrator, or SYSTEM and root | Always SYSTEM on Windows, root on Linux and macOS |
| Agents per endpoint | Several, one per mode and user, to compare levels of rights | One per executor. The executor finds the endpoint by the EDR's device ID, then by its MAC address, so an endpoint that already has an OpenAEV agent gets the EDR's agent next to it |
| Endpoints in OpenAEV | Each agent enrolls its endpoint when it starts | The executor imports the endpoints of the groups you choose in the EDR |
| What to deploy | The agent, with its antivirus exclusion | Two scripts or packages in the EDR console, and an API key |

Choose the OpenAEV agent to simulate an attacker with limited rights, or on endpoints without an EDR. Choose your EDR executor to cover many endpoints at once without installing anything on them, when the attacker you simulate has full control of the machine.

## How an EDR executor works

An EDR executor runs inside the platform. Once started, it works in three loops:

1. **It imports the endpoints.** Every 20 minutes by default, it reads the endpoints of the groups you named from the EDR's API. Each one becomes an endpoint in :menu[Assets], with an agent of the executor, and each group becomes an asset group in :menu[Asset groups], named after it. The last time the EDR saw the endpoint gives the agent its **Last Seen**: an agent unseen for one hour is inactive.
2. **It runs the injects.** When an inject targets these endpoints, the platform builds the command that downloads and starts the implant, encodes it in Base64, and asks the EDR's API to run your OpenAEV script with it, in batches every 5 seconds to respect the API's limits. The script decodes the command and runs it. The implant downloads into `runtimes/implant-<id>` under `C:\Program Files (x86)\Filigran\OAEV Agent` on Windows or `/opt/openaev-agent` on Linux and macOS, runs the actions, and sends its results to OpenAEV.
3. **It cleans the endpoints.** It sends a command that deletes the folders under `runtimes` and `payloads` older than 24 hours: every 8 hours by default for CrowdStrike Falcon, Palo Alto Cortex and Tanium, every day at 03:00 for SentinelOne, and before each inject for Microsoft Defender for Endpoint.

So each endpoint must reach the OpenAEV URL over HTTPS, `openaev.agent-url` or `openaev.base-url` when the first is not set, from the [`openaev` block](/reference/openaev/configuration/#openaev). The EDR never needs to reach OpenAEV: the platform calls its API.

On Windows, Microsoft Defender for Endpoint and Palo Alto Cortex end every process their remote session started. For these two, the command starts the implant from a scheduled task of the SYSTEM account, `OpenAEV-Inject-<inject>-Agent-<agent>`, which deletes itself when the implant ends.

## Where an executor is configured

You deploy an EDR executor from the integrations catalog of OpenAEV, like any other integration. It runs inside the platform, so XTM Composer is not needed. Deploying, configuring, starting and stopping it requires an Enterprise Edition license and the **Manage tenant settings** capability. Each executor's page gives the steps and the fields of its form.

Each executor also has a block of the configuration: [`executor.crowdstrike`](/reference/openaev/configuration/#executor-crowdstrike), [`executor.mde`](/reference/openaev/configuration/#executor-mde), [`executor.sentinelone`](/reference/openaev/configuration/#executor-sentinelone), [`executor.paloaltocortex`](/reference/openaev/configuration/#executor-paloaltocortex), [`executor.tanium`](/reference/openaev/configuration/#executor-tanium) and [`executor.caldera`](/reference/openaev/configuration/#executor-caldera). The platform reads these keys once, the first time it starts: when the block's `enable` key is `true`, it turns them into a deployed executor, and it ignores them afterwards. Change an executor in the interface, not in the configuration. Microsoft Defender for Endpoint is set up in the interface only: its keys never create it.

::screenshot{id=openaev-integrations-executors-available alt="The executors of Integrations, each with its Deploy button: Caldera, CrowdStrike, Microsoft Defender for Endpoint, Palo Alto Cortex, SentinelOne and Tanium, which register their managed hosts as OpenAEV executors."}

## When an inject fails on an EDR executor

1. Run the inject and wait until it fails or times out.
2. In the EDR console, look for the execution of your OpenAEV script. When there is none, the problem is upstream: the API key, the script's name or ID, the endpoint's group, or the executor's form.
3. Copy the command OpenAEV sent and decode it from Base64. Check its paths for the endpoint's system, its parameters and its quoting.
4. Run the decoded command on the endpoint and keep its whole output: it shows why the implant did not start or did not report.

An agent can be active and its inject still end in `TIMEOUT`: active means the EDR saw the endpoint in the last hour, while `TIMEOUT` means the implant did not report to OpenAEV within 10 minutes, the default of `inject.execution.threshold.minutes` in the [`inject` block](/reference/openaev/configuration/#inject). When the command ran, check that the endpoint reaches the OpenAEV URL (network, proxy, firewall, certificate) and that no antivirus deletes the implant. Inject results are explained in [Inject status](/openaev/use/injects/status/).

## What's next

- [Install the OpenAEV agent on your endpoints](/openaev/use/agents/)
- [Run actions through CrowdStrike Falcon](/openaev/use/executors/crowdstrike/), [Microsoft Defender for Endpoint](/openaev/use/executors/mde/), [SentinelOne](/openaev/use/executors/sentinelone/), [Palo Alto Cortex](/openaev/use/executors/palo-alto-cortex/) or [Tanium](/openaev/use/executors/tanium/)
- [How collectors measure what your security tools saw](/openaev/use/collectors/): read back what the EDR detected.
