---
title: Let AI run a penetration test
description: Give an objective in plain words and let the XTM One orchestrator plan, run and adapt a chained attack against your authorized scope, with proof of what it reached.
doc_type: how-to
products:
  - openaev
  - xtm-one
personas: [validation-engineer, ai-builder]
owner: docs-openaev
last_reviewed: 2026-11-07
edition: ee
migrated_from:
  - openaev:/usage/attack-chaining/autonomous/overview/
  - openaev:/usage/attack-chaining/autonomous/scenario-creation/
---

An autonomous run is an AI-driven penetration test. You give an objective in plain words, such as "from the front-desk workstation, reach the booking database server", and the orchestrator in XTM One turns it into a chain, runs it against your authorized scope, reads what it finds, and decides the next step on its own, until it reaches the objective or needs you. It is a way to launch a chained scenario, not a different kind: you let the AI build and drive the logic instead of authoring it. This page runs your first autonomous attack. How a chained scenario works by hand is in [How a chained scenario works](/openaev/use/attack-chaining/).

## Before you start

| Requirement | Why |
| --- | --- |
| **Enterprise Edition** | Autonomous attack chaining is an AI feature, Enterprise Edition like every AI feature of OpenAEV. See [Editions](/start/editions/) |
| **OpenAEV connected to XTM One** | The orchestrator runs in XTM One. OpenAEV runs the steps, enforces the scope and shows the run. See [Connect the modules](/deploy/configure/connect-modules/) |
| **Manage assessment**, and **Launch assessment** to start a run | The run is created and launched like any chained scenario |
| **Installed injectors and actions** | The AI can only run techniques your Threat Arsenal supports. A missing one becomes advice, not a silent failure |

The **Autonomous** button and the **AI builder** appear only when XTM One is connected and the Filigran AI terms are accepted. Without the Enterprise Edition, they show an **EE** chip and open the Enterprise Edition dialog.

## How an autonomous run works

::diagram{id=openaev-attack-chaining-autonomous-loop}

The run splits across the two platforms:

- **OpenAEV** holds the run, executes its injects through a chained simulation, enforces the scope, and shows the live attack path and the decision timeline.
- **XTM One** hosts the orchestrator, which plans the attack, decides each step, calls back into OpenAEV to run it, reads the results, and consults the specialist agents you allow.

The orchestrator streams what it does to a **decision timeline** beside the graph. Each entry has a type:

| Entry | What it is |
| --- | --- |
| **Narration** | Free-form reasoning as it thinks |
| **Decision** | A choice it made: a technique, a target, a pivot |
| **Action** | A call into OpenAEV: recon, launch an inject, read the scope |
| **Handover** | A switch between its sub-agents |
| **Agent delegation** | Consulting a specialist agent, such as the payload creator |
| **Capability gap** | No installed action can run a technique it needs |
| **Question** | It is blocked and asks you; the run waits for your answer |
| **Proof** | A confirmed compromise, kept as evidence for the report |
| **Directive** | A steering instruction you sent that it applied |
| **Status** | A change of the run's status |

A run moves through **Created**, **Running**, **Waiting for input** (it asked a question), then **Completed**, **Failed** or **Canceled**. A build-only session uses **Planning** then **Plan ready**.

## Launch an autonomous attack

There are three ways in, all opening the same drawer, an **objective**, the **agents** the orchestrator may consult, and an **allow and deny scope**. Choose how much you want to set before anything runs.

:::steps
1. **Create the scenario.** From the scenarios list, click **Create scenario**, pick **Chained scenario**, and turn on **Generate with AI**. Name it and create it: it opens with the AI builder drawer already open.
2. **Set the objective.** Pick a ready-made objective from the gallery, such as **Reach the Domain Controller** or **Prove Data Exfiltration**, or write your own in **Objective (free text)**. An **Environment** objective lets the AI find its own targets; a **Target** objective needs one, and the AI asks for it if you leave the scope empty.
3. **Choose the agents and the time budget.** Keep the built-in **OpenAEV Payload Creator** on so the orchestrator can build payloads on demand, add any agent of your XTM One, and set the **Timeout (hours)**, the ceiling of the run.
4. **Scope the perimeter.** Add the targets to the **Allow list** and the carve-outs to the **Deny list**, which always wins. Leave them empty and the AI asks which targets are in scope before it attacks.
5. **Build or launch.**
   - **Build** writes the chain onto the scenario's **Logic** tab without running anything. You review and edit it, then launch it yourself, in **Normal** or **Autonomous** mode. On a scenario that already has logic, this reads **Rebuild with AI**, with a choice between **Refine the existing logic** and **Rebuild from scratch**.
   - **Autonomous** launches a live run at once: the orchestrator seeds itself with whatever logic the scenario holds and drives it, adapting from what it finds.
   - **Save** keeps the objective, agents and scope on the scenario to build or launch later.
