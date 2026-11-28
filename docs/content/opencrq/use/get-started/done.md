---
title: Reach the end of Get started and open your first risk scenarios
description: Reach the end of Get started once no step is left to do, read how many risk scenarios OpenCRQ computed, and choose where to go next.
sidebar_position: 10
doc_type: how-to
no_screenshot: "The last screen opens only once no step of the Enterprise risk scope is left to do, which the demo data of the captures does not reach: the diagram shows how Get started ends."
products: [opencrq]
personas: [risk-manager, admin]
owner: docs-opencrq
last_reviewed: 2026-11-20
---

Once no step of :menu[Get started] is left to do, its last screen tells you that your platform is set up, counts the risk scenarios OpenCRQ has computed for the **Enterprise** risk scope, and leads you to what you can do with them.

::diagram{id=opencrq-get-started-done}

## Before you start

- You are a member of the tenant.
- Every step of Get started is done or started. A step that is started, such as **Defenses** with some controls assessed or **Finances** with some answers, is enough; a step still **To do** keeps Get started unfinished. The **Scenarios** step is done once a scenario of the scope has a computed loss.

## Finish Get started

:::steps
1. On the **Scenarios** step, select **Next**. Once Get started is finished, it opens the last screen; until then, it returns to the overview of the steps, where the rows still to do show.
2. Read the card **Your platform is set up**. It counts the scenarios of the **Enterprise** scope whose loss is computed, such as **5 risk scenarios computed**. Above it, the stepper shows every step, each one a link back to its page.
3. Select **See your risk scenarios** to open the scenarios of the scope, or choose a row under **What you can do now**.
:::

The rows of **What you can do now** open the pages of the **Enterprise** scope:

| Row | What it says | Where it leads |
| --- | --- | --- |
| **Risk scenarios** | Track your estimated annual losses | [Build and price a risk scenario](/opencrq/use/quantify/risk-scenarios/) |
| **Assets** | See which assets weigh most | [Build the attack surface from your assets](/opencrq/use/attack-surface/assets/) |
| **Treatment plans** | Compare a measure's cost with the loss it avoids | [Choose the treatment plan that removes the most loss](/opencrq/use/decide/treatment-plans/) |
| **Budget reviews** | Arbitrate your security budget | [Review a year's security budget against the loss it avoided](/opencrq/use/decide/budget-reviews/) |
| **Compliance** | Assess yourself against NIST CSF 2.0 | [Measure your compliance maturity against a framework](/opencrq/use/defenses/compliance/) |

The screen opens only once Get started is finished: opened earlier, its address sends you to the overview of the steps. Once it is finished, the tenant's home no longer opens Get started on your first visit. The steps stay open: you can come back to any of them to complete what you started, and the platform's pages edit the same scope.

:::example[Grand Meridian's first results]
Claire Morel has described Grand Meridian Hotels, declared part of its estate, assessed its endpoint controls and answered five financial questions. The **Defenses** and **Finances** steps are only started, but none is left to do. On the **Scenarios** step, the five scenarios she created for the **Enterprise** scope have been priced, and she selects **Next**.

The last screen reads **Your platform is set up** and **5 risk scenarios computed**. She selects **See your risk scenarios** to read their estimated annual losses, then comes back to **Treatment plans** to compare multi-factor authentication for the front desks with the loss it avoids.
:::

## Check that it works

- **Next** on the **Scenarios** step opens **Your platform is set up**, not the overview of the steps.
- The count of computed scenarios matches the scenarios of the **Enterprise** scope that show a loss in their list.
- Opening the tenant's home in a new browser opens **Home**, not Get started.

## What's next

- [Build and price a risk scenario](/opencrq/use/quantify/risk-scenarios/), to read and refine the scenarios Get started created.
- [Create and price your first risk scenarios](/opencrq/use/get-started/scenarios/), the step before, and [Get your first quantified risk scenario in eight steps](/opencrq/use/get-started/), the whole path.
- [Set how strong your controls are in the Defenses step](/opencrq/use/get-started/defenses/) and [Give your financial figures in the Finances step](/opencrq/use/get-started/finances/), to complete the steps you only started.
- [How OpenCRQ quantifies risk](/opencrq/concepts/how-risk-is-quantified/), to understand the figures.
