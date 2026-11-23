---
title: Set the threat event frequency
description: Read the attack frequency OpenCRQ's catalogue gives a risk scope for its country, sector and size, and replace it with your own estimate for a risk scenario when you know better.
sidebar_position: 2
doc_type: how-to
products:
  - opencrq
personas: [risk-manager]
owner: docs-opencrq
last_reviewed: 2026-11-20
---

The **Threat Event Frequency** (TEF) of a risk scenario is how many attack attempts it faces in a year, successful or not, as a range: a minimum, a most likely value and a maximum. OpenCRQ draws from that range in every simulated year, so the frequency scales the whole price. By default, a scenario takes its frequency from OpenCRQ's catalogue, read for its risk scope's country, sector and size. This page shows how to read that frequency, and how to set your own when your incident data or your threat intelligence says otherwise. [How OpenCRQ quantifies risk](/opencrq/concepts/how-risk-is-quantified/) explains where the frequency enters the price.

## Before you start

- You are a member of the tenant and the risk scenario exists: see [Build and price a risk scenario](/opencrq/use/quantify/risk-scenarios/).
- The risk scope has a **Country**, a **Sector** and an **Employee Count** (its **Identity**). The employee count weighs the most. Without it, the scope's revenue band, set by the **Annual revenue** of its financial profile, gives its size. A part left empty reads every value of it: every country, every sector, or every size.

## How the catalogue reads a scope

The catalogue gives, for each attack outcome, a range of attempts per year by country, by sector and by size class, in employees: 0-9, 10-49, 50-249, 250-999, 1,000-4,999 and 5,000 or more. OpenCRQ reads the row of the scope's country, sector and size class:

- When the catalogue has no row for that sector in that country, it reads the country's row for every sector, then for every sector and every size, then the same rows for every country.
- Without an employee count, OpenCRQ reads the size classes the revenue band spans, at $100K to $600K of revenue per employee, and mixes their ranges by weight. Without a revenue band either, it mixes every size class alike.

Some rows do not measure their country, sector or size for themselves: their figure comes from wider data, and their range is wider. The screen marks those parts, so that you know which figures to challenge first.

## Read the catalogue's frequency for a scope

:::steps

1. In the sidebar, under **Risk scopes**, open your risk scope and click :menu[Threat Event Frequency]. The line under the title names the catalogue in force: its version, its date and its sources.

2. Read **Profile**. **Attack attempts per year** is the sum of the most likely values of every attack outcome, with how often that is in plain words. **Range** goes from the sum of the outcomes' minimums to the sum of their maximums, on a scale from 0.01 to 100 a year. Beside them, **Country**, **Sector** and **Size** show what the catalogue read for the scope.

   ::screenshot{id=opencrq-threat-event-frequency-scope alt="The Threat Event Frequency of Grand Meridian France: 0.702 attack attempts a year, about once every 1.4 years, read for France, Hospitality and 4,200 employees, the size marked as not measured."}

3. Read **Attack outcomes**: for each outcome, the **Attempts per year** at its most likely value, the **Range**, the **Recurrence** in plain words, and the **Risk Scenarios** of the scope that use it.

:::

A **Country**, **Sector** or **Size** with an info icon is a part the catalogue does not measure for the scope. Hover it to read why:

| Part | What the hint says | What it means |
| --- | --- | --- |
| **Country** | Not filled in, or not measured | The figure comes from every country, or the country's level comes from a group of countries. |
| **Sector** | Not filled in, or not measured | The figure comes from every sector, of the country or of every country, or from other countries' surveys of that sector. |
| **Size** | Not filled in, inferred from the revenue, or not published by the surveys | The figure is the average across all sizes, the mix of the classes the revenue band spans, or extended from the sizes the surveys publish. |

A scope without an employee count shows **Fill in the employee count** next to its size: it opens the scope to edit. Of the three parts, the size moves the figure the most.

In the **Risk Scenarios** column, **Manual** says a scenario of that outcome runs on a frequency typed by hand that differs from the catalogue's. **To recompute** says a scenario in **Auto** mode was estimated before the latest change to the scope, or on another catalogue. Hover the chip to read the catalogue's value and each scenario's.

To compare scopes, open :menu[Threats > Threat Event Frequency] instead. Its **Risk scopes** tab lists every scope with its **Country**, **Sector** and **Size**, its **Attempts/yr**, its **Range**, and one column of attempts a year per attack outcome. Type in **Search a risk scope...** to filter, and click a row to open that scope's frequency.