:::

::screenshot{id=openaev-attack-chaining-autonomous-launch alt="The Launch in autonomous mode drawer on its Objective step, before the Allow list and Deny list steps: the gallery of ready-made objectives, from Reach the Domain Controller to Assumed-Breach Assessment, above the Objective (free text) field."}

The compact **AI builder** button on any chained scenario opens the same drawer to **Save** or **Build**. The **Autonomous** button next to **Normal** opens it to **Launch**.

## Watch, steer and review

While the run is live, the scenario's page is its cockpit: the overview, the live attack path and an always-open reasoning panel, with a stop control. The manual scope and logic are read-only, since the AI owns them.

- **Steer it.** Send a directive, such as "focus on the booking network" or "avoid the payment gateway": the orchestrator applies it on its next cycle, as a **Directive** entry, without stopping. You can also adjust the scope or the rate limit live.
- **Answer a question.** When it reaches **Waiting for input**, answer in the panel to unblock it.
- **Read the proof.** On **Completed**, the **Proof of exploitation** panel lists every confirmed compromise. A proof counts only when at least one finding backs it; one without a finding is flagged as unverified. The capability gaps recorded along the way show beside the proofs. **Export report** downloads the run as a Markdown file.

A slow step, such as a phishing email that waits for a click, does not block the run: the orchestrator moves on and folds the result back in when it resolves.

## What you see when it worked

- The decision timeline fills with decisions, actions and, when the AI proves access, **Proof** entries.
- The attack path graph grows live as the orchestrator runs steps.
- The run reaches **Completed**, with a numbered **Proof of exploitation** panel and, where techniques were missing, a list of capability gaps with the integration that would close each one.

:::example[Let the AI reach the booking database of Grand Meridian]
After Velvet Harbor's loader ran on the front-desk workstation `fd-paris-01`, the validation team asks how far an attacker could go from there toward the booking database server.

| Step | Value |
| --- | --- |
| **Create scenario** | **Chained scenario**, **Generate with AI** on, name `Velvet Harbor path - front desk to booking database` |
| **Objective** | Free text: "Starting from the front-desk workstation in scope, reach and prove access to the booking database server. Record every hop as proof." |
| **Agents** | **OpenAEV Payload Creator** on |
| **Timeout** | 4 hours |
| **Allow list** | `fd-paris-01` and the subnet `10.20.8.0/24` |
| **Deny list** | `10.20.8.40`, the payment gateway |

The team clicks **Autonomous**. The timeline shows recon from `fd-paris-01`, a **Decision** to test the saved front-desk credential, a **Proof** when `MERIDIAN\night.audit` opens the booking database on `10.20.8.15`, and a **Capability gap** where a technique needed an injector they have not installed. The run reaches **Completed**; Hélène Marchetti's team exports the report as the evidence that the single saved credential on the front desk is the hop to close.
:::

:::xtm-one[The orchestrator and its agents]
The orchestrator and the specialist agents it consults are XTM One agents. Which agents it may use by default, and how much each may create while it investigates, is set per tenant. See [Configure autonomous attack chaining](/openaev/configure/autonomous-attack/).
:::

## What's next

- [Configure autonomous attack chaining](/openaev/configure/autonomous-attack/): objectives, the specialist agents and steering a live run.
- [How a chained scenario works](/openaev/use/attack-chaining/): the chain an autonomous run builds on.
- [Read how far a chained simulation got](/openaev/use/attack-chaining/attack-path-map/): the attack path the run draws.
