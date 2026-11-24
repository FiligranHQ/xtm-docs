---
title: Build and price a risk scenario
description: Create a risk scenario on the primary assets of a risk scope, let OpenCRQ price it as an annualized loss, and read the price on its CRQ tab.
sidebar_position: 2
doc_type: how-to
products:
  - opencrq
personas: [risk-manager]
owner: docs-opencrq
last_reviewed: 2026-11-20
---

A risk scenario is one line of your risk register, priced in money: an attack outcome, such as ransomware or a data breach, on the primary assets of a risk scope. OpenCRQ prices it as an **Annualized Loss Expectancy** (ALE), with the range of annual losses around it, and keeps the price current as threats, controls and assets change. This page creates a scenario, reads its price, and follows it in the scenario list. If you are new to the method, read [How OpenCRQ quantifies risk](/opencrq/concepts/how-risk-is-quantified/) first.

## Before you start

- You are a member of the tenant. Every member works on risk scopes, scenarios and assets; no capability is needed.
- The risk scope exists and holds at least one primary asset, with its business resources. See [Risk scopes, primary assets and supporting assets](/opencrq/concepts/risk-scopes-and-assets/).
- The scope's financial profile is answered, so the scenario's losses have amounts: see [Set what an attack costs with the financial profile](/opencrq/use/quantify/loss-drivers/).
- Threats reach the tenant from OpenCTI: see [Connect OpenCTI with a Live Stream](/opencrq/use/sources/opencti/). Without it, the threats of **OpenCRQ Default Threats** can stand in. A new tenant starts with them switched off: someone with **Manage Security** turns them on in the **Threats** step of **Get started**. OpenCRQ prices a scenario against the threats likely to lead to its outcome, weighed more when they target the scope's sector and country: see [Follow the threats that target your scopes](/opencrq/use/threats/threat-center/). A scenario that no threat weighs on stays unpriced.

## Create the scenario

:::steps

1. In the left navigation, open your risk scope, listed below **Budget Reviews**, click :menu[Risk Scenarios], then **Create**. From the tenant's own :menu[Risk Scenarios], the wizard starts with a **Risk scope** step instead.

2. On **Outcome**, choose the attack outcome: what the attacker is after. You cannot change it after creation, because the scenario's timing assumptions and threat intelligence derive from it. The [attack outcomes reference](/reference/opencrq/attack-outcomes/) describes the seven. Click **Next**.

   ::screenshot{id=opencrq-risk-scenarios-create-outcome alt="The Create Risk Scenario wizard on its Outcome step, with the seven attack outcomes from Data Breach & Resale to Service Disruption."}

3. On **Target Assets**, select the primary assets the outcome would hit. The assets under **Suggested for this outcome** hold business resources the outcome has loss drivers for; the others come under **Other assets**. Click **Next**.

4. On **Review & Create**, check the **Name** that OpenCRQ built from the outcome and the assets, and change it to the wording of your risk register. Add a **Description** and an **Owner**, then click **Create Scenario**.

:::

The scenario opens on its **Overview** tab, and OpenCRQ runs its first simulation. When **No loss driver is recommended** shows on the last step, the outcome and the business resources of these assets have no pairing in the loss model: you can still create the scenario and price any driver by hand.

You can also start from a threat you follow. In :menu[Threats > Threats], the header of a threat offers **Quantify risk** when it names the outcome the threat best matches. It opens this wizard with that outcome chosen on **Outcome**, which you can still change. From the tenant's threat center, the wizard starts with the **Risk scope** step. See [Follow the threats that target your scopes](/opencrq/use/threats/threat-center/).

## Check what the price rests on

A scenario's tabs hold its inputs. Two of them decide most of the price:

- **Threat** holds the **Threat Event Frequency (TEF)**, how many attack attempts a year the scenario faces. A new scenario takes it from the catalogue, in **Auto** mode. See [Set the threat event frequency](/opencrq/use/threats/threat-event-frequency/). The same tab lists the **Relevant Threats** that weigh on the scenario, whatever their source: click one to see why it weighs, and whether its probability on the outcome comes from OpenCRQ's model or from the judgment of **OpenCRQ Default Threats**. See [How a threat gets into a scenario](/opencrq/use/threats/threat-center/#how-a-threat-gets-into-a-scenario). When OpenCTI or OpenAEV is connected to the tenant and holds the same threat, its mark beside the threat's name opens it there, in a new tab. Hover the mark to preview what that product holds on the threat first.
- **Financial Impact** holds the loss estimates, derived from the scope's financial profile. See [Set what an attack costs with the financial profile](/opencrq/use/quantify/loss-drivers/).

