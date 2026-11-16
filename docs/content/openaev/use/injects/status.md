---
title: Inject execution statuses
description: Every status an inject and its execution traces can take in OpenAEV, what each one means, and how the platform computes the status of an inject from its agents.
doc_type: reference
products:
  - openaev
personas: [validation-engineer]
owner: docs-openaev
last_reviewed: 2026-11-07
migrated_from:
  - openaev:/usage/inject-status/
---

Every inject that runs gets an execution status, and every step it takes on a target leaves an execution trace with a status of its own. This page lists them all, says what each one means, and explains how OpenAEV computes the status of an inject from the reports of its agents. Use it when an inject ends in **Error** or **Partial** and you need to know why.

:::warning[The execution status is not your security posture]
The execution status says whether the action ran: the command started, crashed or timed out. It says nothing about your defenses. A command that your EDR blocked can still read **Executed**. Whether an attack was detected or prevented is in the [expectations](/openaev/use/expectations/) of the inject, which collectors fill from your security tools.
:::

::screenshot{id=openaev-atomic-testing-booking-db alt="The atomic test Booking database export on booking-db-01 in Error, while its Prevention and Detection tiles read Prevented and Detected."}

## Where the statuses show

| Where | What it shows |
| --- | --- |
| The header of an inject's result page | The status of the inject. When it is **Error** or **Partial**, its tooltip gives the first error message |
| **Execution status** column of the **Injects results** list (simulation **Overview**) and of :menu[Atomic testings] | The status of each inject |
| **Execution details** tab of an inject's result | The traces of the inject as a whole: the distribution to the agents, a late start, a timeout without agents |
| **Execution** section of a target, in **Results by target** | The traces of that endpoint or agent, one line per step, with the status of each agent |
| **Terminal view** section of an endpoint or an agent | The output each step printed |

::screenshot{id=openaev-injects-status-execution alt="The Execution section of booking-db-01: the agent root in Timeout, its start and end dates, and its two traces, INFO Implant spawn by the agent, then TIMEOUT Agent root did not respond within the 10 minutes threshold."}

How to read the result page is in [Read the result of an inject](/openaev/use/injects/results/).

## Inject statuses

| Status | Meaning |
| --- | --- |
| **Draft** | The inject has not run yet. In a simulation that has started, an inject without a status shows **Pending** instead, except in a canceled simulation |
| **Queued** | The inject was launched, for example an atomic test, and waits for the platform to start it |
| **On-going** | The platform is executing the inject |
| **Pending** | The inject was handed to its agents or its injector, and OpenAEV waits for their reports |
| **Executed** | Every counted target ran the action to completion |
| **Partial** | Some counted targets ran the action, others failed |
| **Error** | No counted target ran the action, or the inject could not start |

**Executed**, **Partial** and **Error** are final: once an inject reaches one of them, its status no longer changes.

The **Injects** tab of a scenario or simulation has its own **Status** column, about the inject as you defined it, not about a run: **Enabled**, **Disabled** (the inject is skipped when the simulation runs) or **Missing content** (a mandatory field of the action is empty, and the inject cannot run).

## Trace statuses

Each agent reports its progress as traces, one per step: the prerequisite check, the installation of missing prerequisites, the attack command, the cleanup, then a last trace that gives the agent's status.

### The action ran

| Status | Meaning | What to check |
| --- | --- | --- |
| **Executed** | The command ran to completion without a system error | Nothing. It ran: it did not necessarily get past your defenses |
| **Cleanup failed** | The command ran, but the cleanup step failed | Whether the action locked a file or changed permissions. Artifacts may remain on the endpoint |
| **Executed with warning** | The command completed and wrote to its error output | The error output in **Terminal view** |
| **Access denied** | The operating system refused the command for lack of privileges | The account the agent runs under. A denial counts as a run: the agent tried and a control stopped it |

### The action failed

