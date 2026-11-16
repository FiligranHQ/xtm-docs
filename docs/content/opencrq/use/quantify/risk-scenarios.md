---
title: Build and price a risk scenario
description: Create a risk scenario on the primary assets of a risk scope, let OpenCRQ price it as an annualized loss, and read the price on its CRQ tab.
sidebar_position: 2
doc_type: how-to
products:
  - opencrq
personas: [risk-manager]
owner: docs-opencrq
last_reviewed: 2026-11-04
---

A risk scenario is one line of your risk register, priced in money: an attack outcome, such as ransomware or a data breach, on the primary assets of a risk scope. OpenCRQ prices it as an **Annualized Loss Expectancy** (ALE), with the range of annual losses around it, and keeps the price current as threats, controls and assets change. This page creates a scenario, reads its price, and follows it in the scenario list. If you are new to the method, read [How OpenCRQ quantifies risk](/opencrq/concepts/how-risk-is-quantified/) first.

## Before you start

- You are a member of the tenant. Every member works on risk scopes, scenarios and assets; no capability is needed.
- The risk scope exists and holds at least one primary asset, with its business resources. See [Risk scopes, primary assets and supporting assets](/opencrq/concepts/risk-scopes-and-assets/).
- The scope's financial profile is answered, so the scenario's losses have amounts: see [Set what an attack costs with the financial profile](/opencrq/use/quantify/loss-drivers/).
- Threats reach the tenant from OpenCTI: see [Connect OpenCTI with a Live Stream](/opencrq/use/sources/opencti/). Until then, the threats of **OpenCRQ Default Threats** stand in. OpenCRQ prices a scenario against the threats likely to lead to its outcome, weighed more when they target the scope's sector and country: see [Follow the threats that target your scopes](/opencrq/use/threats/threat-center/). A scenario that no threat weighs on stays unpriced.

## Create the scenario

:::steps

1. In the sidebar, under **Risk scopes**, open your risk scope, click :menu[Risk Scenarios], then **Create**. From the tenant's own :menu[Risk Scenarios], the wizard starts with a **Risk scope** step instead.

2. On **Outcome**, choose the attack outcome: what the attacker is after. You cannot change it after creation, because the scenario's timing assumptions and threat intelligence derive from it. The [attack outcomes reference](/reference/opencrq/attack-outcomes/) describes the seven. Click **Next**.

   ::screenshot{id=opencrq-risk-scenarios-create-outcome alt="The Create Risk Scenario wizard on its Outcome step, with the seven attack outcomes from Data Breach & Resale to Service Disruption."}

3. On **Target Assets**, select the primary assets the outcome would hit. The assets under **Suggested for this outcome** hold business resources the outcome has loss drivers for; the others come under **Other assets**. Click **Next**.

4. On **Review & Create**, check the **Name** that OpenCRQ built from the outcome and the assets, and change it to the wording of your risk register. Add a **Description** and an **Owner**, then click **Create Scenario**.

:::

The scenario opens on its **Overview** tab, and OpenCRQ runs its first simulation. When **No loss driver is recommended** shows on the last step, the outcome and the business resources of these assets have no pairing in the loss model: you can still create the scenario and price any driver by hand.

## Check what the price rests on

A scenario's tabs hold its inputs. Two of them decide most of the price:

- **Threat** holds the **Threat Event Frequency (TEF)**, how many attack attempts a year the scenario faces. A new scenario takes it from the catalogue, in **Auto** mode. See [Set the threat event frequency](/opencrq/use/threats/threat-event-frequency/). The same tab lists the **Relevant Threats** OpenCTI brought.
- **Financial Impact** holds the loss estimates, derived from the scope's financial profile. See [Set what an attack costs with the financial profile](/opencrq/use/quantify/loss-drivers/).