**Targets** shows the supporting assets behind the primary assets, **Findings** the vulnerabilities on them, **TTP** the techniques the scenario's threats use, and **Controls** how your controls stand against those techniques. When an input is missing, the scenario says which one, with a link to the tab that fills it.

To test the scenario's attackers against your defenses, click **Simulate in OpenAEV** in the scenario's header. It opens, in a new tab, the OpenAEV scenario built from one of the **Relevant Threats**: the one with the highest **Score** among those OpenAEV holds. The button shows only when OpenAEV is connected to the tenant and holds such a scenario. What the simulation proves about your controls comes back through the OpenAEV connector: see [Import OpenAEV results](/opencrq/use/sources/openaev/).

## Read the price on the CRQ tab

The **Overview** tab gives the headline: the **Annual Loss Exposure**, the mean ALE that the **CRQ** tab shows as **Annualized Loss Expectancy**, and the three factors it multiplies, the **Threat event frequency (TEF)**, the **Effective susceptibility** and the **Loss magnitude (LM)**. The **Effective susceptibility** is the share of an attack's loss that lands: an eviction counts 15 % of the loss magnitude, the response it cost, a noticed impact 75 %, an unnoticed impact 100 %, a blocked attack nothing. The **CRQ** tab shows the latest run in full.

::screenshot{id=opencrq-risk-scenario-crq alt="The CRQ tab of Data Breach on Guest Records: the attack run outcomes, nearly every attack blocked before the objective, the loss event frequency with its TEF input, the annualized loss expectancy with its p10 to p90 band, and what the run rests on, with its two admitted actors."}

| Section | What it tells you |
| --- | --- |
| **Attack run outcomes** | The share of attacks **Blocked before the objective**, **Evicted in time**, ending in an **Impact, noticed** or an **Impact, unnoticed (lower bound)**. The four sum to one |
| **Loss Event Frequency (LEF)** | How many loss events a year, with the **TEF input** the run drew from |
| **Annualized Loss Expectancy (ALE)** | The mean annual loss, with its p10 to p90 band: the range the expected annual loss takes in eight worlds out of ten, as the uncertain inputs vary. It is not the range of a single year's loss. The mean is the figure that adds up across scenarios, the band is not |
| **What this run rests on** | The run date, the **Worlds drawn**, the calibration and the control catalogue, the **Admitted actors**, the **Estate walked** and the **Target boost**. Two runs that differ here are not comparable |
| **Actors** | Each class of attacker the run weighed, its **Share**, how often it **Lands an impact**, and its ALE. When no asset of the scope can be attacked without initial access, the card names the attack path the run did not play |
| **Attacks the run could not score** | The technique and asset family pairs no control of yours contests (your gap), and those no control of the catalogue reaches (a modelling gap, absent from every figure) |
| **Technique values** | For each technique, what the ALE would drop by if a control stopped it every time (**By preventing it**), if its detection were fully covered (**By detecting it**), and the two added (**Value of interdicting**). Each is a ceiling for that technique alone: the amounts overlap and do not add up across techniques |
| **Simulation History** | Every run, with its ALE mean, LEF mean and technique coverage |

Every **What it means and how it is computed** button explains its figure, with its caveat. OpenCRQ works in US dollars: every amount you enter and every price it gives is in dollars.

## See what changed between two runs

When a scenario has two runs or more, the **Overview** tab shows **What changed** beside the **Annual Loss Exposure**. Click it, or click the exposure or one of its three factors, to open the comparison on that figure. It compares two runs of the scenario: by default the latest one and the last one at least seven days older.

- The tabs choose the figure: **Annual Loss Exposure**, **Threat event frequency (TEF)**, **Effective susceptibility** or **Loss magnitude (LM)**. Each shows the figure **Before** and **After**, its change, and its curve over every run of the scenario.
- **Previous run**, **7 days**, **30 days**, **90 days** and **1 year** pick the two runs, ending on the latest. A click on the curve moves the nearer of the two onto that run. **Zoom in** fits the curve to the runs compared.
- The tiles below count the changes between the two runs by category: **Run results**, **Model**, **Controls**, **Critical findings**, **High findings**, **Other findings**, **Critical assets**, **High assets**, **Other assets** and **Attack frequency**. Click tiles to keep only their categories; **Show all** brings the others back.
- The chronology lists each change in order, from the **Earlier run** to the **Later run**, such as a control's strength or coverage that moved or a TEF switched from automatic to manual, with a link to the tab where it happened.