| Status | Meaning | What to check |
| --- | --- | --- |
| **ERROR** | A failure of no known kind, or several different errors on the same agent | The agent logs and the error output |
| **Command not found** | The program the command calls is missing on the endpoint | That the tool, such as `curl` or `powershell`, is installed and in the `PATH` |
| **Command cannot be executed** | The program exists but cannot run | Its execute permission and its processor architecture |
| **Prerequisite failed** | A prerequisite check or installation failed before the command | The prerequisites of the action, in the Threat Arsenal |
| **Invalid usage** | The command was called with wrong arguments or syntax | The arguments of the inject and of the action |
| **Timeout** | The agent did not report back in time, or the simulation timed out | See [Time limits](#time-limits) |
| **Interrupted** | The inject stopped before it completed: a system signal, a person, a lack of resources, or the simulation was stopped | The simulation's history and the endpoint |

### Not counted

These traces explain why a target did not run the action. They are left out when OpenAEV computes the status of the inject.

| Status | Meaning | What to check |
| --- | --- | --- |
| **Agent inactive** | The agent had not reported for an hour when the inject started | The connectivity of the endpoint. See [OpenAEV agents](/openaev/use/agents/) |
| **Asset agentless** | The targeted endpoint has no agent | Install an agent on it |
| **Asset not targeted** | The endpoint's marking clearance does not cover the markings of the inject | The markings of the inject and the clearance of the asset |
| **AGENT_OVERLOADED** | The agent already had too many jobs waiting, so the inject was not sent to it | The queue threshold, `openaev.agent.queue-threshold`, off (`0`) by default. See the [configuration reference](/reference/openaev/configuration/#openaev-agent) |
| **INFO** | A notice, such as the implant starting on the endpoint or the inject being distributed to its agents | Nothing |

A targeted agent whose executor cannot be found gets an **ERROR** trace: `Cannot find the executor for the agent`. It counts as a failure. So does an agent whose command OpenAEV refuses to prepare, such as a `cmd` command with a double quote in an argument value: its **ERROR** trace starts with `Unable to prepare the command.` See [When a command is refused](/openaev/use/injects/#when-a-command-is-refused).

### Statuses of older results

Results recorded by earlier versions can carry **MAYBE_PREVENTED** or **MAYBE_PARTIAL_PREVENTED**. OpenAEV no longer sets them and counts them as errors. An agent that does not report in time now gets **Timeout**.

## How OpenAEV computes the status

::diagram{id=openaev-injects-status-rollup}

### The status of an agent

OpenAEV reads every trace of one agent and keeps the first rule that applies:

1. A step other than the prerequisites and the cleanup failed: the agent takes that error, such as **Command not found**. Two different errors give **ERROR**.
2. A prerequisite check or installation failed: **Prerequisite failed**.
3. The command succeeded and the cleanup failed: **Cleanup failed**.
4. Every step succeeded: **Executed**.

### The status of the inject

OpenAEV then takes the status of each agent, leaves out the ones that are not counted, and decides:

| The counted agents | Inject status |
| --- | --- |
| All ran the action | **Executed** |
| All failed | **Error** |
| Some ran it, some failed | **Partial** |
| None: every agent was inactive, overloaded or not targeted, or no endpoint had an agent | **Error** |

The statuses that count as a run are **Executed**, **Cleanup failed**, **Executed with warning** and **Access denied**. Every status of [The action failed](#the-action-failed) counts as a failure.

:::example[Replay Velvet Harbor's PowerShell loader on the front desks]
A validation engineer of Grand Meridian Hotels runs the PowerShell (T1059.001) action of the Velvet Harbor scenario on the asset group **Front desk workstations**, which holds five endpoints with one agent each:

| Endpoint | What happened | Agent status |
| --- | --- | --- |
| `fd-paris-01` | The loader ran; the cleanup could not delete its file | **Cleanup failed**, a run |
| `fd-paris-02` | The loader ran to completion | **Executed**, a run |
| `fd-madrid-02` | Meridian EDR refused to let the script run | **Access denied**, a run |
| `fd-barcelona-03` | The implant never sent its result | **Timeout**, a failure |
| `fd-nice-04` | The workstation was off that night | **Agent inactive**, not counted |

Three counted agents ran the action and one failed: the inject reads **Partial**. The Madrid block counts as a run, because the agent did try. Whether Meridian EDR prevented the loader in Madrid and detected it in Paris is read on the prevention and detection expectations, not here.
:::

## Time limits

### A start that comes too late

When a scheduled inject starts later than its planned time plus a grace period, OpenAEV does not run it. The inject ends in **Error**, with the trace `Inject is now too old for execution`, followed by the inject's ID, its launch date and the current date. It happens when the platform could not start the inject in time, for example after a restart or under heavy load.

The grace period is 4 minutes, `openaev.scheduling.inject-staleness-threshold`, and cannot be less than 1 minute. See the [configuration reference](/reference/openaev/configuration/#openaev-scheduling).

### Agents that do not report

Once an inject is sent, OpenAEV waits for each targeted agent to report. The wait is 10 minutes by default, `INJECT_EXECUTION_THRESHOLD_MINUTES` (`inject.execution.threshold.minutes`, in the [configuration reference](/reference/openaev/configuration/#inject)). When it expires:

- each agent that did not report gets a **Timeout** trace: `Agent <user> did not respond within the 10 minutes threshold.`;
- an inject without agents, such as a network scan, gets a **Timeout** trace: `The inject did not complete within the 10 minutes threshold and was marked as timed out.`;
- OpenAEV computes the status of the inject.

### A report that comes after the timeout

An agent can still report after its **Timeout**, when the endpoint was slow or offline. The status of the inject was final and does not change. The **Timeout** trace keeps its status, and its message ends with `It reported later, see the next traces.` The traces the agent sends afterwards appear in the execution details as they came, and change neither the status nor the end date of the inject.

### An active asset that times out

The status of an agent and a **Timeout** measure two different things. An agent is active when it was seen in the last hour; for an agent run through an EDR's executor, that date comes from the EDR. A **Timeout** means the implant on the endpoint never sent its result to OpenAEV. When an active asset times out, check that:

1. The endpoint reaches the URL of OpenAEV: network, proxy, firewall and TLS certificate.
2. No antivirus or EDR blocks or deletes the implant.
3. The executor delivered the command. See [Executors](/openaev/use/executors/).

## What's next

- [Read the result of an inject](/openaev/use/injects/results/): the traces in context, with the expectations.
- [How expectations score an inject](/openaev/use/expectations/): whether the attack was prevented or detected.
- [OpenAEV agents](/openaev/use/agents/): install and monitor the agents that report these traces.