::screenshot{id=opencrq-threat-event-frequency-scopes alt="The Risk scopes tab of Threat Event Frequency: Grand Meridian France and Grand Meridian Spain, both Hospitality, with their employee counts, attempts a year, range and one column per attack outcome."}

Its **Simulator** tab gives the frequency of any profile, before you create a scope or to see what a change would do. Under **Simulated profile**, choose a **Country**, a **Sector** and a **Number of employees**, or **Unknown** and then a **Revenue band**. **Start from a risk scope** loads a scope's profile to change from there, and **Back to the default profile**, or back to the profile of that scope, undoes your changes. Without a scope, the simulator starts from the United States of America, Finance, and a revenue of $100M to $1B.

## Keep the catalogue's frequency

A new scenario is in **Auto** mode. On its **Threat** tab, the **Threat Event Frequency (TEF)** card shows the catalogue's **Min (events/year)**, **Likely (events/year)** and **Max (events/year)** for its attack outcome, read only. Under them come the attack outcome and the country, sector and size the catalogue read, with the same info icon on a part it does not measure. **See it in the threat event frequency** opens the scope's frequency on that outcome.

When the country, the sector, the employee count or the revenue band of the scope changes, OpenCRQ recomputes the frequency of every scenario of the scope in **Auto** mode, then prices them again.

## Set the frequency by hand

Use your own estimate when you have a better source than the catalogue: your incident history, your sector's incident reports, or what your CTI team sees targeting you.

:::steps

1. Open the risk scenario and its **Threat** tab.

2. On the **Threat Event Frequency (TEF)** card, click **Manual**, the pencil. The three fields open with the values the catalogue gave, ready to edit.

3. Type the **Min (events/year)**, **Likely (events/year)** and **Max (events/year)**. Count attempts, successful or not: the attack run decides how many get through your controls. **Likely** must be above zero, **Likely** at least **Min**, and **Max** at least **Likely**.

   ::screenshot{id=opencrq-threat-event-frequency-manual alt="The Threat Event Frequency card of Ransomware on the Booking Platform in Manual mode with 1, 3 and 10 events a year, 51.9 times the catalogue's 0.0578."}

:::

There is no save button: OpenCRQ saves the range a second after you stop typing, as soon as it is valid, and prices the scenario again on it. Under the fields, **Catalog** recalls the catalogue's value and range for the scope today, and how many times the catalogue your estimate is; hover that ratio to read the outcome, country, sector and size it compares with. Click **Run CRQ** to price the scenario right away.

To go back to the catalogue, click **Auto**, the sparkles: OpenCRQ reads the catalogue's frequency for the scope as it is today.

## Check that it works

- The **CRQ** tab of the scenario shows your range as **TEF input**, and a new line in its **Simulation History**.
- On the scope's :menu[Threat Event Frequency], the scenario carries **Manual** in the **Risk Scenarios** column of its outcome.

:::example[Set the ransomware frequency from the sector's incident reports]
On 12 January 2026, the catalogue gives Grand Meridian France, a French hospitality company of 4,200 employees, 0.0578 ransomware attempts a year (range 0.026 to 0.13), about once every 17 years. Its **Size** carries the info icon: the surveys do not publish the 1,000-4,999 class for French hospitality, so the figure is extended from the published sizes. The risk manager reads the hospitality sector's incident reports and the campaigns Example CERT tracks against hotel chains, and estimates attempts at Grand Meridian's size far more often.

On 19 January, on the **Threat** tab of Ransomware on the Booking Platform, they click **Manual** and type:

| Field | Value |
| --- | --- |
| **Min (events/year)** | `1` |
| **Likely (events/year)** | `3` |
| **Max (events/year)** | `10` |

The card reads 51.9 times the catalogue. The next run prices the scenario from a **TEF input** of 1.000 - 3.000 - 10.000 events/year. The data breach scenario beside it stays in **Auto**, at the catalogue's 0.046 to 0.24 attempts a year.
:::

:::xtm-one[Ask XTM One about a frequency]
With XTM One connected, an AI agent can read a scenario's frequency, in either mode, and set a manual range, for example from the campaigns OpenCTI reports against your sector. The change waits for a person to approve it.
:::

## What's next

- [Set what an attack costs with the financial profile](/opencrq/use/quantify/loss-drivers/), the other half of the price.
- [Build and price a risk scenario](/opencrq/use/quantify/risk-scenarios/), to read the price on the CRQ tab.
- [Follow the threats that target your scopes](/opencrq/use/threats/threat-center/).
