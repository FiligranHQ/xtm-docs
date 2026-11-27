---
title: Build the logic of a chained scenario
description: Gate each action of a chained scenario with an event whose conditions read what earlier actions found, and check on the canvas that every condition can be met.
doc_type: how-to
products:
  - openaev
personas: [validation-engineer]
owner: docs-openaev
last_reviewed: 2026-11-07
edition: ee
migrated_from:
  - openaev:/usage/attack-chaining/logic-creation/
---

The logic of a chained scenario is the graph on its **Logic** tab: actions, and the events that decide when each action runs. An event is a set of conditions on what the earlier actions found, such as "a port equals 1433" or "an account name contains `night.audit`". When its conditions hold, it triggers the actions linked to it. With events, a step runs only when the attack really got there, the way an intruder decides the next move from the last one. This page builds events, links them to actions and reads the canvas. Adding the actions themselves is in [Choose the actions of a chained scenario](/openaev/use/attack-chaining/actions/).

## Before you start

- The Enterprise Edition is active, and the scenario is a **Chained scenario**.
- Your role has **Manage assessment**.
- The canvas holds the actions you want to chain. The actions whose results an event reads produce outputs: their output parsers extract typed values. See [Turn an action's output into findings](/openaev/use/threat-arsenal/output-parsers/).

## How actions and events link

::diagram{id=openaev-attack-chaining-logic-example}

- A link always goes from an event to an action: the event gates the action. Nothing links an action to an event; an event reads the outputs of every action that ran.
- An action that no event gates starts at the launch, as soon as the values its linked arguments need exist.
- An action can wait on several events: its card shows how many under **Waits on**.
- The fields an event can check come from the output types of the Threat Arsenal, not from the canvas: you can write a condition on a field that no action produces yet. The canvas then warns you, since such an event can never hold.

::screenshot{id=openaev-attack-chaining-logic alt="The Logic tab of the chained scenario Velvet Harbor path - front desk to booking database, before its first component: No components yet, and the Add component button that starts the canvas."}

## Add an event

:::steps
1. On the **Logic** tab, click **Add component**, then choose **Event**, "Define conditions to trigger the next actions".
2. Set its **Name** and, optionally, a **Description**.
3. Under **Trigger Conditions**, set the first condition:
   - **Field to Check**: the output to inspect, such as **Port**, **Username** or **CVE**. Hover a field to see the **Actions on the logic flow which produce this input:**. **Action output** is the raw output of the actions, for a keyword that no output parser extracts.
   - **Operator**: **Equals**, **Not equals**, **Contains**, **Not contains**, **Is null** or **Is not null**, and on numeric fields **Greater than**, **Greater than or equals**, **Less than** and **Less than or equals**. **Contains** takes a list of values separated by commas and holds when one of them is found.
   - **Expected Value**: the value to compare, except for **Is null** and **Is not null**. It must fit the field, such as a number for a port, or the form shows an error. The **Aa** toggle makes a text comparison **Case-sensitive**.
4. Click **Add Condition** to add another condition to the group, and set the group's operator: **And** holds when every condition holds, **Or** when one does.
5. To combine groups, click **Add Condition Group** and choose **And** or **Or** between the groups: an **Or** between two **And** groups reads "(A and B) or (C and D)".
6. Click **Add event**. The event appears on the canvas.
:::

## Gate an action with the event

:::steps
1. On the event's card, drag its right handle onto the action it should trigger. Or click its **+**, "Add an action gated by this trigger", to pick a new action that the event gates from the start.
2. Repeat for each action the event triggers. A solid link joins the event to each action.
:::

To change an event, open it from its card: the form shows **Update event**. Deleting an event or an action removes its links.

## Check that every condition can be met

When an event checks a field that no action on the canvas produces, a **Warning** banner names it: Event "Booking database reachable" **references field:** Port **which is** **not provisioned by any action.**

:::steps
1. In the banner, click **Add Compatible Action**. The action list opens with a **Compatible output** chip, showing only the actions that provide the missing field.
2. Select one or more actions and add them. They land on the canvas without any link, and the warning clears.
3. Configure the new action, and gate it with the event that should come before it, if any.
:::

The banner only checks that some action provides the field. Whether that action runs before the event is the graph you draw.

## Read the canvas

| On the canvas | What it means |
| --- | --- |
| Solid link | A saved link: the event triggers the action. The only links the run follows |
| Dashed link | Drawn when you select an event: from every action that produces a field its conditions read. A hint, never saved |
| Dimmed cards | Outside the path of the selected event: its providers, the event and the actions it triggers stay lit |
| **Waits on** | The number of triggers that gate an action |
| **Not scoped yet** | The action has no target the allow list can give it |

::screenshot{id=openaev-attack-chaining-logic-built alt="The Logic canvas of the chained scenario Velvet Harbor path - booking web front to database: the action Discover the booking web front under Credential Access, with 1 target and 8 outputs, the event Booking database reachable, Port equals 5432, and its solid link to the action Export the booking database, with 1 target and 1 trigger."}

The canvas groups the cards by MITRE ATT&CK tactic, or in chain order with **Switch to chain layout**. **Auto-organize** rearranges cards you moved, and **Fit to view** brings the whole graph back on screen.

Once a simulation is launched, its logic is read-only: **This simulation has been launched. Its logic map is read-only. Reset the simulation to edit it, or update the scenario and run it again.**

:::example[From the front desk to the booking database]
The night test of Grand Meridian asks how far an attacker on the front-desk workstation `fd-paris-01` gets toward the booking database server. Its logic has three actions, all user-created Command Line actions of the Threat Arsenal, and two events:

| Component | Field | Value |
| --- | --- | --- |
| Action | **Title** | `List the connections of the front desk`, runs `netstat -ano -p TCP` on `fd-paris-01`; its output parser extracts **Ports scan** values |
| Event | **Name** | `Booking database reachable` |
| | Condition | **Port** **Equals** `1433` |
| | Gates | `Look for saved booking credentials` |
| Action | **Title** | `Look for saved booking credentials`, runs `cmdkey /list`; its output parser extracts **Username** values |
| Event | **Name** | `Front-desk account saved` |
| | Conditions | **Username** **Contains** `night.audit` **And** **Host** **Is not null** |
| | Gates | `Query the booking database from the front desk` |
| Action | **Title** | `Query the booking database from the front desk`, its `db_host` argument linked to **Host** with **Limit to Local Scope** on |

If Meridian EDR blocks the credential search, the second event never holds and the query never runs: the attack path stops at the front desk.
:::

:::xtm-one[Let the AI write the logic]
With OpenAEV connected to XTM One, the **AI builder** button of a chained scenario writes the actions and events for you from an objective, such as "from the front-desk workstation, reach the booking database server". Nothing runs while it builds: you review the canvas, change what you need, then launch. See [Let AI run a penetration test](/openaev/use/attack-chaining/autonomous/).
:::

## What's next

- [Read how far a chained simulation got](/openaev/use/attack-chaining/attack-path-map/)
- [Set the targets and limits of a chained scenario](/openaev/use/attack-chaining/scope/)
- [Choose the actions of a chained scenario](/openaev/use/attack-chaining/actions/)
