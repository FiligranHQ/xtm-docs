---
title: Add injects to scenarios and simulations
description: Create the injects of a scenario or simulation from the Threat Arsenal, set when they run, whom or what they target and what they expect, then manage them in the Injects tab.
doc_type: how-to
products:
  - openaev
personas: [validation-engineer, exercise-planner]
owner: docs-openaev
last_reviewed: 2026-11-07
migrated_from:
  - openaev:/usage/inject-overview/
---

An inject is one action of a simulation: an email to a team, a command on endpoints, a network scan, a media article. Each one comes from an action of the Threat Arsenal, runs at a set delay after the start of the simulation, reaches its targets, and is scored by its expectations. This page adds injects to a scenario or a simulation and manages them in the **Injects** tab. To run a single inject on its own, use an [atomic test](/openaev/use/atomic-testing/). The ideas behind injects are in [Injects and expectations](/openaev/concepts/injects-expectations/).

::diagram{id=openaev-injects-anatomy}

## Before you start

- Your role has **Manage assessment**, or the right to manage the scenario or simulation.
- The action you need is in the Threat Arsenal, and its injector or executor is connected. See [The Threat Arsenal](/openaev/use/threat-arsenal/).
- The targets exist: teams and persons for injects to people, assets and asset groups for injects on endpoints, each with an [OpenAEV agent](/openaev/use/agents/). See [Targets](/openaev/use/targets/).

A scenario is the plan, and each simulation launched from it takes a copy of its injects. An inject you add to a simulation stays in that simulation; one you add to the scenario reaches the simulations launched afterwards. A scenario or simulation built with attack chaining, an Enterprise Edition feature, has no **Injects** tab: its actions are in **Logic**. See [Attack chaining](/openaev/use/attack-chaining/).

## Create an inject

:::steps

