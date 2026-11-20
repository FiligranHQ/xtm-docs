---
title: Set what an attack costs with the financial profile
description: Answer a risk scope's financial questionnaire so OpenCRQ derives what an attack costs each scenario, then correct any loss driver by hand.
sidebar_position: 1
doc_type: how-to
products:
  - opencrq
personas: [risk-manager]
owner: docs-opencrq
last_reviewed: 2026-11-04
---

What an attack costs is the other half of a risk's price, beside how often it happens. OpenCRQ breaks the cost into **loss drivers**: one line each for the forensics firm, the downtime, the notification of the people affected, the regulator's fine. Each driver is a range, minimum, most likely and maximum, in your currency. You do not type them one by one: you answer the **financial profile** of the risk scope, a questionnaire of 21 answers about revenue, downtime, data held and contracts, and OpenCRQ derives the ranges of every scenario of the scope from it. You then correct any driver you know better. [How OpenCRQ quantifies risk](/opencrq/concepts/how-risk-is-quantified/) explains how the amounts enter the price.

## Before you start

- You are a member of the tenant and the risk scope exists: see [Risk scopes, primary assets and supporting assets](/opencrq/concepts/risk-scopes-and-assets/).
- Have the figures at hand: revenue and gross margin from finance, recovery targets from IT, the number of records and of card transactions from the data protection officer, the liability clauses from legal. Bands are enough for most answers.

## Answer the financial profile

You can answer it when you create the risk scope, on its optional **Financial profile** step, or at any time from the scope's details.

:::steps

1. In the sidebar, under **Risk scopes**, open your risk scope, click :menu[Details], then the **Financial profile** tab. It shows how many answers are given, and each answer with the loss drivers it **Informs**.

2. Click **Answer the questionnaire**. The scope's edit panel opens on its **Financial profile** tab.

   ::screenshot{id=opencrq-loss-drivers-questionnaire alt="The Financial profile tab of the Edit Grand Meridian France panel with 21 of 21 answered, its A · Revenue and margin section open on a loaded daily labour cost of 450 USD, an annual revenue of 620,000,000 USD and a gross margin of 32%."}

3. Answer the sections, in any order. Each card names the loss drivers its answer informs, so you can see what a blank costs before leaving it blank.

4. Click **Save**. OpenCRQ recomputes the loss estimates of every scenario of the scope, then prices again those whose amounts moved.

:::

| Section | What it asks | What it drives |
| --- | --- | --- |
| **A · Revenue and margin** | **Loaded daily labour cost**, **Annual revenue**, **Gross margin**, **Operating days per year** and the seasonal peak | Every driver that scales with revenue, the lost margin of an outage. The annual revenue also sets the scope's revenue band, which positions its [threat event frequency](/opencrq/use/threats/threat-event-frequency/) |
| **B · Digital dependency** | The share of turnover that stops when IT stops, the **Maximum tolerable downtime (MTD)**, the **Recovery time objective (RTO)**, whether lost business comes back, how recently a restore was tested | The downtime, restoration and response drivers |
| **C · Data held** | How many people's personal data you hold, card transactions, sensitive records, the largest single repository | Notification, identity protection, claims, regulatory fines, card-brand assessments |
| **D · Regulatory** | The strictest regime that applies: NIS2, DORA or another sectoral supervision | **Regulatory fines** and **Incident legal counsel** |
| **E · Customers and contracts** | The customer base and its B2C share, recurring revenue, revenue under SLA or breach-liability clauses, the contractual liability cap | Customer attrition, crisis communications, contractual penalties, settlements |
| **F · Funds and IP** | The daily value of outbound payments and the approval ceiling above which they are verified out of band, the value of your R&D | **Stolen funds, net of recovery** and **Competitive-advantage erosion** |
| **G · Extortion** | Whether your organization would pay a ransom | **Extortion payment** |
| **H · Loss history** | The worst outage and the costliest incident of the past three years | Widens the worst case of the downtime drivers, and raises the floor the totals are checked against |

A few answers need care:

- **Loaded daily labour cost** comes prefilled from a benchmark for the scope's country. Correct it if you know your own rate.
- **Restore test recency** left blank counts as tested. A restore tested long ago or never widens the worst case of the seven drivers that hang on the restore time, from forensics to customer attrition.
- The workstations and servers of the scope are no longer asked here: declare them on the scope's **Declared estate** tab, beside **Financial profile**. The counts declared there, an exact number or a range, replace the estimate from the scope's size in the forensics and restoration drivers, and lift its low-confidence flag once both are declared. See [Risk scopes, primary assets and supporting assets](/opencrq/concepts/risk-scopes-and-assets/).
- **Ransom payment posture** at **No** records the extortion payment as zero in every scenario, a **declared zero**, rather than leaving it unestimated.
- An outbound payment flow without an approval ceiling leaves the stolen funds unbounded.