**Targets** shows the supporting assets behind the primary assets, **Findings** the vulnerabilities on them, **TTP** the techniques the scenario's threats use, and **Controls** how your controls stand against those techniques. When an input is missing, the scenario says which one, with a link to the tab that fills it.

## Read the price on the CRQ tab

The **Overview** tab gives the headline: the **Annual Loss Exposure**, and the three factors it multiplies, the **Threat event frequency (TEF)**, the **Effective susceptibility** (the share of attacks that land an impact) and the **Loss magnitude (LM)**. The **CRQ** tab shows the latest run in full.

::screenshot{id=opencrq-risk-scenario-crq alt="The CRQ tab of Data Breach on Guest Records: 99.8% of attacks blocked before the objective, a loss event frequency of 0.001 a year, an annualized loss expectancy of $33K with a p10 of $217 and a p90 of $3K, and what the run rests on, with its two admitted actors."}

| Section | What it tells you |
| --- | --- |
| **Attack run outcomes** | The share of attacks **Blocked before the objective**, **Evicted in time**, ending in an **Impact, noticed** or an **Impact, unnoticed (lower bound)**. The four sum to one |
| **Loss Event Frequency (LEF)** | How many loss events a year, with the **TEF input** the run drew from |
| **Annualized Loss Expectancy (ALE)** | The mean annual loss over the simulated years, with the p10 to p90 band that holds eight years in ten. The mean is the figure that adds up across scenarios, the band is not |
| **What this run rests on** | The run date, the **Worlds drawn**, the calibration and the control catalogue, the **Admitted actors**, the **Estate walked** and the **Target boost**. Two runs that differ here are not comparable |
| **Actors** | Each class of attacker the run weighed, its **Share**, how often it **Lands an impact**, and its ALE. When no asset of the scope can be attacked without initial access, the card names the attack path the run did not play |
| **Attacks the run could not score** | The technique and asset pairs no control of yours contests (your gap), and those no control of the catalogue reaches (a modelling gap, absent from every figure) |
| **Technique values** | What the ALE would drop by if a technique were prevented, or fully detected |
| **Simulation History** | Every run, with its ALE mean, LEF mean and technique coverage |

Every **What it means and how it is computed** button explains its figure, with its caveat. Amounts are in your currency; the interface shows dollars today.

## See what changed between two runs

When a scenario has two runs or more, the **Overview** tab shows **What changed** beside the **Annual Loss Exposure**. Click it, or click the exposure or one of its three factors, to open the comparison on that figure. It compares two runs of the scenario: by default the latest one and the last one at least seven days older.

- The tabs choose the figure: **Annual Loss Exposure**, **Threat event frequency (TEF)**, **Effective susceptibility** or **Loss magnitude (LM)**. Each shows the figure **Before** and **After**, its change, and its curve over every run of the scenario.
- **Previous run**, **7 days**, **30 days**, **90 days** and **1 year** pick the two runs, ending on the latest. A click on the curve moves the nearer of the two onto that run. **Zoom in** fits the curve to the runs compared.
- The tiles below count the changes between the two runs by category: **Run results**, **Model**, **Controls**, **Critical findings**, **High findings**, **Other findings**, **Critical assets**, **High assets**, **Other assets** and **Attack frequency**. Click tiles to keep only their categories; **Show all** brings the others back.
- The chronology lists each change in order, from the **Earlier run** to the **Later run**, such as a control's strength or coverage that moved or a TEF switched from automatic to manual, with a link to the tab where it happened.

::screenshot{id=opencrq-risk-scenario-changes alt="What changed for Data Breach on Guest Records, on its Annual Loss Exposure tab over 7 days: $33K on the run of Mar 2, 2026 and $33K on the run of Apr 12, 2026, a curve that fell from $97K in January, and two changes, one in Other findings and one in Run results."}

A change of **Model**, such as a new calibration or control catalogue, moves the figure without any change of your risk: weigh it before you report the change.

## Follow your scenarios