1. Open the scenario or the simulation, then its **Injects** tab, and click **Create**. The **Create a new inject** picker opens on the actions of the Threat Arsenal. It lists only the actions whose injector is registered in your tenant: an action without one would fail at launch, so it does not show.
2. Find the action: type part of its name or a technique in the search bar, or filter by **Domains**, **Platform**, **Status** or **Author**. Switch between **Grid view** and **List view** to compare **Attack patterns** and **Kill chain phase**.
3. Click the action's card. Its configuration form opens.
4. Under **General**, give the inject a **Title**, and optionally a **Description** and **Tags**.
5. Under **Scheduling**, set the **Days**, **Hours** and **Minutes** after the start of the simulation when the inject runs.
6. Choose the targets. For an action on people, under **Targeted teams**, click **Modify target teams** and pick the teams, or turn on **All teams**. For an action on endpoints, pick the **Assets** or **Asset groups** under **Inject data**. A field marked as required must be filled.
7. Under **Inject data**, fill in what the action needs: the subject and body of an email, the arguments of a command. Click **Available variables** to insert a variable, such as the first name of each person. An argument that takes a file is a list of the platform's [documents](/openaev/use/components/documents/): pick the file there. The reset button puts back the action's defaults. For a command that runs in `cmd`, an argument value cannot hold a double quote (`"`): see [When a command is refused](#when-a-command-is-refused).
8. Under **Inject expectations**, check the expectations the action proposes, and change or add some. See [Set the expectations of an inject](/openaev/use/expectations/management/).
9. Under **Inject documents**, attach files when the action sends them.
10. Click **Create**.

:::

The inject appears in the list with the status **Enabled**. To add several actions at once, tick their cards in the picker and click **Add 2 injects**, or the number you selected: they are created with the action's defaults and show **Missing content** until you fill in their mandatory fields.

## Manage the injects

The **Injects** tab lists the injects with their **Type**, **Title**, **Trigger** delay, **Platform(s)**, **Status** and **Tags**. Change the view with the buttons at its top: **List view**, **Interactive view**, a timeline where you can also chain injects, or **Distribution view**, charts of the injects by type and by team. In a simulation, the **Distribution view** also charts the scores of the expectations as they come in. **Distribution of expected total score by inject type** counts every kind of inject: a team's result for an email, the result of each asset group or directly targeted asset for a command, each counted once.

The menu at the end of a line holds:

| Action | What it does |
| --- | --- |
| **Update** | Opens the inject's form, with its **Logical chains** tab. See [Run injects on conditions and move them between scenarios](/openaev/use/injects/chaining/) |
| **Duplicate** | Creates a copy |
| **Test** | Sends an email or SMS inject to you alone. See [Try email and SMS injects on yourself before a simulation](/openaev/use/injects/tests/) |
| **Exporter (JSON)** | Downloads the inject as a ZIP archive, to import it elsewhere |
| **Disable** / **Enable** | A disabled inject stays in the list, editable, and never runs |
| **Delete** | Deletes the inject |

Tick several injects to act on them together: the toolbar exports them, adds, replaces or removes their assets, asset groups or teams, tests them or deletes them. The **Import injects** button at the top of the list brings injects from an export, with **Import JSON**, or from a spreadsheet, with **XLS import (mapper)**. Both need only the right to manage the scenario or simulation: an XLS mapper is created in the settings, but any editor of the scenario or simulation can pick it. See [Run injects on conditions and move them between scenarios](/openaev/use/injects/chaining/#reuse-injects-elsewhere) and [Import a timeline of injects from a spreadsheet](/openaev/use/scenarios/import/).

When the action of an inject changed in the Threat Arsenal after you created it, the header shows **Review expectations**. See [Set the expectations of an inject](/openaev/use/expectations/management/#realign-expectations-with-their-action).

### When a command is refused

OpenAEV passes the arguments of a command to the shell as variables, so that a value never changes the command itself. A `cmd` command cannot carry a double quote in such a value, so OpenAEV refuses to prepare it. The agent then gets an **ERROR** trace that starts with `Unable to prepare the command.` and names the argument, and its execution ends at once. It counts as a failure: when every agent is refused, the inject ends in **Error**. Remove the double quote from the argument, or use an action that runs in PowerShell. How the status of an inject is computed is in [Inject execution statuses](/openaev/use/injects/status/).

:::example[Replay Velvet Harbor against the front desks]
Example CERT builds the scenario **Velvet Harbor replay** for Grand Meridian Hotels, to check Meridian EDR and the Meridian email gateway against the techniques of the 9 January 2026 intrusion. The validation engineer adds, in the **Injects** tab:

| Title | Action searched in the picker | Scheduling | Targets |
| --- | --- | --- | --- |
| `Fake summit booking request` | An email action | 0 days, 0 hours, 0 minutes | **Targeted teams**: **Front desk France**, **Front desk Spain** |
| `PowerShell loader` | `T1059.001` | 0 days, 0 hours, 15 minutes | **Asset groups**: **Front desk workstations** |
| `Local guest records` | `T1005` | 0 days, 0 hours, 30 minutes | **Asset groups**: **Front desk workstations** |
| `HTTPS beacon` | `T1071.001` | 0 days, 0 hours, 45 minutes | **Asset groups**: **Front desk workstations** |

Each inject keeps the expectations its action proposes; for the three commands, Meridian EDR must prevent and detect them. When the simulation runs, the four injects go out at the start, then after 15, 30 and 45 minutes.
:::

::screenshot{id=openaev-injects-list alt="The Injects tab of Velvet Harbor replay: the email Fake summit booking request at 0 minutes, then the PowerShell loader, the local guest records and the HTTPS beacon at 15, 30 and 45 minutes, each with its domain and Enabled."}

## What's next

- [Read the result of an inject](/openaev/use/injects/results/).
- [Run injects on conditions and move them between scenarios](/openaev/use/injects/chaining/).
- [Set the expectations of an inject](/openaev/use/expectations/management/).
- [Inject execution statuses](/openaev/use/injects/status/).
