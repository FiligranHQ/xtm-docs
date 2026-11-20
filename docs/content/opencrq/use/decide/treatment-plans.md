---
title: Choose the treatment plan that removes the most loss
description: Build a treatment plan of control moves in OpenCRQ, enter what it costs, and read its annual loss reduction and one-year return per scenario.
sidebar_position: 2
doc_type: how-to
products:
  - opencrq
personas: [risk-manager, executive]
owner: docs-opencrq
last_reviewed: 2026-11-04
---

A treatment plan is the set of controls you would fund for one risk scope, each aimed at a better posture. OpenCRQ prices the scope's risk scenarios again as if the plan were done, so you read the annual loss the plan removes against what it costs, and rank plans by their return before you ask for the money. If you are new to how OpenCRQ prices a scenario, read [How OpenCRQ quantifies risk](/opencrq/concepts/how-risk-is-quantified/) first.

## Before you start

- Any member of the tenant can create, edit and delete treatment plans: no capability is needed.
- The risk scope needs risk scenarios that OpenCRQ has priced at least once: a plan is read against the scope's scenarios, and a scenario with no simulation has no figure to compare. See [Build and price a risk scenario](/opencrq/use/quantify/risk-scenarios/).
- The scope's controls are declared, with their current strength and coverage. See [Set how strong your controls are and read how they held](/opencrq/use/defenses/controls/).
- OpenCRQ shows every amount in dollars today; the choice of currency is coming.

## How a plan is priced

A plan moves controls of its scope from their current posture to a target, on one or both of two dimensions:

| Dimension | Levels, from weakest to strongest |
| --- | --- |
| **Strength** | **Not deployed**, **Basic**, **Intermediate**, **Master** |
| **Coverage** | **None**, **Partial**, **Broad**, **Full** |

When you save the plan's moves, OpenCRQ simulates each ready scenario of the scope twice on the same simulated years: at the posture the scope holds today, and with the plan's targets in place. The difference is the scenario's **ALE reduction**, the Annualized Loss Expectancy (ALE) the plan removes. The plan's **Annual ALE reduction** is the sum over the scope's scenarios.

What the plan costs comes from two fields you enter: a **One-off cost** and an **Annual recurring cost**. Their sum is the **First-year cost**. The **One-year return** compares the two:

> One-year return = (Annual ALE reduction − First-year cost) / First-year cost

A plan that removes less loss in a year than it costs in its first year has a negative return. A plan with no cost entered has no return.

Two plans that move the same control both claim its reduction. That is why the list never adds plans up: compare them side by side, and build a single plan for the controls you would fund together. Controls that cover the same attack techniques remove less together than the sum of what each removes alone, and the plan's own figure already accounts for that.

## Create a plan

You can start a plan from three places:

- The list: :menu[Treatment Plans] in the sidebar lists the plans of every scope, with a **Risk scope** filter. The **Treatment Plans** entry under a risk scope lists that scope's plans.
- The **Risk treatment recommendations** group of the [CISO dashboard](/opencrq/use/decide/dashboards/), which lists the control moves worth the most and has a **Create a risk treatment plan** button.
- A risk scenario's controls: select the controls in its **How to strengthen your defence** table, then click **Add to a treatment plan**. The drawer adds the moves the scenario priced to an existing plan, or to a **New plan**.

From the list:

:::steps
1. Open :menu[Treatment Plans], then click **Create**. The **Create a risk treatment plan** drawer opens.
2. From the list of every scope, choose the **Risk scope**. From a scope's own list, the plan belongs to that scope. The scope cannot change afterwards: the plan's figures are priced for it.
3. Enter the **Name** of the plan. It is the only other required field.
4. Choose the **Owner**, the person accountable for delivering it, and the **Start date** and **Due date**.
5. Write the **Rationale**: what the plan sets out to obtain, and why now. It is what a board reads first on the plan's page.
6. Click **Create**. The plan's page opens, as a **Draft**.
:::

## Choose the controls the plan moves

:::steps
1. On the plan's page, click **Choose the controls** in the **Moves of the plan** card. The edit drawer opens on its **Controls** tab.
2. Read the **Gain** column first. It rates each move out of five dollar signs, against the best one-step move the scope offers. The table is sorted by it, so the moves worth the most come first.

   ::screenshot{id=opencrq-treatment-plans-controls alt="The Controls tab of the plan's edit drawer, with one row per control and dimension, the Gain rated in dollar signs, the Current posture and a Target menu set to Unchanged."}

3. For each control to fund, open the **Target** menu of a dimension and choose the level to reach. Leave a dimension on **Unchanged** to keep it as it is. Choosing a target on any dimension puts the control in the plan; setting all its dimensions back to **Unchanged** takes it out.
4. Switch to the **Identity** tab and enter the **One-off cost** and the **Annual recurring cost** of the plan.
5. Click **Save**. The controls and the identity are saved together, and OpenCRQ starts simulating the scope's scenarios against the plan.
:::

A few things to know when you choose targets:

- The rating is what one move is worth on its own, read on each scenario's latest simulation. It ranks the moves; it does not add up, because two controls covering the same technique remove less together.
- A control that is **Not deployed** is priced on its first step, which raises its strength and its coverage at once: both of its rows show the same rating.
- A warning sign next to a target means the move buys nothing yet: a control needs both a strength and a coverage to protect anything. Its tooltip says which one is missing.

## Read what the plan buys

::screenshot{id=opencrq-treatment-plan alt="The page of the treatment plan MFA for the front-desk accounts, with its Identity card, the Cost and return card at a -100% return, $0 of annual ALE reduction and $115K of first-year cost, the Moves of the plan card where the Identity Platform already holds its Intermediate strength and Full coverage targets, and the Per scenario table."}

