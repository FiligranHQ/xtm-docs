---
title: Check one attack technique with an atomic test
description: Run a single action of the Threat Arsenal on chosen endpoints, read at once whether it was prevented and detected, and replay it on a schedule to check that a defense keeps working.
doc_type: how-to
products:
  - openaev
personas: [validation-engineer, soc-analyst]
owner: docs-openaev
last_reviewed: 2026-11-07
migrated_from:
  - openaev:/usage/atomic/
---

An atomic test runs one action, such as a PowerShell payload or a phishing email, on the targets you pick, and scores it alone: was it prevented, was it detected. It answers a precise question in minutes, without a scenario: does the new EDR rule block this technique, does this payload work before I add it to a scenario. Scheduled, it replays the same action every day or week, so you know when a defense stops working. To replay a whole attack, build a scenario instead: see [Build a scenario that replays an attack](/openaev/use/scenarios/build/).

::diagram{id=openaev-atomic-testing-flow}

## Before you start

- Your role has **Manage assessment** to create an atomic test, and **Launch assessment** to launch, relaunch or schedule it. **Delete assessment** deletes it.
- The action exists in the Threat Arsenal: see [Threat arsenal](/openaev/use/threat-arsenal/).
- The endpoints run an OpenAEV agent or another executor, and a collector reads the security tool to measure. See [OpenAEV agents](/openaev/use/agents/) and [Collectors](/openaev/use/collectors/).

## Create and launch an atomic test

:::steps

1. Open :menu[Atomic testings] and click **Create**. The **Create a new atomic test** page lists the actions an atomic test can run. An action whose injector is not registered in your tenant is not listed, since it would fail at launch.
2. Find the action: filter on the left by **Domains**, **Platform**, kill chain phase or author, or search by name or MITRE ATT&CK ID. **Grid view** and **List view** switch the display.
3. Click the action. A panel opens with its **Title**, filled with the action's name.
4. Fill in the targets and the content: the endpoints or asset groups for an action on endpoints, the **Targeted teams** for an email, and the **Inject data** the action needs, such as a command argument.
5. Under **Inject expectations**, keep the expectations to score, such as prevention and detection.
6. Click **Create**. The atomic test opens.
7. Click **Launch now**. The action runs at once.

:::

When the action is not ready, for example when a required field is empty, **Configure** shows instead of **Launch now**: it opens the panel to complete it.

::screenshot{id=openaev-atomic-testing-create alt="The Create a new atomic test page: the actions of the Threat Arsenal to pick from, with the filters by domain and platform."}

## Read the result

The **Overview** tab scores the test like a simulation: a gauge per axis, and each target with its results. The test is done when every expectation is answered or has expired: 6 hours by default for detection and prevention. See [Read the results](/openaev/use/simulations/#read-the-results).

::screenshot{id=openaev-atomic-testing-booking-db alt="The atomic test Booking database export on booking-db-01: Prevented and Detected by Meridian EDR, its status Error because the EDR killed the payload as it ran, with its start and end dates and its duration."}

The **Targets** pane lists what the action aimed at, one tab per kind: **Asset groups**, **Teams**, **Assets**, **Persons**, **Agents**, and **Manual targets**. Click a target to read its expectations in **Results by target**. An action that aims at no target, such as an HTTP request, has nothing to list per target: the **Targets** pane reads **This inject has no target. Its results are in the Execution details tab.** Read its expectations and traces there.

A network scan such as Nmap or Nuclei can aim at addresses typed by hand instead of assets: **Type of targets** set to **Manual**, and the IP addresses, subnets or hostnames in **Manual targets (comma-separated)**. The **Manual targets** tab lists each of them once, in the order typed. A manual target carries no expectation and no execution trace of its own, since the scan reports for all of them at once: read what it found in the **Findings** tab.

- **Execution details**: the expectations of each target, and the traces of the execution on each endpoint, such as the command run and its output.
- **Action details**: the payload that ran, for a payload action.
- **Findings**: what the action found, such as open ports or credentials.
- **Remediations**: the detection rules that would catch the action, for a payload action. It requires the Enterprise Edition.

:menu[Atomic testings] lists every test with its **Execution status**, its **Target** and its **Global score**, to see at a glance which techniques your tools handle.

## Relaunch it or run it on a schedule

- **Relaunch now** runs it again. The new run replaces the previous one: its results are deleted. Tick **Realign expectations to the current action before relaunching** when the action changed in the Threat Arsenal since the test was created.
- **Scheduling** opens the same dialog as for a scenario: **Once**, **Hourly**, **Daily**, **Weekly** or **Monthly**, a **Start date** and an **End date (optional)**. The header then reads **Scheduled**, and **Stop** removes the schedule. OpenAEV launches the test a minute before each time, unless the previous run is still waiting to start.

To schedule a test through the API, give its recurrence as a Spring cron of six fields, seconds first, read in UTC, such as `0 0 2 * * *`, or as an ISO 8601 period such as `P1D`. OpenAEV refuses another expression, such as a five-field Unix cron, with an error that names these two forms.

Each scheduled run replaces the results of the one before. To keep the history of a technique's results, run it in a scheduled scenario instead: each simulation keeps its own results.

The menu of the header also offers **Update**, **Duplicate**, the JSON export and **Delete**. **Import JSON**, at the top of :menu[Atomic testings], imports an exported test.

:::example[Check every night that Meridian EDR blocks the Velvet Harbor loader]
After the night of 9 January 2026, the SOC of Grand Meridian Hotels added a Meridian EDR rule against the PowerShell loader Velvet Harbor ran on a front-desk workstation. The validation engineer wants to know the night that rule stops working.

| Field | Value |
| --- | --- |
| Action | **Velvet Harbor - booking attachment loader**, the PowerShell (T1059.001) action of the Threat Arsenal that replays the loader |
| **Title** | `Velvet Harbor loader on the front desks` |
| Asset groups | `Front desk workstations` |
| **Inject expectations** | Prevention and detection |
| **Scheduling** | **Daily**, **Time** 03:00, from 15 January 2026 |

The first run reads **Prevention** 100 and **Detection** 100. On the morning a Meridian EDR update reverts the rule, the **Overview** shows **Not Prevented** on the workstations it reached, and the SOC knows before an attacker does.
:::

:::xtm-one[Get a detection rule for what was missed]
With the Enterprise Edition, the **Remediations** tab of a payload action offers **Generate with AI** when XTM One is connected: an XTM One agent writes a detection rule for the action, for the security platforms that support it. Review it before you deploy it. See [XTM One in OpenAEV](/xtm-one/use/in-openaev/).
:::

## What's next

- [Build a scenario that replays an attack](/openaev/use/scenarios/build/), to add the action to a whole attack.
- [Inject results](/openaev/use/injects/results/), for every part of a result.
- [Share simulation results in a report](/openaev/use/simulations/reports/), with **Atomic testing** as the subject.
