---
title: Set the expectations of an inject
description: Add, tune and remove the expectations of an inject, choose who must answer them, how long they wait and how a group validates them, and realign them when their action changes.
doc_type: how-to
products:
  - openaev
personas: [validation-engineer, exercise-planner]
owner: docs-openaev
last_reviewed: 2026-11-07
migrated_from:
  - openaev:/usage/expectations/management/
---

The expectations of an inject decide what its result measures: whether Meridian EDR must prevent the command or only detect it, whether one alert of the SOC is enough or every workstation must raise one, how long OpenAEV waits. An inject starts with the expectations its action proposes; this page changes them, adds others, and brings them back in line when the action changes in the Threat Arsenal. How the answers become a score is in [How expectations score an inject](/openaev/use/expectations/).

::diagram{id=openaev-expectations-management-drift}

## Before you start

- Your role has **Manage assessment**, or the right to manage the scenario, simulation or atomic test.
- To be answered automatically, prevention and detection need a [collector](/openaev/use/collectors/) for each kind of security platform you expect.

## Add an expectation

:::steps

1. Open the inject: in the **Injects** tab of a scenario or simulation, open the menu of its line and click **Update**. For an atomic test, open its menu and click **Update**.
2. Find **Inject expectations**. The list shows each expectation with its name, its description, its score, **Automatic** or **Manual**, and its expected security platforms.
3. Click **Add expectations**. The **Add expectation in this inject** dialog opens. The button is missing when the action allows no other expectation: each type can be added once, unless the action allows more.
4. Choose the **Type**. Only the types the action allows are offered, and the dialog fills in the action's defaults for that type.
5. Give it a **Name**, and a **Description** if the result needs explaining. A manual expectation is known by its name in [logical chains](/openaev/use/injects/chaining/).
6. Set the **Expiration time** in **Days**, **Hours** and **Minutes**: how long OpenAEV waits for answers before it fails the expectation. A vulnerability expectation has no expiration time to set.
7. Set the **Success score**, a whole number from 1 to 100. An answer at or above it is a success. It starts at 100, or at 50 for a manual expectation.
8. For prevention, detection and vulnerability, choose the **Expected security platforms**: EDR, XDR, SIEM, SOAR, NDR, EMAIL SECURITY and others. Leave it empty for **Any security platform**.
9. Choose the **Validation mode**: **All assets (per group) must validate the expectation** or **At least one asset (per group) must validate the expectation**; for persons, **All persons (per team)** or **At least one person (per team)**. A vulnerability expectation has no validation mode.
10. Click **Create**, then **Update** in the inject's form to save the inject.

:::

::screenshot{id=openaev-expectations-list alt="The Inject expectations of the PowerShell loader inject: Meridian EDR raises an alert, score 100, Automatic, expected from an EDR, and Prevention from any security platform, with the Add expectations button below them."}

To change an expectation, open the menu at the end of its line and click **Update**; to take it out, click **Remove**. Changes apply to the next run of the inject: a simulation that already ran keeps the expectations it ran with.

::screenshot{id=openaev-expectations-update alt="The Update the expectation dialog of a detection named Meridian EDR raises an alert: an expiration time of 2 hours, a success score of 100, EDR as the expected security platform, and the validation mode At least one asset (per group) must validate the expectation."}

### What each choice changes

| Choice | Effect | When it is wrong |
| --- | --- | --- |
| **Expected security platforms** | Only collectors of these kinds answer, and a manual result can be added for these platforms only | With no collector of these kinds, the expectation fails as soon as the inject runs, with **No automated security platform detected** |
| **Expiration time** | How long the expectation waits. OpenAEV extends a prevention or a detection to twice the polling period of its collectors | Too short, and the expectation fails before the SOC or the collector answers; too long, and the simulation stays open |
| **Success score** | The score an answer must reach to count as a success | Above what your collectors report, and every answer counts as a failure |
| **Validation mode** | How the agents of an asset, the assets of a group and the persons of a team add up | "All" fails a group for one missing alert; "at least one" hides a workstation that saw nothing |

The defaults of the platform, when an action sets none, come from its configuration: 6 hours for technical expectations, 24 hours for human ones, a success score of 50 for manual ones. An administrator changes them with the keys of the [`openaev.expectation` block](/reference/openaev/configuration/#openaev-expectation).

## Realign expectations with their action

An inject copies the expectations of its action when you create it, and keeps them when the action changes later. When the action of at least one inject now asks for different expectations, the header of the scenario, simulation or atomic test shows **Review expectations**. Only the type, the validation mode and the expected security platforms count as a change: a different name, score, description or expiration time does not.

:::steps

1. Click **Review expectations**. Under **Validation requirements**, the popover says how many injects drifted, such as **3 of 12 injects use expectations that no longer match the validation requirements defined by their actions**.
2. Decide whether the difference is on purpose:
   - If you tuned these expectations deliberately, click **Dismiss**. The warning shrinks to an icon, for every user, until the expectations match again.
   - Otherwise, click **Realign expectations**. OpenAEV replaces the expectations of every drifted inject with the current ones of its action. Your changes to those injects are lost.

:::

::screenshot{id=openaev-expectations-drift alt="The Validation requirements popover of the scenario: Security posture requirements have changed, 2 of 4 injects use expectations that no longer match the validation requirements defined by their actions, with the Dismiss, Cancel and Realign expectations buttons."}

To bring a dismissed warning back, click its icon, then **Restore warning**. When you relaunch an atomic test whose expectations drifted, the confirmation offers **Realign expectations to the current action before relaunching**.

:::example[Make one EDR alert enough for the front desks]
The `PowerShell loader` inject of the **Velvet Harbor replay** scenario targets the asset group **Front desk workstations** of Grand Meridian Hotels. Its action proposes a **Detection** that every asset must validate. The SOC of Grand Meridian reads one Meridian EDR alert as an incident for the whole hotel chain, so the validation engineer updates the detection:

| Field | Value |
| --- | --- |
| **Name** | `Meridian EDR raises an alert` |
| **Expiration time** | 0 days, 2 hours, 0 minutes |
| **Success score** | 100 |
| **Expected security platforms** | EDR |
| **Validation mode** | **At least one asset (per group) must validate the expectation** |

Since the validation mode and the expected security platforms changed, the scenario's header now shows **Review expectations**: 2 of its 4 injects drifted, the loader and **Fake summit booking request**, whose manual expectation **Reported to the SOC** its email action does not propose. Both changes are deliberate: the engineer clicks **Dismiss**. On the next simulation, the detection succeeds as soon as Meridian EDR's collector reports one alert, and fails after 2 hours without any.
:::

## What's next

- [How expectations score an inject](/openaev/use/expectations/).
- [Add expectation results by hand](/openaev/use/expectations/validation/).
- [Run injects on conditions and move them between scenarios](/openaev/use/injects/chaining/): branch on an expectation.
