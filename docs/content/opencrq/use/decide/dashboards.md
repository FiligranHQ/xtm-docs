---
title: Track cyber risk in money with dashboards
description: Read OpenCRQ's CISO dashboard and its twelve-month trends, pick the dashboard each person opens on Home, and present the cyber risk of a scope in money.
sidebar_position: 1
doc_type: how-to
products:
  - opencrq
personas: [risk-manager, executive]
owner: docs-opencrq
last_reviewed: 2026-11-04
---

OpenCRQ's dashboards put the cyber risk of your organization in money on one screen: the annual loss to expect, how likely a loss is this year, how far each scenario sits from your risk tolerance, and which control would remove the most. Every figure comes from the latest simulation of each risk scenario, so the trends move when OpenCTI reports a new threat or a control changes. This page shows how to read them, how to narrow them to one risk scope, and how to choose the dashboard Home opens on.

## Before you start

- Every member of the tenant reads the dashboards and chooses their own home dashboard: no capability is needed.
- Setting the home dashboard of the whole tenant needs the **Manage Security** capability.
- The figures need priced risk scenarios. See [Build and price a risk scenario](/opencrq/use/quantify/risk-scenarios/).
- OpenCRQ shows every amount in dollars today; the choice of currency is coming.

## The dashboards

OpenCRQ ships five dashboards. :menu[Dashboards] in the sidebar lists them, with a search field and, in the **Home** column, which one is **My home** and which one is the **Tenant home**.

::screenshot{id=opencrq-dashboards-list alt="The list of dashboards: CISO, Risk treatment, Risk (Sankey), Findings & Assets and Remediation pace, each with its description, the CISO dashboard marked My home and Tenant home."}

| Dashboard | What it answers |
| --- | --- |
| **CISO** | Aggregate risk, recommendations, top risks and scope inputs. It is the default home dashboard |
| **Risk treatment** | Each risk scenario's likelihood, annual loss exposure, loss exceedance curve against its tolerance, and the controls to improve first |
| **Risk (Sankey)** | How the annual loss expectancy flows from threat actors to attack outcomes, techniques and loss drivers, and the levers that reduce it |
| **Findings & Assets** | The findings backlog, their triage and the exposed estate |
| **Remediation pace** | Incoming findings against the team's closing capacity, per severity band, over a **Time window** you choose |

Each dashboard has a **Risk scope** menu at the top right: **All risk scopes** adds up every scope of the tenant, and a scope narrows every figure to it. Under a risk scope in the sidebar, **Overview** opens your home dashboard on that scope, and **Dashboards** lists the five dashboards for it.

## Read the CISO dashboard

The CISO dashboard reads from top to bottom like a board briefing.

::screenshot{id=opencrq-dashboards-aggregate-risk alt="The Aggregate risk group of the CISO dashboard: a Likelihood of 4%, up 3 points, a Loss magnitude of $12.6M, down $28.4M, and an Annual loss exposure of $512K, up $182K, each with its twelve-month curve."}

- **Risk scope**: what the figures cover, the tenant or one scope, with its count of risk scenarios.
- **Aggregate risk**: three figures, each with a curve of the last twelve months and its change:
  - **Likelihood**: the probability of at least one loss event in the year, across the scope's scenarios.
  - **Loss magnitude**: the mean loss per loss event.
  - **Annual loss exposure**: the loss to expect on average over a year, the loss event frequency times the loss magnitude. It is the sum of the scenarios' mean annual loss; no band is given, since adding the scenarios' bands would assume they all have their worst year at once.
