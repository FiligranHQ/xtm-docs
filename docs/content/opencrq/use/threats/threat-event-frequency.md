---
title: Set the threat event frequency
description: Read the attack frequency OpenCRQ's catalogue gives a risk scope, and replace it with your own estimate for a risk scenario when you know better.
sidebar_position: 2
doc_type: how-to
products:
  - opencrq
personas: [risk-manager]
owner: docs-opencrq
last_reviewed: 2026-11-04
---

The **Threat Event Frequency** (TEF) of a risk scenario is how many attack attempts it faces in a year, successful or not, as a range: a minimum, a most likely value and a maximum. OpenCRQ draws from that range in every simulated year, so the frequency scales the whole price. By default, a scenario takes its frequency from OpenCRQ's catalogue, positioned on its risk scope's sector, country and revenue. This page shows how to read that frequency, and how to set your own when your incident data or your threat intelligence says otherwise. [How OpenCRQ quantifies risk](/opencrq/concepts/how-risk-is-quantified/) explains where the frequency enters the price.

## Before you start

- You are a member of the tenant and the risk scenario exists: see [Build and price a risk scenario](/opencrq/use/quantify/risk-scenarios/).
- The risk scope has a **Sector** and a **Country** (its **Identity**), and an **Annual revenue** in its financial profile, which sets its revenue band. Without them, the catalogue cannot position the scope and gives the all-sector, all-size, all-region average.

## Read the catalogue's frequency for a scope

The catalogue starts from the average frequency of each attack outcome over every organization of its sources, then multiplies it by the scope's sector, revenue band and zone.

:::steps

1. In the sidebar, under **Risk scopes**, open your risk scope and click :menu[Threat Event Frequency].

2. Read **Positioning**: the attack attempts a year the scope faces across all outcomes, their 90% range, and the factor against the average. The table below it gives the effect of the **Sector**, the **Revenue** and the **Zone**, their **Raw product**, and the **Overlap correction** that pulls the product back toward ×1, because the three factors partly describe the same organizations.

   ::screenshot{id=opencrq-threat-event-frequency-scope alt="The Threat Event Frequency of Grand Meridian France: at most 0.89 attack attempts a year, 2.23 times the average, from the Hospitality, $100M-$1B and Europe, Middle East and Africa factors."}

3. Read **Attack outcomes**: for each outcome, the **Attempts per year** at its most likely value, the **Range**, the **Recurrence** in plain words, the split by **Initial access vector**, and the **Risk Scenarios** of the scope that use it.

:::

A value marked **≤** is **Below the catalog's resolution**: the catalogue's floor, not a measurement, so a total that includes it is an upper bound. **Manual** next to a scenario says its frequency was typed by hand; **To recompute** says it was estimated before the latest change to the scope.

To compare scopes, open :menu[Threats > Threat Event Frequency] instead. Its **Risk scopes** tab lists every scope with its attempts a year, its 90% range and its factor, and **Unpositioned** for a scope the catalogue cannot place. Its **Simulator** tab gives the frequency of any **Sector**, **Country** and **Revenue band**, or of a scope you **Start from a risk scope** and then change.

## Keep the catalogue's frequency

A new scenario is in **Auto** mode. On its **Threat** tab, the **Threat Event Frequency (TEF)** card shows the catalogue's **Min (events/year)**, **Likely (events/year)** and **Max (events/year)** for its attack outcome, read only, and the profile they come from. **See it in the threat event frequency** opens the scope's frequency on that outcome.

When the sector, the country or the revenue band of the scope changes, OpenCRQ recomputes the frequency of every scenario of the scope in **Auto** mode, then prices them again.

## Set the frequency by hand

Use your own estimate when you have a better source than a sector average: your incident history, your sector's incident reports, or what your CTI team sees targeting you.

:::steps

1. Open the risk scenario and its **Threat** tab.

2. On the **Threat Event Frequency (TEF)** card, click **Manual**, the pencil. The three fields open with the values the catalogue gave, ready to edit.

3. Type the **Min (events/year)**, **Likely (events/year)** and **Max (events/year)**. Count attempts, successful or not: the attack run decides how many get through your controls. **Likely** must be above zero, **Likely** at least **Min**, and **Max** at least **Likely**.

   ::screenshot{id=opencrq-threat-event-frequency-manual alt="The Threat Event Frequency card of Ransomware on the Booking Platform in Manual mode with 1, 3 and 10 events a year, 41.8 times the catalogue's 0.072."}

:::

There is no save button: OpenCRQ saves the range a second after you stop typing, as soon as it is valid, and prices the scenario again on it. Under the fields, **Catalog** recalls the catalogue's value and range, and how many times the catalogue your estimate is. Click **Run CRQ** to price the scenario right away.

To go back to the catalogue, click **Auto**, the sparkles: OpenCRQ recomputes the catalogue's frequency for the scope as it is today.

## Check that it works

- The **CRQ** tab of the scenario shows your range as **TEF input**, and a new line in its **Simulation History**.
- On the scope's :menu[Threat Event Frequency], the scenario carries **Manual** in the **Risk Scenarios** column of its outcome.

:::example[Set the ransomware frequency from the sector's incident reports]
On 12 January 2026, the catalogue gives Grand Meridian France 0.072 ransomware attempts a year (range 0.013 to 0.39), about once every 14 years, and Ransomware on the Booking Platform is priced at less than $10K a year. The risk manager reads the hospitality sector's incident reports and the campaigns Example CERT tracks against hotel chains, and estimates attempts at Grand Meridian's size far more often.

On 19 January, on the **Threat** tab of Ransomware on the Booking Platform, they click **Manual** and type:

| Field | Value |
| --- | --- |
| **Min (events/year)** | `1` |
| **Likely (events/year)** | `3` |
| **Max (events/year)** | `10` |

The card reads 41.8 times the catalogue. The next run prices the scenario at $263K a year, from a **TEF input** of 1.000 - 3.000 - 10.000 events/year. The data breach scenario beside it stays in **Auto**, at the catalogue's 0.003 to 1.569 attempts a year.
:::

:::xtm-one[Ask XTM One about a frequency]
With XTM One connected, an AI agent can read a scenario's frequency, in either mode, and set a manual range, for example from the campaigns OpenCTI reports against your sector. The change waits for a person to approve it.
:::

## What's next

- [Set what an attack costs with the financial profile](/opencrq/use/quantify/loss-drivers/), the other half of the price.
- [Build and price a risk scenario](/opencrq/use/quantify/risk-scenarios/), to read the price on the CRQ tab.
- [Follow the threats that target your scopes](/opencrq/use/threats/threat-center/).
