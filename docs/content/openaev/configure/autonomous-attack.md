---
title: Configure autonomous attack chaining
description: Set the specialist agents an autonomous attack may consult by default and how far each may go, understand objective scope modes, and steer a live run.
doc_type: how-to
products:
  - openaev
  - xtm-one
personas: [admin, ai-builder, validation-engineer]
owner: docs-openaev
last_reviewed: 2026-11-07
edition: ee
migrated_from:
  - openaev:/usage/attack-chaining/autonomous/configuration/
---

An autonomous attack is driven by the XTM One orchestrator, which can consult specialist agents, each focused on one job: building a payload, generating code, reconnaissance, exploitation support. This page sets which agents it consults by default for your tenant and how much each may create, explains the scope mode an objective carries, and shows how to steer a run while it is live. Running an autonomous attack itself is in [Let AI run a penetration test](/openaev/use/attack-chaining/autonomous/).

## Before you start

- The Enterprise Edition is active and OpenAEV is connected to XTM One: the settings page stays hidden otherwise, since the orchestrator is an XTM One capability. See [Connect the modules](/deploy/configure/connect-modules/).
- To change the tenant defaults, your role is a platform or tenant administrator. The page shows an **EE** message with an Enterprise Edition button when the license is missing.
- The agents you want to add exist in your XTM One instance.

## Set the default specialist agents

::diagram{id=openaev-autonomous-attack-agents}

The tenant defaults prefill every new autonomous attack; each run can still change them in its launch drawer.

:::steps
1. Open :menu[Settings > Customization > Autonomous attack].
2. The **Autonomous orchestrator** is always on: it plans and drives the attack and cannot be turned off.
3. Turn the specialist agents on or off. The built-in **OpenAEV Payload Creator**, marked **Built-in**, is on by default; turn it off to stop the orchestrator generating payloads on its own. Any agent of your XTM One can be a default here.
4. For each enabled agent, set its **Discovery mode**: how much it may create while it investigates.
5. To add an agent that does not exist yet, use the link to create one in XTM One; it becomes available here once it exists.
:::

The change is saved as you make it.

| Discovery mode | What the agent may do |
| --- | --- |
| **Existing only** | Only enrich assets, teams and persons that already exist and are in scope; it creates nothing |
| **In scope** | May create new assets, findings and persons, but only inside the run's allow-scope perimeter |
| **Expansive** | May create new assets, findings and persons anywhere; the deny list still wins |

A specialist agent starts at **Expansive** when you enable it; the orchestrator itself stays scoped.

::screenshot{id=openaev-autonomous-attack-settings alt="The Autonomous attack settings of the tenant: the Autonomous orchestrator, always on, and the built-in OpenAEV Payload Creator, enabled in Expansive discovery mode, with the link to create an agent in XTM One."}

## Understand objective scope modes

Every objective carries a scope mode that tells the orchestrator whether it needs a target:

- **Environment** objectives work over the whole authorized scope, such as "find and prove any path to domain admin". The AI discovers its own targets.
- **Target** objectives need a specific target, such as "exploit this web application". When the objective needs a target and you gave none, the AI raises a **Question** and waits rather than guess.

The ready-made objectives carry their mode: **Reach the Domain Controller**, **Prove Data Exfiltration**, **Validate EDR / SIEM Detections**, **Gain Initial Access** and most others are environment objectives; **Crown-Jewel Assessment** and **Web Application Exploitation** are target objectives. The mode applies the same whether you build the scenario with AI or launch it autonomously.

## Steer a live run

You do not stop a run to change its course. From the run's cockpit on the scenario page:

- **Send a directive**: an instruction such as "focus on the booking network" or "avoid the production database". The orchestrator applies pending directives on its next decision cycle and records a **Directive** entry, without stopping.
- **Edit the live configuration**: adjust the scope or the rate limit; the change reaches the live run and its simulation without a restart.
- **Answer a question**: when the run is **Waiting for input**, answer to unblock it.
- **Stop**: stops the run and its simulation.

## Check that it worked

- :menu[Settings > Customization > Autonomous attack] lists your agents with their state and discovery mode, and the launch drawer of a new autonomous attack arrives prefilled from them.
- A directive you send shows as a **Directive** entry on the timeline once the orchestrator applies it.

:::example[Scope the agents for Grand Meridian's night tests]
Hélène Marchetti's team runs autonomous attacks at night and wants them to map the real environment without inventing assets outside the test perimeter.

| Setting | Value |
| --- | --- |
| **Autonomous orchestrator** | On (always) |
| **OpenAEV Payload Creator** | On, **Discovery mode** **In scope** |
| A custom recon agent from XTM One | On, **Discovery mode** **Existing only** |

Every new autonomous attack of the tenant then starts with these agents. On a run against the booking network, the team sends the directive "avoid 10.20.8.40, the payment gateway" mid-run; the orchestrator applies it on its next cycle, and the timeline records it.
:::

:::xtm-one[The agents live in XTM One]
The specialist agents, including the ones you create, are XTM One agents. This page selects which the orchestrator may consult for OpenAEV's autonomous attacks and bounds what they may create; their instructions and governance are in XTM One. See [XTM One in OpenAEV](/xtm-one/use/in-openaev/).
:::

## What's next

- [Let AI run a penetration test](/openaev/use/attack-chaining/autonomous/): launch and watch a run.
- [Set the targets and limits of a chained scenario](/openaev/use/attack-chaining/scope/): the scope a run enforces.
- [How a chained scenario works](/openaev/use/attack-chaining/): the chain an autonomous run builds on.
