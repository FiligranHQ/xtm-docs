---
title: How OpenAEV keeps what you test and what it finds
description: The objects behind every OpenAEV screen - scenarios, simulations, injects, targets, the Threat Arsenal, expectations and findings - which ones a tenant shares, and what a change reaches.
doc_type: explanation
products: [openaev]
personas: [validation-engineer, exercise-planner, admin]
owner: docs-openaev
last_reviewed: 2026-11-27
---

OpenAEV keeps a small set of connected objects, and every screen is a view of them. A **scenario** holds the attack you test, as **injects** aimed at **assets** and **teams**, and each inject runs an action of the **Threat Arsenal** or an inject type of an injector. Each launch copies the scenario into a **simulation**, and each run leaves **expectations**, scored per target, and **findings**, what the injects found. Some objects belong to one scenario; others are shared by the whole tenant, so a change to them reaches every scenario that uses them. This page describes each object, where it lives in the menu, and what a change to it affects.

::diagram{id=openaev-data-model-objects}

## What OpenAEV stores

| What you work with | Object | Where in the menu | Belongs to |
| --- | --- | --- | --- |
| The attack you test, written once | Scenario: its injects, teams, variables, media pressure articles, objectives and lessons learned questions | :menu[Scenarios] | The tenant |
| One run of it | Simulation: a copy of the scenario, with its status, dates and results | :menu[Simulations] | The tenant, linked to its scenario if it has one |
| One action tested on its own | Atomic testing: an inject that belongs to no scenario and no simulation | :menu[Atomic testings] | The tenant |
| A step of the attack | Inject: its type, its targets, its delay or the inject it depends on, its expectations | The **Injects** tab of a scenario or a simulation | One scenario or one simulation, never both |
| What an inject can do | An action of the Threat Arsenal, with the payload it runs on endpoints; or an inject type of an injector, such as an email or a media article | :menu[Threat Arsenal]; the injectors under :menu[Integrations] | The tenant |
| The machines and services you test | Assets: endpoints with their agents, other targets such as web applications or cloud accounts, and your security platforms | :menu[Assets], :menu[Security platforms] | The tenant |
| Machines tested together | Asset group: assets chosen by hand, and those a filter selects | :menu[Asset groups] | The tenant |
| The people you test | Persons, their teams and their organizations | :menu[Persons], :menu[Teams], :menu[Organizations] | The tenant; a contextual team, one scenario |
| What makes an exercise believable | Documents, channels and challenges | :menu[Components] | The tenant |
| What must happen for an inject to succeed | Expectation, one per target: prevention, detection, vulnerability, manual, challenge or article | The results of an inject | Its simulation or its atomic testing |
| What the injects found | Finding: a value an inject reported, such as an open port, a user name or a CVE, linked to the assets, teams or persons it concerns | :menu[Findings], and the **Findings** tab of a scenario or a simulation | The tenant |
| How things are classified | Tags, attack patterns, kill chain phases, vulnerabilities | :menu[Settings > Taxonomies] | The tenant |

Every object belongs to one tenant: a platform with several tenants keeps each one's scenarios, assets and results apart. Persons are the exception: a person is an account of the platform, a member of the tenants it can open. See [Give each organization its own tenant](/openaev/administer/multi-tenancy/).

## Owned or shared: what a change reaches

A scenario owns some of its parts and points to the others, and the difference decides how far an edit reaches:

- **Owned by the scenario**: its injects, its variables, its media pressure articles, its objectives, its lessons learned questions, and the persons enabled in each of its teams. Changing them changes that scenario only.
- **Shared by the tenant**: assets and asset groups, teams and persons, the actions of the Threat Arsenal and their payloads, documents, channels, challenges and tags. A scenario points to them. Renaming an asset, adding a person to a team or changing a payload reaches every scenario that uses it, and the next run of each.

An asset group is resolved when an inject runs, not when you build the scenario: an endpoint that joins the group, by hand or because it now matches the group's filter, is targeted by the next run. See [Add the assets your simulations target](/openaev/use/targets/assets/).

