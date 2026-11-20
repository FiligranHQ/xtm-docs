---
title: Choose the actions of a chained scenario
description: Pick the Threat Arsenal actions of a chained scenario, configure them, and feed their arguments from variables or from what earlier steps found.
doc_type: how-to
products:
  - openaev
personas: [validation-engineer]
owner: docs-openaev
last_reviewed: 2026-11-07
edition: ee
migrated_from:
  - openaev:/usage/attack-chaining/action-selection/
---

Each step of a chained scenario is an action of the Threat Arsenal, configured for this chain: a command on an endpoint, a scan, an email. What makes it a chained step is where its arguments come from. Instead of a typed value, an argument can read the values that earlier steps found, such as the host a scan reported or the account a credential search returned, so the step runs on what the attack really discovered. This page adds actions to the **Logic** tab and links their arguments. How events decide when they run is in [Build the logic of a chained scenario](/openaev/use/attack-chaining/logic/).

## Before you start

- The Enterprise Edition is active, and the scenario is a **Chained scenario** whose scope is set. See [Set the targets and limits of a chained scenario](/openaev/use/attack-chaining/scope/).
- Your role has **Manage assessment**.
- The actions you need are in the Threat Arsenal, with output parsers when later steps read their results. See [Turn an action's output into findings](/openaev/use/threat-arsenal/output-parsers/).

## Add an action

:::steps
1. Open the scenario and its **Logic** tab, then click **Add component**.
2. Choose **Action**, "Execute an action with configured parameters". The **Select an action** step lists the Threat Arsenal.
3. Narrow the list with the filters: **Action type** (the injector that runs it), **Platforms**, **Domains**, **Tags**, and **Providing**, the outputs an action produces. A **Compatible output** chip shows when the list was opened to provide a field an event needs; remove the chip to see the whole arsenal.
4. Click the action. The **Configure** step opens. To add several actions at once, check their rows and add them together; you then configure each one on the canvas.
5. Set the **Title** of the step, as it shows on the canvas.
6. Under **Inject data**, set the arguments. They start from the action's default values, which apply unless you change them; **Reset default value** brings them back.
7. Link the arguments that should read earlier results instead of a typed value: see [Feed an argument from earlier steps](#feed-an-argument-from-earlier-steps).
8. Under **Inject expectations**, keep or adjust the expectations of the action, such as **Prevention** and **Detection**.
9. Click **Save**. The action appears on the canvas as a card with its **Type**, its **Tactic**, its **Targets** and the number of triggers it **Waits on**.
:::

::screenshot{id=openaev-attack-chaining-action-drawer alt="The Select an action step of the Add component drawer, between Choose type and Configure: the Threat Arsenal with its search and filters, the Velvet Harbor actions among the built-in ones, and the Add actions button for a bulk selection."}

The targets come from the scope: an action that runs on endpoints lists the assets of the allow list under **Initial Source Assets**, and an email lists the allowed teams under **Targeted teams**. Steps further down the chain can reach other targets of the allow list as the run discovers them.

## Feed an argument from earlier steps

::diagram{id=openaev-attack-chaining-actions-values}

Every value in a chained run has a type: **Port**, **Host**, **Username**, **CVE**, **Ip subnet** and the other output types of the Threat Arsenal. Linking an argument binds it to one or more of those types, not to a given action: at run time, the argument takes the values of those types that the run holds.

:::steps
1. In the **Configure** step, next to the argument, click **Link an Output**.
2. Choose one or more types. The argument shows them with a link icon; **Edit links** changes them and **Unlink** brings back the typed value.
3. Choose where the values come from with **Limit to Local Scope**:
   - Off (the default): every value of these types the run holds, from the scope variables and from every action that ran, in any branch.
   - On: only the values that the event right before this action kept, the ones that satisfied its conditions.
4. Click **Save**.
:::

When an argument has both a typed value and links, the typed value is used as well as the linked ones. The action runs once for each combination of values, and never twice with the same values on the same target.

Local scope matters when several branches produce the same type. An event that holds when **Port** equals `445` keeps `445` on its branch: an action after it, with **Limit to Local Scope** on, reads `445` only, while the same action with the switch off also reads the ports every other scan found, and runs once for each. For an event to keep a value, its condition needs a value to compare: **Equals**, **Not equals**, **Contains** or **Not contains**. **Is null** and **Is not null** only check that a value exists, and keep none.

## Check the action

- The card of the action shows on the canvas, with **Not scoped yet** under **Targets** when the allow list has no target it can use.
- Hover a field in the condition builder of an event: its tooltip lists **Actions on the logic flow which produce this input:**, your new action among them when it provides that field.

:::example[Query the booking database on the host the scan found]
The chain of the night test of Grand Meridian starts on the front-desk workstation `fd-paris-01`. Its third step reaches the booking database server with the shared night auditor account, on the host that the first step found listening on port 1433.

| Field | Value |
| --- | --- |
| Action | **Query the booking database**, a Command Line action of the Threat Arsenal: `sqlcmd -S #{db_host} -E -Q "SELECT name FROM sys.databases"` |
| **Title** | `Query the booking database from the front desk` |
| Argument `db_host` | **Link an Output**: **Host**, with **Limit to Local Scope** on |
| **Inject expectations** | **Prevention** and **Detection**, for Meridian EDR |

With local scope on, the query runs only on `10.20.8.15`, the host the event **Booking database reachable** kept, and not on the other hosts the scan saw in `10.20.8.0/24`.
:::

:::xtm-one[Let the AI choose the actions]
With OpenAEV connected to XTM One, the **AI builder** of a chained scenario picks the actions from your Threat Arsenal and links their arguments for you, from an objective in plain words. When a technique needs an action you do not have, it says which integration would provide it. See [Let AI run a penetration test](/openaev/use/attack-chaining/autonomous/).
:::

## What's next

- [Build the logic of a chained scenario](/openaev/use/attack-chaining/logic/): gate each action with an event.
- [Set the properties of an action](/openaev/use/threat-arsenal/action-properties/): the arguments an action takes.
- [Read how far a chained simulation got](/openaev/use/attack-chaining/attack-path-map/)
