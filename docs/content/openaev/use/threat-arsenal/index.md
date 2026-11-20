---
title: Build the actions of your threat arsenal
description: Create, import and manage the actions OpenAEV runs in injects, understand where they come from, and turn a selection into a scenario or an atomic test.
doc_type: how-to
products:
  - openaev
personas: [validation-engineer]
owner: docs-openaev
last_reviewed: 2026-11-18
migrated_from:
  - openaev:/usage/build/threat-arsenals/threat-arsenals/
---

The **Threat Arsenal** is where you manage every action OpenAEV can run. An action is what happens when an inject executes on a target: a command, an executable, a file dropped on a host, or a host name resolved. It unifies what used to be payloads and injector contracts into one list, so a scenario, an atomic test or a chained scenario all draw their steps from the same place. This page explains where actions come from, creates one, and turns a selection into a test. The fields of the form are in [Action properties](/openaev/use/threat-arsenal/action-properties/).

## Where actions come from

::diagram{id=openaev-threat-arsenal-sources}

An action has one of three sources, which decides what you may change:

| Source | What it is | What you can edit |
| --- | --- | --- |
| **User-created** | Built in the Threat Arsenal, run by the OpenAEV Implant Injector | Every field |
| **Injector-provided** | Added by an injector of the platform, such as Nuclei | Domains, attack patterns and tags only |
| **Collector-provided** | Added by a collector, such as Atomic Red Team | Nothing: the collector owns them |

The list, under :menu[Threat Arsenal], shows each action's **Type**, **Name**, **Domains**, **Platform**, **Tags**, **Status** and **Updated** date. The **Status** says how far OpenAEV trusts the action:

- **Verified**: OpenAEV tested it and it works as expected.
- **Unverified**: OpenAEV has not tested it; it may or may not work.
- **Deprecated**: its source marked it deprecated; it stays for reference, with no guarantee.

Filter the list by **Action type**, **Platform**, **Domains**, **Status** and **Author**, in the sidebar. The author of an action is a person or an organization, never a team: the person who created or duplicated it, or the organization behind the injector or collector that provides it. **No author** isolates the actions without one, such as those the platform creates on its own. Reading **Threat Arsenal** takes **Access threat arsenal**. A user without it still sees :menu[Threat Arsenal] when one of their groups has a grant on a single action, **Access** or **Manage+Delete**, in its **Threat Arsenal** grants: the list then shows only the actions granted to them.

::screenshot{id=openaev-threat-arsenal-list alt="The Threat Arsenal list with the three Velvet Harbor actions, booking attachment loader, local guest records collection and HTTPS beacon, Verified, next to the actions the platform ships."}

## Create an action