The panel warns when an answer contradicts another, such as a recovery time objective more than four times the maximum tolerable downtime.

## Read a scenario's loss estimates

Open a risk scenario and its **Financial Impact** tab. The **Loss Magnitude Summary** gives the **Min**, **Most Likely**, **Max** and **PERT Expected** total of an incident, and how many of the drivers shown are estimated. The table below groups the drivers by loss category, each with its **Source**:

- **Auto**: derived from the financial profile. An icon marks an estimate derived through a proxy or a weak coefficient: treat it as an approximation.
- **Manual**: typed by hand.

A scenario prices the drivers its attack outcome and the business resources of its primary assets call for; the [business resources reference](/reference/opencrq/business-resources/) lists them. Drivers outside that recommendation show under **Not recommended for this scenario**: their amounts are kept and counted.

::screenshot{id=opencrq-loss-drivers-financial-impact alt="The Financial Impact tab of Data Breach on Guest Records: a loss from $19.4M to $125.2M, $52.4M most likely and a PERT expected $59.0M, 10 of 10 drivers estimated, and the response, notification and identity protection drivers, each marked Auto: derived from the financial profile."}

## Correct a driver by hand

:::steps

1. On the scenario's **Financial Impact** tab, click **Edit estimates**.

2. On the driver's row, click **Manual**, the pencil, and type its **Min ($)**, **Most Likely ($)** and **Max ($)**: zero or more, with **Min** at most **Most Likely**, and **Most Likely** at most **Max**. The totals at the top follow as you type.

3. To price a driver the scenario does not recommend, pick it in **Add a not-recommended driver**.

4. Click **Save**.

:::

A manual amount stays when the financial profile changes: OpenCRQ only recomputes the drivers in **Auto**. To return a driver to the profile, click **Auto**, the sparkles, and save; it is unavailable when the profile gives that driver no value. If someone saved the same estimates meanwhile, OpenCRQ reloads the latest values and asks you to review your edits and save again.

## Look up a loss driver

The scope's :menu[Details] has a **Loss Driver** tab: the catalogue of loss drivers, grouped by loss category, with the **Dwell sensitivity** of each, how much the attacker's time inside changes its size. Click a driver for its **Loss category**, its **Scaling variable**, and **When this driver applies**: a grid of attack outcomes by business resource, marked **Applies**, **Realistic pairing, driver not called** or **Pairing not held realistic**. Loss drivers are platform reference data and cannot be edited. The [loss drivers reference](/reference/opencrq/loss-drivers/) lists them all.

## Check that it works

- The scope's **Financial profile** tab counts your answers, such as **21 of 21 answered**.
- A scenario's **Financial Impact** tab shows **Auto** amounts, and none reads **Not estimated** for lack of an answer. While a recompute runs, a notice says the amounts shown are the previous ones.
- The scenario's **Overview** shows the new **Loss magnitude (LM)**, the PERT expected total, and its **CRQ** tab a new run.

:::example[The financial profile of Grand Meridian France]
The risk manager of Grand Meridian France answers the 21 questions with the finance, IT and data protection teams. Among them:

| Answer | Value |
| --- | --- |
| **Loaded daily labour cost** | `450` USD a day |
| **Annual revenue** | `620,000,000` USD, which falls in the $100M-$1B band |
| **Gross margin** | `32` % |
| **Operating days per year** | **365 (continuous)**, seasonal peak uplift `35` % |
| **Share of turnover that stops when IT stops** | `70` % |
| **Maximum tolerable downtime (MTD)** | **< 6h** |
| **Recovery time objective (RTO)** | **3 – 7 days** |
| **Restore test recency** | **Tested over 12 months ago** |
| **Personal data held** | **Yes**, **1M – 5M** individuals |
| **Payment-card transactions** | **500k – 5M** |
| **Regulatory regime** | **NIS2 important** |
| **Ransom payment posture** | **No** |

The panel warns that a recovery target of three to seven days is more than four times a tolerable downtime under six hours. For Data Breach on Guest Records, OpenCRQ derives ten drivers: an incident costs $19.4M at least, $52.4M most likely and $125.2M at most, a PERT expected $59.0M. Notification & call handling alone weighs $9.4M to $21.5M, for the 2.4 million guests to notify.
:::

## What's next

- [Set the threat event frequency](/opencrq/use/threats/threat-event-frequency/), the other half of the price.
- [Build and price a risk scenario](/opencrq/use/quantify/risk-scenarios/).
- [Loss drivers](/reference/opencrq/loss-drivers/) and [business resources](/reference/opencrq/business-resources/) references.
