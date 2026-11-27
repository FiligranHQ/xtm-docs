---
title: How a chained scenario works
description: How a chained scenario runs each step only when the results of the earlier ones allow it, inside the targets and limits you set, and draws the attack path it took.
doc_type: explanation
products:
  - openaev
personas: [validation-engineer, soc-manager]
owner: docs-openaev
last_reviewed: 2026-11-07
edition: ee
migrated_from:
  - openaev:/usage/attack-chaining/overview/
---

A chained scenario runs an attack the way an intruder does: it starts with one step, reads what the step found, and decides from that result which steps come next. A time-based scenario plays a fixed list of injects at fixed times, whatever happens; a chained one only runs lateral movement once a reachable host is found, and only tries an account once one has been harvested. This page explains what a chained scenario is made of, how a run unfolds, and what bounds it. The pages under it walk through each task, and the attack path it draws shows how far an attacker got.

## What a chained scenario is made of

::diagram{id=openaev-attack-chaining-cycle}

You choose the type when you create the scenario, or a simulation without a scenario: **Chained scenario** (or **Chained simulation**) instead of **Time-based scenario**. A chained scenario has its own tabs, **Overview**, **Scope**, **Logic**, **Attack Path**, **Execution**, **Findings** and **Statistics**, in place of **Injects** and **Tests**.

| Part | What it is | Where you set it |
| --- | --- | --- |
| **Scope** | The targets the run may touch (the allow list), those it must never touch (the deny list), variables, and the run's limits: a time out and a rate limit | **Scope** tab. See [Set the targets and limits of a chained scenario](/openaev/use/attack-chaining/scope/) |
| **Action** | A step that runs an action of the Threat Arsenal: a command on an endpoint, a scan, an email | **Logic** tab. See [Choose the actions of a chained scenario](/openaev/use/attack-chaining/actions/) |
| **Event** | A set of conditions on what the actions found, such as "a port equals 1433". When they hold, the event triggers the actions linked to it | **Logic** tab. See [Build the logic of a chained scenario](/openaev/use/attack-chaining/logic/) |
| **Attack path** | The graph of what really ran: actions, the endpoints they reached, the findings they produced and the events those findings triggered | **Attack Path** tab. See [Read how far a chained simulation got](/openaev/use/attack-chaining/attack-path-map/) |

Chained scenarios and simulations require the Enterprise Edition: without it, the **Chained scenario** card shows an **EE** chip and opens the Enterprise Edition dialog. See [Editions](/start/editions/).

::screenshot{id=openaev-attack-chaining-overview alt="The chained scenario Velvet Harbor path - front desk to booking database, not run yet: its tabs Overview, Scope, Logic, Attack Path, Execution, Findings and Statistics, and the Normal and Autonomous launch buttons."}

## How a run unfolds

When you launch a chained scenario, OpenAEV creates a simulation and starts its run:

1. The entries of the allow list and the scope variables become the first values of the run: the IP addresses, subnets, domains and endpoints the run may use.
2. The actions that wait on no event start, on the endpoints of the allow list.
3. When an action ends, OpenAEV keeps its outputs: the ports, hosts, accounts or vulnerabilities its output parsers extracted, its execution status and its expectation results. A value that the scope does not allow, such as an IP address of the deny list, is dropped.
4. Every event checks its conditions against those values. An event whose conditions hold triggers the actions linked to it, once for each combination of values they can use.
5. The run ends when no action is left to run, when you stop it, or at its time out.

An action runs only when the event before it holds: if no step finds a reachable database server, nothing tries to query one. An action does not run twice with the same values on the same target. The **Execution** tab follows the injects as they move from **Up next** to **In flight** to **Completed**; the list is not known in advance, unlike the injects of a time-based simulation.

A launched simulation keeps the logic and the scope it started with: its **Logic** tab is read-only. To change the chain, reset the simulation, or change the scenario and launch it again.

## What keeps a run under control

| Safeguard | What it does |
| --- | --- |
| **Allow list** | A chained scenario cannot be launched without one: the launch button stays disabled with **A chained scenario requires a defined scope.** Values an action discovers outside of it do not feed the next steps |
| **Deny list** | Takes priority over the allow list: a target in both is never reached |
| **Simulation time out** | Ends the run after a total duration, one hour for a new scenario. Paused time does not count |
| **Simulation rate limit** | Caps how many injects start in a time window, to model a slow attacker or stay under a lockout threshold |
| **Read-only logic once launched** | What ran stays the record of what ran |

## Chained scenarios and the alternatives

| You want to | Use |
| --- | --- |
| Replay a known attack at set times, step by step, whatever the results | A time-based scenario. See [Build a scenario](/openaev/use/scenarios/build/) |
| Make one inject wait for the result of another, in a time-based scenario | Inject chaining. See [Chain injects](/openaev/use/injects/chaining/) |
| Branch on what each step found, reuse its outputs, bound the run by a scope | A chained scenario, this page |
| Let an AI plan and adapt the chain toward an objective | An autonomous run of a chained scenario. See [Let AI run a penetration test](/openaev/use/attack-chaining/autonomous/) |

:::xtm-one[Build or drive the chain with XTM One]
With OpenAEV connected to XTM One, the **AI builder** of a chained scenario writes the logic for you from an objective in plain words, and the **Autonomous** launch lets the XTM One orchestrator drive the run live, adding steps from what it finds. See [Let AI run a penetration test](/openaev/use/attack-chaining/autonomous/).
:::

## What's next

- [Set the targets and limits of a chained scenario](/openaev/use/attack-chaining/scope/)
- [Choose the actions of a chained scenario](/openaev/use/attack-chaining/actions/)
- [Build the logic of a chained scenario](/openaev/use/attack-chaining/logic/)
- [Read how far a chained simulation got](/openaev/use/attack-chaining/attack-path-map/)
- [Let AI run a penetration test](/openaev/use/attack-chaining/autonomous/)