- **Risk treatment recommendations**: the **Top recommended controls**, the control moves that would remove the most annual loss, one per control, and the **Top recommended findings** to remediate first. Do not add up the **ALE reduction** column: each move is priced as if the others did not happen. **Create a risk treatment plan** starts a plan from here: see [Choose the treatment plan that removes the most loss](/opencrq/use/decide/treatment-plans/).
- **Gap to tolerance**: how far each scenario sits from the risk tolerance applied to it, its own or its scope's. A bar to the right overruns the tolerance by that amount; to the left, it keeps that margin. Scenarios with no tolerance are counted below, with a **Set its risk tolerance** button.
- **Where the risk sits**: the **Riskiest risk scopes**, ranked by annual loss exposure; the **Top risks**, a flow from primary assets to scenarios, attack outcomes and loss categories with each one's share; and the **Worst performing risk scenarios** and **Best performing risk scenarios**, whose likelihood rose or fell most over the last month.
- **Threats**, **Attack surface** and **Defenses**: the threats OpenCTI streams to the tenant, the estate and its findings, and how the controls stand.

The **What it means and how it is computed** button next to a figure or a group gives its definition, its computation and its limits.

## Read a trend

The curves are drawn from the daily figures of the last twelve months, each day reading the latest simulation of every scenario on that day. The chip next to each figure is the change from the first day of the curve to today, red when the risk rises and green when it falls. When the scenarios were first priced less than twelve months ago, the curve starts on that day, and the chip's tooltip says **Change since** that month instead of **Change over 12 months**.

A step in a curve is a day something changed in the model: a new threat streamed from OpenCTI, a control declared at a new posture, an estimate set by hand. To explain a step to a board, open the scenario that moved: its own history says which input changed.

## Choose the dashboard Home opens on

Each person chooses their own; an administrator chooses the one for everyone who has not.

:::steps
1. Open your profile: click **Account** in the top bar, then **Profile**.
2. In the **Preferences** card, choose a dashboard in **Home dashboard**. **Tenant home** followed by the tenant's choice in parentheses keeps the tenant's home dashboard, and follows it when it changes.
:::

The choice is saved at once, and **Home** opens that dashboard from then on. On a new tenant whose **Get started** wizard is not finished, the first time you open the tenant in a browser, OpenCRQ opens **Get started** instead; **Home** in the sidebar opens your dashboard. See [Set your profile and follow your notifications](/opencrq/use/profile/).

To set the tenant's home dashboard, open :menu[Settings > Parameters] and choose a dashboard in **Home dashboard**: it is shown on Home to every user who has not chosen their own. Without the **Manage Security** capability, the field shows the current choice and cannot be changed.

:::example[Show the board how the risk of Grand Meridian Hotels moved since January]
On 12 April 2026, the risk manager of Grand Meridian Hotels opens Home, the **CISO** dashboard, on **All risk scopes**, and presents the **Aggregate risk** group:

- **Annual loss exposure**: $512K, up $182K since January 2026.
- **Likelihood**: 4%, up 3 points.
- **Loss magnitude**: $12.6M per loss event, down $28.4M.

The **Annual loss exposure** curve tells the story of the quarter: $330K on 12 January, when the scenarios were first priced; a step up on 19 January, once the risk manager set the ransomware frequency from the sector's incident reports, 1 to 10 attempts a year, 3 most likely; a step down on 2 March, when MFA covered every front-desk account; and a rise again on 10 April, when OpenCTI reported Ember Tide's new campaign against hotel chains, to $512K.

**Riskiest risk scopes** puts Grand Meridian France first, at $287K, ahead of Grand Meridian Spain at $225K. In **Top risks**, the online booking platform carries $254K, about half of the exposure. Among the **Top recommended controls** is Secure Remote Access (VPN / ZTNA), not deployed at Grand Meridian France yet. That is the next plan to price.
:::

## What's next

- [Choose the treatment plan that removes the most loss](/opencrq/use/decide/treatment-plans/), to turn a recommendation into a costed plan.
- [Review a year's security budget against the loss it avoided](/opencrq/use/decide/budget-reviews/), to report what last year's spend bought.
- [How OpenCRQ quantifies risk](/opencrq/concepts/how-risk-is-quantified/), for the model behind every figure.