:::steps
1. Open :menu[Threat Arsenal] and click the create button.
2. On the **General** tab, set the **Name**, and add a **Description**, **Attack patterns**, **Tags**, **Domains** and the **Expectations** the action raises, such as **Prevention** and **Detection**.
3. On the **Commands** tab, choose the **Type**: **Command Line** (a command run through an executor such as PowerShell or Bash), **Executable** (a binary run as its own process), **File Drop** (a file placed on the host without running it) or **DNS Resolution** (host names resolved to IP addresses). Set the **Platforms** and the command, and add a cleanup executor and command to undo what the action leaves behind.
4. On the **Output** tab, add output parsers to pull structured values from the raw output and, when you want, generate findings. See [Turn an action's output into findings](/openaev/use/threat-arsenal/output-parsers/). This tab is optional.
5. On the **Remediation** tab, write the detection rules a security platform would use to catch the action. This tab requires the Enterprise Edition. See [Action properties](/openaev/use/threat-arsenal/action-properties/#detection-remediation-properties).
6. Create the action. It appears in the list, with the **Verified** status.
:::

Creating an action with a payload also makes a new inject type available, when your implant supports it; the OpenAEV Implant does. Every field of every tab is in [Action properties](/openaev/use/threat-arsenal/action-properties/). Creating and editing actions takes **Manage threat arsenal**.

::screenshot{id=openaev-threat-arsenal-general alt="The Create a new action panel on its General tab: name, description, attack patterns, tags, domains and expectations, Prevention and Detection by default, and the security platforms expected to fulfil each, with the Commands, Output and Remediation tabs."}

## Update, duplicate or delete an action

What you may do depends on the source:

- **User-created**: open the action to edit any field, duplicate it, or delete it. Editing and duplicating take **Manage threat arsenal**; deleting takes **Delete threat arsenal**.
- **Injector-provided**: edit only its domains, attack patterns and tags. You cannot delete it unless its source marked it deprecated.
- **Collector-provided**: read-only; the collector manages it.

Without these capabilities, a group can still be given one action: a **Manage+Delete** grant on it, in the group's **Threat Arsenal** grants, shows **Update** and **Delete** in that action's menu. Duplicating always takes **Manage threat arsenal**. See [How access works in OpenAEV](/openaev/concepts/access/).

OpenAEV refuses to delete an action that injects still use, in a scenario, a simulation or an atomic testing: deleting it would delete those injects. The message names up to three of them; move their injects to another action or remove them, then delete the action. A bulk **Delete** from the toolbar skips such actions and deletes the others.

## Run a test from the list

Select one or more actions with the checkboxes to open the toolbar, then **Run a test**:

| Option | What it does |
| --- | --- |
| **Create a new scenario** | Opens the scenario funnel with the selected actions prefilled as inject steps |
| **Add to existing scenario** | Adds the selected actions as steps to one or more existing scenarios |
| **Run atomic test** | Runs a single selected action at once, as a one-off simulation |

The toolbar also **Delete**s user-created actions in bulk, **Tag**s several at once, and **Export**s them. A chained scenario reuses actions too, as steps of its logic: see [Choose the actions of a chained scenario](/openaev/use/attack-chaining/actions/).

## Import and export actions

Export the current list as a **CSV (Comma-Separated Values)** file, which carries the columns the page shows, for any action. Export user-created actions as a JSON ZIP archive instead, which carries the whole action, command, arguments and output parsers, to import on another platform. The archive also holds the file of an **Executable** or **File Drop** action, and importing it attaches that file to the new action. The import button takes such an archive. Import the archive as it was exported: a file changed or renamed inside it no longer matches its name, and the action is imported without it.

Importing a scenario or a simulation also brings the actions of its injects. An action a collector provides, such as Atomic Red Team, is matched by the identifier its collector gives it. Any other action is reused only when your tenant holds an identical one, and created otherwise. Identical means the same type, name, description, platforms, architecture, command or attached file, cleanup, arguments, prerequisites, expectations, output parsers, detection rules and attack patterns, on an action you can read. An action that differs in one detail, such as a changed command, is imported as a new one and never replaces yours.

## What you see when it worked

- The new action shows in the list, **Verified**, with the type, domains and platforms you set.
- A new inject type matching the action is available when you build an inject.
- Selecting the action and running an atomic test executes it once and shows its result.

:::example[A credential-search action for the front-desk test]
The validation team of Grand Meridian builds the step that looks for a saved booking credential on a front-desk workstation, to reuse in the night test against the booking database.

| Field | Value |
| --- | --- |
| **Name** | `Look for saved booking credentials` |
| **Domains** | **Endpoint** |
| **Attack patterns** | Credentials from Password Stores (T1555) |
| **Type** | **Command Line**, executor **Command Prompt**, command `cmdkey /list` |
| **Platforms** | **Windows** |
| **Expectations** | **Prevention**, **Detection** |
| **Output** | A parser of type **Username** on the output of `cmdkey`, **Create finding(s)** on |

Once created, the team selects it and three others, clicks **Run a test** and **Create a new scenario**, to build the chained scenario that walks from the front desk to the booking database.
:::

:::xtm-one[Generate rules and payloads with XTM One]
With OpenAEV connected to XTM One, the **Remediation** tab generates detection rules for CrowdStrike and Splunk with AI, and an autonomous attack lets the orchestrator build a payload on demand for a technique your arsenal does not cover. See [Set the properties of an action](/openaev/use/threat-arsenal/action-properties/#detection-remediation-properties) and [Let AI run a penetration test](/openaev/use/attack-chaining/autonomous/).
:::

## What's next

- [Set the properties of an action](/openaev/use/threat-arsenal/action-properties/): every field of the form.
- [Turn an action's output into findings](/openaev/use/threat-arsenal/output-parsers/): extract structured values from an execution.
- [Security control domains](/openaev/concepts/security-domains/): how domains classify actions.
- [Injects](/openaev/use/injects/) and [Atomic testing](/openaev/use/atomic-testing/): run an action.
- [Findings](/openaev/use/findings/): the data an action's output parsers extract.