The plan's page stacks four cards:

- **Identity**: the **Scope**, **Owner**, **Start date**, **Due date** and **Rationale**.
- **Cost and return**: the **One-year return**, green when positive and red when negative, the **Annual ALE reduction**, and the **First-year cost**. **Enter the cost** or **Edit the cost** opens the edit drawer.
- **Moves of the plan**: one row per control and dimension the plan moves, with its **Current** level and its **Target**. A row opens the control.
- **Per scenario**: for each scenario of the scope, its **ALE** today and with the plan (an arrow between the two), the **ALE reduction** with the share of the scenario's loss it removes, and the date of the **Last simulation**. Hover the reduction to read its spread: the p10, the mean and the p90 of the simulated years. A row opens the scenario.

The figures stay on the page while they age, with a chip that says why:

| Chip or text | Meaning | What to do |
| --- | --- | --- |
| **Expired** | The figure was priced against an earlier set of moves | Click **Re-simulate scenarios** |
| **Older than the ALE** | The scenario was simulated again since, on new threats or a new posture, and the plan was not | Click **Re-simulate scenarios** |
| **Not computed** | No simulation carries a figure for this plan. Under the name, **Not simulated since the plan last changed.** says the scenario is ready to be simulated | Click **Re-simulate scenarios**, or complete the scenario if it is not ready |

**Re-simulate scenarios**, at the top of the **Per scenario** card, shows only when a scenario needs it, and turns into a spinner while the simulations run.

When the scope's posture reaches a plan's targets, for example once the controls are deployed and declared, the plan has nothing left to remove: its **Annual ALE reduction** falls to $0 and its return to −100%. The loss it removed shows in the scope's own figures and in the [dashboards' trends](/opencrq/use/decide/dashboards/).

## Compare the plans

The list shows the plans as cards, sorted by **Return**, highest first; **Lines view** shows them as a table with the same order. Each card gives the plan's **Return**, **First-year cost**, **ALE reduction**, due date, and a bar of the time elapsed from its start date to its due date.

::screenshot{id=opencrq-treatment-plans-list alt="The three treatment plans of Grand Meridian France as cards, best return first, from Zero-trust remote access at +80.4% to MFA for the front-desk accounts at -100%, under four counters: Active plans, Overdue, Negative return at 2 out of 3, and First-year cost."}

Four counters sit above the list:

- **Active plans**, out of every plan listed.
- **Overdue**: the active plans whose due date has passed. A plan past its due date shows an **Overdue** chip, on the list and on its page.
- **Negative return**: the plans whose first-year cost exceeds the annual loss they remove.
- **First-year cost**: the one-off cost plus a year of recurring cost, summed over the active plans, with the total of every plan below it.

No counter adds up the ALE reductions, for the reason given in [How a plan is priced](#how-a-plan-is-priced).

A plan is created as **Draft**. The list also knows the **Active**, **Completed** and **Abandoned** statuses, which the counters read; this development version has no control yet to move a plan out of **Draft**.

To delete a plan, open the menu of its card or row and click **Delete**. The plan and the control targets it holds are deleted for good; the scenarios and the controls are not touched.

:::example[Price MFA for the front-desk accounts at Grand Meridian France]
On 9 January 2026, a front-desk account without a second factor exported guest records from Grand Meridian's booking platform. On 26 January, the risk manager prices the fix for the risk scope Grand Meridian France.

In :menu[Treatment Plans], **Create**:

- **Risk scope**: Grand Meridian France
- **Name**: MFA for the front-desk accounts
- **Start date**: Feb 1, 2026; **Due date**: Jun 30, 2026
- **Rationale**: The incident of 2026-01-09 went through a front-desk account without a second factor.

In the edit drawer, **Controls** tab, row **IDP Identity Platform (IdP, SSO, MFA, Conditional Access)**: **Strength** from **Basic** to **Intermediate**, **Coverage** from **Partial** to **Full**. **Identity** tab: **One-off cost** 85,000, **Annual recurring cost** 30,000. **Save**.

The **First-year cost** reads $115K, and the **Per scenario** table prices Data Breach on Guest Records and Ransomware on the Booking Platform with and without the plan. On 2 March, the scope's Identity Platform is declared at those targets, MFA on every account: Data Breach on Guest Records falls from $97K to $33K a year that day. Since then the posture holds the plan's targets, so on 12 April the plan reads an **Annual ALE reduction** of $0.

On 10 April, after Ember Tide's return, the risk manager prices two alternatives the same way. On 12 April the list compares the three plans, best return first:

| Plan | Moves | First-year cost | ALE reduction | Return |
| --- | --- | --- | --- | --- |
| Zero-trust remote access for the hotels | Secure Remote Access (VPN / ZTNA) to **Basic** strength and **Partial** coverage | $35K | $63K | +80.4% |
| Network intrusion prevention at the head office | Network Intrusion Prevention to **Basic** strength and **Partial** coverage | $195K | $52K | −73.3% |
| MFA for the front-desk accounts | Done | $115K | $0 | −100% |

**Negative return** reads 2 out of 3 plans with a return. The remote access plan is the one to fund.
:::

## What's next

- [Review a year's security budget against the loss it avoided](/opencrq/use/decide/budget-reviews/), once the plan's work is paid for.
- [Track cyber risk in money with dashboards](/opencrq/use/decide/dashboards/), to show the board the loss the plans removed.
- [How OpenCRQ quantifies risk](/opencrq/concepts/how-risk-is-quantified/), for the model behind every figure.
