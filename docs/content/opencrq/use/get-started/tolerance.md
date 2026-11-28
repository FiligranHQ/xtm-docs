---
title: Set the yearly loss your company accepts
description: Set the risk tolerance of your first risk scope in Get started, an annual loss and the highest probability you accept of exceeding it, so each scenario reads within or outside it.
sidebar_position: 7
doc_type: how-to
products: [opencrq]
personas: [risk-manager, executive]
owner: docs-opencrq
last_reviewed: 2026-11-20
---

**Tolerance** is the sixth step of :menu[Get started]. It sets the risk tolerance of the **Enterprise** risk scope: an annual loss, and the highest probability you accept that a year's loss exceeds it. Each scenario of the scope then reads **Within tolerance** or **Outside tolerance**, so you can show the board which risks lie beyond what the company accepts.

## How a scenario is placed within or outside it

Each scenario's latest simulation draws its loss exceedance curve: the probability that a year's loss exceeds each amount. OpenCRQ reads that curve at your annual loss:

- When the probability of exceeding it is above your maximum, the scenario reads **Outside tolerance**.
- When it is equal to your maximum or below, the scenario reads **Within tolerance**.

The tolerance applies to each scenario on its own, never to the sum of the scope's scenarios. Two scenarios can each be within tolerance while their losses added together exceed it.

A scenario can also set its own threshold, with **Edit** on the **Loss exceedance curve** card of its **Overview**. The card then reads **Scenario's own threshold** instead of **Risk scope threshold**, and the scope's tolerance no longer applies to that scenario.

## Before you start

- You are a member of the tenant. Setting the tolerance needs no capability.
- You know the two values the company accepts, as agreed with whoever owns the risk appetite, such as the chief financial officer or the board.

## Set the tolerance

:::steps
1. Open :menu[Get started] and select the **Tolerance** row, or select **Next** on the **Finances** step. The step opens on the **Risk tolerance** card, with **Why this step?**: "Your tolerance sets the loss you accept. Each scenario will be placed within or outside it."
2. In **Annual loss (USD)**, enter the yearly loss of one scenario you would accept. It must be above zero.
3. In **Maximum probability of exceeding it (%)**, enter the highest chance you accept that a year costs more, between 0.1 and 100.

   ::screenshot{id=opencrq-get-started-tolerance alt="The Tolerance step of Get started for Grand Meridian Hotels: the stepper at Tolerance, the Risk tolerance card with its Why this step? card, Annual loss (USD) at 2000000 and Maximum probability of exceeding it (%) at 10."}

4. Leave the field. The tolerance saves at once, with no message, once both values are valid.
5. Select **Next** to go on to the **Protect** step.
:::

The step has no save button, and "Both values are needed to save the tolerance." A field you leave while the other is empty saves nothing. Once you have left both fields, a missing or wrong value says why: **Enter an annual loss.**, **The annual loss must be above zero.**, **Enter a maximum probability.** or **The maximum probability must be between 0.1% and 100%.**

If you leave the step with one value only, the **Incomplete tolerance** dialog asks "The tolerance needs both values. Leave without saving it?". **Cancel** keeps you on the step and shows what is missing; **Leave without saving** leaves it.

A tolerance can be changed but not removed: emptying both fields saves nothing, and the step shows the stored values again when you come back. To change it later, use the step again, or **Edit** on the **Risk tolerance** card of the scope's **Details**: **See the risk scope** opens them.

:::example[Grand Meridian accepts one bad year in ten]
Grand Meridian Hotels' board accepts that a single cyber risk may cost the chain more than $2 million in a year, but no more often than one year in ten. Claire Morel, its risk manager, opens the **Tolerance** step:

- **Annual loss (USD)**: 2000000
- **Maximum probability of exceeding it (%)**: 10

She leaves the second field, and the stepper checks **Tolerance**. Once the **Scenarios** step prices the chain's scenarios, one with a 4% chance of costing more than $2 million in a year reads **Within tolerance**. One with a 15% chance reads **Outside tolerance**: it is the first one to treat.
:::

## Check that it works

- The stepper shows a check on **Tolerance**, and the overview of Get started reads **Already configured** on its row.
- The **Risk tolerance** card of the **Enterprise** scope's **Details** shows the **Annual loss** and the **Maximum probability of exceeding it**. Once its scenarios are simulated, it counts those outside and within tolerance, with **See the gap to tolerance**.

## What's next

- [Get your first quantified risk scenario in eight steps](/opencrq/use/get-started/), to see which steps are left.
- [Build and price a risk scenario](/opencrq/use/quantify/risk-scenarios/), to read a scenario's loss exceedance curve against the tolerance.
- [Track cyber risk in money with dashboards](/opencrq/use/decide/dashboards/), where **Gap to tolerance** shows how far each scenario sits from it.
- [Review a year's security budget against the loss it avoided](/opencrq/use/decide/budget-reviews/), to tell whether the scope stayed within its tolerance.