::screenshot{id=opencrq-risk-scenario-changes alt="What changed for Data Breach on Guest Records, on its Annual Loss Exposure tab over 7 days: the exposure on the run of Mar 2, 2026 and on the run of Apr 12, 2026, its curve over every run since January, lower after March, and two changes, one in Other findings and one in Run results."}

A change of **Model**, such as a new calibration or control catalogue, moves the figure without any change of your risk: weigh it before you report the change.

## Follow your scenarios

:menu[Risk Scenarios] lists the scenarios of the scope, or of every scope from the tenant's own entry, where each card names its risk scope above the scenario. The **Cards view** shows each scenario's ALE with its p10 to p90 band, the change over its history with a line of past prices, and its **Outcome**, **Targets**, **Owner** and last simulation. The **Lines view** shows the same as columns, with the **Status** of each scenario:

- **Ready**: every input is filled in and reflected by the latest simulation;
- **Stale**: inputs changed after the latest simulation;
- **Incomplete**: the **Threat Event Frequency** or the **Financial Impact** was never filled in;
- **Not simulated**: no simulation has run yet;
- **No mapped asset**: the scope is measured over its mapped assets and holds none.

::screenshot{id=opencrq-risk-scenarios-list alt="The Risk Scenarios of Grand Meridian France as cards: Ransomware on the Booking Platform at $254K a year, down $9K, and Data Breach on Guest Records, down since January, each with its ALE, its p10 to p90 band and a line of past prices."}

## Keep the price current

OpenCRQ prices a scenario again on its own when something its price depends on changes: findings, supporting assets, controls, threats or loss estimates, and once a day for what changes with time alone, such as threat activity. These automatic runs update the price only. A full run also recomputes the what-ifs: the technique values, the CVE values, the control levers and the treatment plans. **Run CRQ**, a control's posture declared on the risk scope, and a financial profile change that moves the scenario's loss estimates each ask for one. A control's strength or coverage changed for the whole tenant recomputes that control's levers only. A card whose figures are older than the price says **Computed on** a date **before the current ALE**, and offers to recompute them. To price a scenario right away, click **Run CRQ** in its header: the run appears in the **Simulation History**. **Run CRQ** waits while OpenCRQ weighs the scenario's threats, after it is created or when its threats change: its tabs then say **Weighing the threats for this scenario. This page updates on its own once it is done.** The [background processes reference](/reference/opencrq/processes/) lists what runs when.

To change the name, description, owner or primary assets, open **More actions** in the header and choose **Edit**. To delete a scenario, open **More actions** on its card and choose **Delete**.

:::example[Price the theft of the guest records]
On 9 January 2026, the shared night.audit front-desk account exported guest records from Grand Meridian's booking platform at night. The risk manager adds the risk to the register of Grand Meridian France:

| Step | Value |
| --- | --- |
| **Outcome** | **Data Breach & Resale** |
| **Target Assets** | **Guest records**, under **Suggested for this outcome** |
| **Name** | `Data Breach on Guest Records` |
| **Description** | Cybercriminals reach the guest records the booking platform holds, copy them and resell them: 2.4 million guests across the EU to notify, with the data protection authorities, and claims to answer. |

OpenCRQ prices it on 12 January, its TEF in **Auto** mode, read in the TEF catalogue for the scope's country, sector and size. After multi-factor authentication reaches every front-desk account on 2 March, the price falls, and stays there. On 12 April, the **CRQ** tab shows nearly every attack blocked before the objective. **What changed**, on its default **7 days**, compares the run of 2 March with the latest one: one new finding and no change of controls.

When attacks rarely land, the mean can sit above the p90: nearly every world prices an expected loss close to zero, and a rare one tens of millions. The ransomware scenario beside it reads $254K with p10 and p90 both at $0, for that reason. For such a scenario, report the mean and the loss exceedance curve of its **Overview**, not the band.
:::

:::xtm-one[Ask XTM One to build or explain a scenario]
With XTM One connected, an AI agent can create a risk scenario, run a CRQ simulation and read its latest run, for example to explain why the ALE of a scenario moved between two runs. When the agent asks before running tools, its changes to a scenario, its loss estimates or the risk tolerance wait for a person to approve them, unless a rule lets them run: see the [tool approvals](/xtm-one/administer/ai-governance/#approve-what-agents-do).
:::

## What's next

- [Set the threat event frequency](/opencrq/use/threats/threat-event-frequency/) of the scenario.
- [Set what an attack costs with the financial profile](/opencrq/use/quantify/loss-drivers/).
- [Choose the treatment plan that removes the most loss](/opencrq/use/decide/treatment-plans/).
- [How OpenCRQ quantifies risk](/opencrq/concepts/how-risk-is-quantified/).