A team is global or contextual. A global team, listed in :menu[Teams], is shared by every scenario it is added to. A contextual team exists only in the scenario it was created in, carries the **Contextual** chip, and does not show in :menu[Teams]. See [Organize the people your injects target](/openaev/use/targets/people/).

## What a launch copies

Launching a scenario creates a simulation that copies what the scenario owns and points to what it shares:

| Copied into the simulation | Shared with the scenario |
| --- | --- |
| The injects, with their content, targets, delays and the links between them | The assets and asset groups the injects target |
| The variables, the articles, the objectives and the lessons learned questions | The global teams and their persons |
| The contextual teams, with the same persons | The documents, channels and challenges |
| The persons enabled in each team | The actions of the Threat Arsenal and their payloads |

The simulation also keeps a link to its scenario, which is how the scenario's **Statistics** compare its runs. So a change to the scenario's injects reaches only the next runs, while a change to a shared asset or team reaches the runs to come of every scenario. Expectations are not copied: an inject creates them when it runs. See [How a scenario becomes a simulation](/openaev/concepts/scenarios-simulations/).

## Expectations and findings: what a run leaves

When an inject runs, it creates its expectations, one per target, and they form a tree:

- **For endpoints**: one expectation per agent of each targeted asset, one per asset, and one per asset group. An agent's result rolls up to its asset, and the assets' results to their group.
- **For people**: one expectation per enabled person of each targeted team, and one per team. The persons' results roll up to their team.

The **Validation mode** of an expectation decides how a group rolls up: **All assets (per group) must validate the expectation**, or **At least one asset (per group) must validate the expectation**, and the same choice for the persons of a team. How each expectation is answered is in [How an attack becomes injects and expectations](/openaev/concepts/injects-expectations/).

A finding is a value an inject reported in its output: an open port, a share, a user name, a password policy, a CVE. It points to the inject that reported it and to the assets, teams or persons it concerns. OpenAEV records it per inject: the same value reported again by the same inject updates it, while the next run, whose injects are copies, records its own. :menu[Findings] and the **Findings** tab of a scenario or a simulation show one line per type and value, and the page of a finding gathers every finding of that type and value in the tenant: its **Occurrences**, **First seen** and **Last seen**, and the assets, teams and persons it impacts. See [How OpenAEV recognizes what it already has](/openaev/concepts/deduplication/). A finding of the CVE kind holds the CVE's identifier: the vulnerability itself, with its score and its remediation, is in :menu[Settings > Taxonomies > Vulnerabilities].

:::example[One change, seen from three screens]
The validation engineer of Grand Meridian Hotels runs the scenario `Velvet Harbor replay` every week. Its inject **PowerShell loader** targets the asset group **Front desk workstations**, and its email **Fake summit booking request** the team **Front desk France**.

- On 13 January 2026, she adds the endpoint `fd-barcelona-03` to **Front desk workstations**. The asset group is shared: the simulation of 14 January targets the new endpoint, and so does any other scenario that targets the group.
- On 15 January, she changes the delay of **PowerShell loader** in the scenario from 15 to 10 minutes. The inject is owned: the simulation of 14 January keeps 15 minutes, the simulation of 21 January plays 10.
- The simulations of 14 and 21 January both report the user name `night.audit` on `fd-paris-07`. Each run records its own finding; :menu[Findings] shows one line for `night.audit`, whose page counts 2 **Occurrences**, first seen on 14 January and last seen on 21 January.
:::

## What's next

- [How a scenario becomes a simulation](/openaev/concepts/scenarios-simulations/)
- [How an attack becomes injects and expectations](/openaev/concepts/injects-expectations/)
- [How injects reach assets and people](/openaev/use/targets/)
- [How OpenAEV recognizes what it already has](/openaev/concepts/deduplication/)
- [How access works in OpenAEV](/openaev/concepts/access/): who may see and change each object