:menu[Risk Scenarios] lists the scenarios of the scope, or of every scope from the tenant's own entry. The **Cards view** shows each scenario's ALE with its p10 to p90 band, the change over its history with a line of past prices, and its **Outcome**, **Targets**, **Owner** and last simulation. The **Lines view** shows the same as columns, with the **Status** of each scenario:

- **Ready**: every input is filled in and reflected by the latest simulation;
- **Stale**: inputs changed after the latest simulation;
- **Incomplete**: the **Threat Event Frequency** or the **Financial Impact** was never filled in;
- **Not simulated**: no simulation has run yet;
- **No mapped asset**: the scope is measured over its mapped assets and holds none.

::screenshot{id=opencrq-risk-scenarios-list alt="The Risk Scenarios of Grand Meridian France as cards: Ransomware on the Booking Platform at $254K a year, down $9K, and Data Breach on Guest Records at $33K, down $64K, each with its p10 to p90 band and a line of past prices."}

## Keep the price current

OpenCRQ prices a scenario again on its own when something its price depends on changes: findings, supporting assets, controls, threats or loss estimates, and once a day for what changes with time alone, such as threat activity. A change of a control's posture or of the scenario's loss estimates also asks for a full run. To price a scenario right away, click **Run CRQ** in its header: the run appears in the **Simulation History**. **Run CRQ** waits while OpenCRQ weighs the scenario's threats, after it is created or when its threats change: its tabs then say **Weighing the threats for this scenario. This page updates on its own once it is done.** The [background processes reference](/reference/opencrq/processes/) lists what runs when.

To change the name, description, owner or primary assets, open **More actions** in the header and choose **Edit**. To delete a scenario, open **More actions** on its card and choose **Delete**.

:::example[Price the theft of the guest records]
On 9 January 2026, the shared night.audit front-desk account exported guest records from Grand Meridian's booking platform at night. The risk manager adds the risk to the register of Grand Meridian France:

| Step | Value |
| --- | --- |
| **Outcome** | **Data Breach & Resale** |
| **Target Assets** | **Guest records**, under **Suggested for this outcome** |
| **Name** | `Data Breach on Guest Records` |
| **Description** | Cybercriminals reach the guest records the booking platform holds, copy them and resell them: 2.4 million guests across the EU to notify, with the data protection authorities, and claims to answer. |

OpenCRQ prices it at $97K a year on 12 January. After multi-factor authentication reaches every front-desk account on 2 March, the run falls to $33K, and stays there: on 12 April the **CRQ** tab reads an ALE of $33K, p10 $217, p90 $3K, from a **TEF input** of 0.003 to 1.569 attempts a year, with 99.8% of the attacks blocked before the objective. Its **Overview** multiplies a TEF of 0.31, an effective susceptibility of 0.2% and a loss magnitude of $59.0M. **What changed**, on its default **7 days**, compares the run of 2 March with the latest one: the same $33K, one new finding and no change of controls.

The mean sits above the p90: a loss event comes about once in a thousand years (LEF mean 0.001), so nearly every simulated year costs little and a rare one costs tens of millions. The ransomware scenario beside it reads $254K with p10 and p90 both at $0, for the same reason. For such a scenario, report the mean and the loss exceedance curve of its **Overview**, not the band.
:::

:::xtm-one[Ask XTM One to build or explain a scenario]
With XTM One connected, an AI agent can create a risk scenario, run a CRQ simulation and read its latest run, for example to explain why the ALE of a scenario moved between two runs. Any change it proposes waits for a person to approve it.
:::

## What's next

- [Set the threat event frequency](/opencrq/use/threats/threat-event-frequency/) of the scenario.
- [Set what an attack costs with the financial profile](/opencrq/use/quantify/loss-drivers/).
- [Choose the treatment plan that removes the most loss](/opencrq/use/decide/treatment-plans/).
- [How OpenCRQ quantifies risk](/opencrq/concepts/how-risk-is-quantified/).
