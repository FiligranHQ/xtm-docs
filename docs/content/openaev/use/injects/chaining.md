---
title: Run injects on conditions and move them between scenarios
description: Make an inject run only when its parent succeeded or failed, to model a multi-stage attack, and export injects to import them into another scenario, simulation or platform.
doc_type: how-to
products:
  - openaev
personas: [validation-engineer]
owner: docs-openaev
last_reviewed: 2026-11-07
migrated_from:
  - openaev:/usage/inject-chaining/
---

A real attack branches: the attacker moves on only when the first stage worked. Logical chains reproduce that in a simulation, by making an inject run only when the execution or an expectation of its parent ended the way you set. The second half of this page exports injects, with their content and expectations, to import them into another scenario, simulation, atomic test or OpenAEV instance. For a full graph of actions with shared outputs and execution boundaries, use [attack chaining](/openaev/use/attack-chaining/), an Enterprise Edition feature.

::diagram{id=openaev-injects-chaining-conditions}

## Before you start

- Your role has **Manage assessment**, or the right to manage the scenario or simulation.
- The injects to chain exist in the same scenario or simulation, and the child is scheduled after its parent, late enough for the parent's expectations to be settled. See [Add injects to scenarios and simulations](/openaev/use/injects/).

## How the conditions are checked

When the time of a child inject comes in a simulation, OpenAEV reads its parent:

- **Execution** is **Success** when the parent has finished with any status but **Error**: **Executed**, and **Partial** too. It is **Fail** when the parent ended in **Error**, or has not finished yet.
- An expectation, such as **Prevention** or **Detection**, or a manual expectation by its name, is **Success** when it succeeded. Anything else is **Fail**: failed, partial, or still pending.

When the conditions hold, the child runs. When they do not, it does not run: it ends in **Error**, with the trace `This inject depends on other injects expectations that are not met`, followed by each condition that failed. Its own children then see a parent in **Error**, and stop too.

Conditions apply in simulations only. An atomic test ignores them.

## Chain two injects

:::steps

1. In the **Injects** tab, open the menu of the child inject and click **Update**, then open the **Logical chains** tab.
2. Under **Parent**, click **Add parent** and choose the parent in **Inject**. An inject has at most one parent.
3. Next to **Execution condition:**, click the condition to change it. Choose what to test: **Execution**, or one of the parent's expectations. Then choose **Success** or **Fail**.
4. To test more than one thing, click **Add condition**. Between two conditions, click **AND** to switch to **OR**: with **AND** every condition must hold, with **OR** one is enough. The choice applies to every condition of that link.
5. To make other injects depend on this one, click **Add child** under **Children** and set their conditions the same way.
6. Click **Update**.

:::

::screenshot{id=openaev-injects-chaining-form alt="The Logical chains tab of the inject Local guest records: its parent PowerShell loader with the execution condition Prevention is Fail and the Add condition button, then its child HTTPS beacon under Children."}

In **Interactive view**, the timeline of the **Injects** tab, you can also drag a link from the dot at the side of one inject to another. The new link's condition is **Execution** is **Success**; change it in the **Logical chains** tab. A link goes only from an inject to a later one. To move a link, drag its end onto another inject: the moved link starts again from **Execution** is **Success**. To remove it, drop its end on an empty part of the timeline.

::screenshot{id=openaev-injects-chaining-timeline alt="The interactive view of the Velvet Harbor replay scenario: Fake summit booking request at 0 minutes, then PowerShell loader at 15 minutes linked to Local guest records at 30 minutes by Prevention is Failure, itself linked to HTTPS beacon at 45 minutes by Execution is Success."}

:::example[Run the search only when the loader got through]
The **Velvet Harbor replay** scenario of Grand Meridian Hotels replays the 9 January 2026 intrusion on the asset group **Front desk workstations**. Its inject `PowerShell loader`, at 15 minutes, runs the PowerShell (T1059.001) action and expects Meridian EDR to prevent it. The validation engineer wants the search for guest records (T1005) to run only when Meridian EDR let the loader through, and the HTTPS beacon only when the search ran:

| Child | Parent | Execution condition |
| --- | --- | --- |
| `Local guest records`, at 30 minutes | `PowerShell loader` | **Prevention** is **Fail** |
| `HTTPS beacon`, at 45 minutes | `Local guest records` | **Execution** is **Success** |

If Meridian EDR blocks the loader, the search ends in **Error** with the unmet condition, and the beacon stops with it: the simulation shows that the attack ended at the loader. Keep in mind that a prevention still pending at 30 minutes also counts as **Fail**: if the EDR collector polls rarely, schedule the search later, or the search runs before the block is known.
:::

## Reuse injects elsewhere

An export is a ZIP archive that holds the injects, their content, expectations, tags and chains, and the files attached to them. Assets and asset groups are never exported: set them again after an import.

### Export

:::steps

1. Open the list of injects: the **Injects** tab of a scenario or simulation, or :menu[Atomic testings].
2. For one inject, open the menu at the end of its line and click **Exporter (JSON)**. For several, in a scenario or simulation, tick them and click **Export** in the toolbar.
3. In the dialog, tick what to include besides the injects: **Teams**, **Persons**, **Variable values**. All three are off by default.
4. Click **Export**. The browser downloads the archive.

:::

Exporting requires read access to the scenario or simulation. An atomic test exports one inject at a time.

### Import

:::steps

1. Open the destination: the **Injects** tab of a scenario or simulation, or :menu[Atomic testings].
2. In a scenario or simulation, click **Import injects**, then **Import JSON**. In :menu[Atomic testings], click **Import JSON**.
3. Choose the archive and click **Import**. The archive must be a ZIP file.

:::

The injects appear with their original configuration. Importing requires the right to manage the destination; into :menu[Atomic testings], **Manage assessment**. Any source goes into any destination: a simulation's injects into an atomic test, an atomic test into a scenario. **XLS import (mapper)**, in the same menu, creates injects from a spreadsheet instead.

## What's next

- [Read the result of an inject](/openaev/use/injects/results/): see which conditions held.
- [Inject execution statuses](/openaev/use/injects/status/).
- [Attack chaining](/openaev/use/attack-chaining/): actions that share their outputs.
