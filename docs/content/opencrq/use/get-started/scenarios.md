---
title: Create and price your first risk scenarios
description: Pick the scenarios OpenCRQ suggests from your primary assets in the Scenarios step of Get started, create them in one go and read the annual loss it computes for each.
sidebar_position: 9
doc_type: how-to
products: [opencrq]
personas: [risk-manager]
owner: docs-opencrq
last_reviewed: 2026-11-20
---

**Scenarios** is the last step of :menu[Get started], and its card reads **Your first scenarios**. A scenario pairs an attack outcome, such as ransomware, with the primary assets it reaches. OpenCRQ simulates thousands of attacks on it to estimate your annual loss. The step suggests the scenarios your primary assets call for, creates the ones you pick in the **Enterprise** risk scope, and prices them.

## Before you start

- The **Enterprise** scope has at least one primary asset. See [Choose what you protect and link the assets behind it](/opencrq/use/get-started/protect/).
- The tenant has threats, so the scenarios have an attack frequency. See [Import the threats that target your company](/opencrq/use/get-started/threats/).
- The **Finances** step is answered, so the scenarios have a cost. See [Give your financial figures in the Finances step](/opencrq/use/get-started/finances/).

## The suggested scenarios

Under **Suggested scenarios**, the step shows one card per attack outcome that reaches at least one of your primary assets and has no scenario in the scope yet. Each card names the primary assets the outcome reaches. Which ones it reaches depends on their business resource:

| Attack outcome | Reaches the primary assets of these templates |
| --- | --- |
| **Ransomware** | **Customer data**, **HR data**, **Production and operations**, **Email and collaboration** |
| **Data theft extortion** | **Customer data**, **HR data**, **Intellectual property** |
| **Data breach and resale** | **Customer data**, **HR data**, **Intellectual property** |
| **Financial fraud and funds theft** | **Payments and treasury** |
| **Espionage** | **Customer data**, **HR data**, **Intellectual property** |
| **Service disruption** | **Customer data**, **HR data**, **Production and operations**, **Email and collaboration** |
| **Destructive attack** | **Customer data**, **HR data**, **Production and operations**, **Email and collaboration** |

An asset you added with your own business resources reaches the outcomes that pair with them. See [Attack outcomes](/reference/opencrq/attack-outcomes/).

**Ransomware**, **Data theft extortion**, **Financial fraud and funds theft** and **Service disruption** are selected at first.

Without a primary asset, the step reads "Scenarios need a primary asset. Add one in the Protect step." When none of them can be reached, it reads "No scenario to suggest: none of your primary assets can be reached by an attack outcome. Add primary assets in the Protect step."

## Create your scenarios

:::steps
1. Under **Suggested scenarios**, select a card to add it to your choice, or to remove it.

   ::screenshot{id=opencrq-get-started-scenarios alt="The Scenarios step of Get started for Grand Meridian Hotels: seven suggested scenarios, each naming the primary assets it reaches, with Ransomware, Data theft extortion, Financial fraud and funds theft, and Service disruption selected."}

2. Select **Create and compute**. It is available once a card is selected. Each card you chose reads **Creating…**.
:::

Each scenario is named after its outcome and its assets, such as `Ransomware · Customer data, Production and operations`, and covers every primary asset the outcome reaches. Its first simulation starts at once. The scenarios then show under **Your scenarios**, and leave the suggestions.

## Read the first results

Under **Your scenarios**, each row shows the scenario's **Expected annual loss** once it is priced, and **Computing…** while it is simulated. A scenario that cannot be priced shows its status, with a link to the step that completes it:

| Status | What it lacks | Step to complete |
| --- | --- | --- |
| **No mapped asset** | A supporting asset linked to its primary assets | **Assets** |
| **Incomplete** | A threat event frequency | **Threats** |
| **Incomplete** | The figures of the financial profile | **Finances** |

The other statuses, **Ready**, **Stale** and **Not simulated**, are the ones of :menu[Risk Scenarios]: see [Build and price a risk scenario](/opencrq/use/quantify/risk-scenarios/). **See all risk scenarios** opens that list.

The step is done once a scenario of the scope has an expected annual loss.

:::example[Price Grand Meridian's first scenarios]
In the **Protect** step, Claire Morel added **Customer data**, **Payments and treasury** and **Production and operations**. The **Scenarios** step suggests all seven outcomes, four of them selected:

- **Ransomware** and **Service disruption**, on **Customer data** and **Production and operations**: the booking platform down in peak season.
- **Data theft extortion**, on **Customer data**.
- **Financial fraud and funds theft**, on **Payments and treasury**.

She also selects **Data breach and resale**: the night-time export of guest records of 2026-01-09 is the incident her board asks about. She leaves **Espionage** and **Destructive attack** aside, and selects **Create and compute**.

Five scenarios show under **Your scenarios**, such as `Data breach and resale · Customer data`. Each reads **Computing…**, then its **Expected annual loss**: the board's first figures in money.
:::

## Check that it works

- **Your scenarios** lists the scenarios you created, each with its **Expected annual loss**.
- **See all risk scenarios** lists them in the **Enterprise** scope.
- The stepper shows a check on **Scenarios**.

## What's next

- Select **Next**. Once every step is done or started, it opens the [end of Get started](/opencrq/use/get-started/done/); otherwise it returns to the [overview of the steps](/opencrq/use/get-started/), where the steps left to do read **To do**.
- [Build and price a risk scenario](/opencrq/use/quantify/risk-scenarios/), to read a scenario's loss and its threats.
- [Set what an attack costs with the financial profile](/opencrq/use/quantify/loss-drivers/), to refine what each scenario costs.
