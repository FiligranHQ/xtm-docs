---
title: Add expectation results by hand
description: Validate the manual expectations of a simulation for each team and person, and add the prevention or detection result of a security platform that no collector reports.
doc_type: how-to
products:
  - openaev
personas: [exercise-planner, validation-engineer, soc-analyst]
owner: docs-openaev
last_reviewed: 2026-11-07
migrated_from:
  - openaev:/usage/expectations/validation/
---

Some results reach OpenAEV through nobody: a crisis team escalated on time, a security tool without a collector blocked the command, the SOC found the alert in a console OpenAEV cannot read. You record them yourself, as a score. An exercise planner validates the manual expectations of the teams; a validation engineer or a SOC analyst adds the result of a security platform to a prevention or detection. How the scores add up is in [How expectations score an inject](/openaev/use/expectations/).

::diagram{id=openaev-expectations-validation-flow}

## Before you start

- Your role has **Manage assessment**, or the right to manage the simulation or atomic test.
- The inject has run, and its expectation is waiting: a manual expectation never answers itself, and a technical one waits until it expires.
- To add the result of a security platform, the platform was created by hand in :menu[Security platforms], and your role has **Access security platforms**. Platforms that a collector registered cannot be chosen: their collector answers for them.

## Validate the manual expectations of a simulation

An exercise planner follows the human side of a simulation from one screen, the **Validations** of its execution.

:::steps

1. Open :menu[Simulations], the simulation, then its **Execution** tab, and click **Validations** in the menu on the right. The **Manual validations** list groups the expectations by inject, then by team. The link of the human response gauge, in the simulation's **Overview**, opens the same list.
2. Narrow the list with the search bar, on the inject's title or description, or by tag.
3. Click the expectation under the team. Its chip says **Pending validation**, or the result already recorded with its score. A drawer opens, with the validation mode of the expectation.
4. Under **Team**, set the **Score**, or choose **Success** or **Failed** in **Status**. The expected score shows under the field. The score of the team is copied to every person of the team.
5. To score a person differently, set their own **Score** under **Persons**. The team's result is then computed from its persons, with the validation mode.
6. Click **Validate**.

:::

The chip of the expectation shows its new result, and the human response gauge of the simulation moves. You can validate again later: a new score replaces the previous one.

::screenshot{id=openaev-expectations-validations alt="The Validations menu of the simulation Velvet Harbor crisis night: the expectation On the bridge within 30 minutes of the Crisis cell team, for all players, waiting for an evaluator."}

## Add a result from the result of an inject

The same works from the result of any inject, atomic tests included, and it is the only way to add the result of a security platform.

:::steps

1. Open the result of the inject. See [Read the result of an inject](/openaev/use/injects/results/).
2. In **Targets**, click the target: a person or a team for a manual expectation, an asset or an agent for a prevention or detection. An asset group cannot be answered directly: it is computed from its assets.
3. On the expectation's card, click **Add a result**: the shield icon for prevention, detection and vulnerability, the clipboard icon for manual, article and challenge. On a manual expectation that already has a score, open its menu and click **Update** instead.
4. For a manual expectation, set the **Score** or the **Status**, as in the **Validations**.
5. For a prevention, detection or vulnerability, choose the **Security platform** that saw the action, then its **Score**. Only the platforms created by hand, of the expected kinds, that have not answered yet are offered. On an asset, the result is written on every agent of the endpoint, as the dialog says.
6. Click **Validate**.

:::

The result shows as a line of the card, under the security platform you chose. For prevention and detection, a success settles the expectation at once, and no later failure lowers it. To correct a result, open the menu of its line and click **Update** or **Delete**.

:::example[Score the front desk and the SOC after the Velvet Harbor drill]
On 14 January 2026, Example CERT runs the **Velvet Harbor replay** simulation for Grand Meridian Hotels. Two answers never reach OpenAEV on their own.

The manual expectation `Reported to the SOC` of the email **Fake summit booking request** targets the team **Front desk France**, with **All persons (per team) must validate the expectation** and a success score of 100. Paul Garnier, Camille Roux and Julien Bernard all called the SOC within 20 minutes. In **Validations**, the exercise planner opens it under **Front desk France** and sets the team to **Success**: every person of the team scores 100.

The `PowerShell loader` detection on `fd-barcelona-03` stayed **Pending**: the Barcelona hotel still runs the EDR of a former provider, which raised the alert but has no collector. The SOC analyst opens the inject's result, the asset **fd-barcelona-03**, clicks **Add a result** on **Detection**, chooses **Barcelona legacy EDR**, a platform of the EDR kind created by hand, enters a **Score** of 100 and clicks **Validate**. The detection of that asset reads **Detected**, and the group's detection succeeds.
:::

## What's next

- [How expectations score an inject](/openaev/use/expectations/).
- [Set the expectations of an inject](/openaev/use/expectations/management/).
- [Collectors](/openaev/use/collectors/): let a security platform answer on its own.
