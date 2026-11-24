---
title: Review a year's security budget against the loss it avoided
description: Enter a fiscal year's security spend in an OpenCRQ budget review, and read the annual loss each line avoided, its return and the risk tolerance it kept.
sidebar_position: 3
doc_type: how-to
products:
  - opencrq
personas: [risk-manager, executive]
owner: docs-opencrq
last_reviewed: 2026-11-04
---

A budget review sets twelve months of security spend for one risk scope against the annual loss it avoided. You enter what the budget paid for, line by line, and OpenCRQ prices each line, the projects and the standing costs, and the whole, on the latest simulation of the scope's scenarios. The page then answers a board's questions in order: what was funded, what it returned, and whether the scope stayed within its risk tolerance. If you are new to how OpenCRQ prices a scenario, read [How OpenCRQ quantifies risk](/opencrq/concepts/how-risk-is-quantified/) first.

## Before you start

- Any member of the tenant can create, edit, close and delete budget reviews: no capability is needed.
- The risk scope has scenarios that OpenCRQ has priced: a review is read against them. See [Build and price a risk scenario](/opencrq/use/quantify/risk-scenarios/).
- The scope's controls are declared, and their changes during the period are declared on the day they happened. OpenCRQ reads the posture of each control at the start and at the end of the period from that record. See [Set how strong your controls are and read how they held](/opencrq/use/defenses/controls/).
- To read the third section, the scope or its scenarios have a risk tolerance: an annual loss and the highest probability you accept of exceeding it.
- OpenCRQ works in US dollars: enter every spend in dollars, and read every figure in dollars.

## How a line is priced

Every line of a review has a nature, and the nature decides what its loss avoided means.

| Nature | What it pays for | Its loss avoided |
| --- | --- | --- |
| **Build** | A project: controls deployed, strengthened or extended during the period | The annual loss the period's posture changes of its controls removed: the scope priced with those controls set back to their posture at the start of the fiscal year, minus the scope priced as they stand |
| **Run** | A standing cost: licences, a team, a managed service that keeps controls running | The annual loss its controls protect against: the scope priced with those controls absent, minus the scope priced with them at the level they held at the start of the fiscal year |

The posture the review reads as "as they stand" is today's while the period runs, the period's end once it is over, and the closing day for a review closed before its end. Every figure is priced on the same draws of the uncertain inputs, so the difference between two postures is not blurred by chance.

Each figure has a return, computed the same way as a [treatment plan's](/opencrq/use/decide/treatment-plans/#how-a-plan-is-priced):

> Return = (Loss avoided per year − Spent) / Spent

- **Prudent (p10)** gives the same return with a cautious loss avoided: for each scenario, the loss avoided that nine draws of the uncertain inputs in ten exceed, added up over the scope's scenarios.
- A build line can be **Amortized over (years)**. Its return is then also read against what the work costs a year: the spend divided by the years, shown as **amortized** under the return.

The **Build** and **Run** rows of the review price their lines together, and **Overall** prices everything the review funds as one basket. None of them is the sum of the lines below it: controls that cover the same techniques return less together than one by one, and removing controls together costs more than removing them one by one.

Over a scope with several scenarios, the p10 to p90 band of a figure adds up each scenario's own band, as if the scenarios moved together. The band is wider than the true band of the total, and the prudent return more cautious. They are exact for a scope with one scenario.

Every simulation of one of the scope's scenarios prices the scope's open reviews too, except the background pass that reprices the scenarios whose inputs changed. After that pass, the review names the scenarios that changed since it was priced, and **Re-simulate scenarios** brings it up to date.

## Create a review

:::steps
1. Open :menu[Budget Reviews] in the sidebar, or **Budget Reviews** under a risk scope, then click **Create**. The **Create a budget review** drawer opens.
2. From the list of every scope, choose the **Risk scope**. The scope cannot change afterwards: every figure is priced for it.
3. Set the **Fiscal year start** (a month) and the **Year**. The period runs twelve months from the first day of that month. The year can be this one or one of the four before, and the months not yet started are not offered.
4. Keep or change the **Name**. It follows the period, **Fiscal year 2026** or **Fiscal year 2026-2027**, until you type your own.
5. Choose the **Owner** and write the **Context**: the budget granted, the fiscal year's trade-offs, who the review is presented to.
6. Click **Create**. The review opens, **Open**, with no budget line.
:::

:::tip
Choose a period that starts after your controls were first declared in OpenCRQ. A control first declared during the period counts as moved from **Not deployed**: a build line holding it would claim the whole control as the year's work. The review warns when its period starts before the scope's posture record: **Posture changes are recorded only since** that date.
:::

## Enter what the budget paid for

The **What was funded** section lists the lines and their total. **Add a line** opens the **Add a budget line** drawer.

::screenshot{id=opencrq-budget-reviews-line-drawer alt="The Add a budget line drawer, with the Name, the Build and Run cards of the Nature, the Spent over the period and Amortized over (years) fields, and the one control that moved from Feb 1, 2026 to Apr 12, 2026, the Identity Platform, already held by the line MFA for the front-desk accounts."}

:::steps
1. Enter the **Name** of the line, the project or the cost as your budget names it.
2. Choose its **Nature**: **Build** or **Run**.
3. Enter what it cost in **Spent over the period**. It is required.
4. For a build line, keep or change **Amortized over (years)**, from 1 to 10. It starts at 3. Clear it to read the full spend against one year.
5. Choose its controls:
   - **Build**: the list shows the scope's **Controls that moved** during the period, each with its posture at the start and now. Select the ones the project moved. A control moves in one build line at most: one already held by another line shows **In** and that line's name.
   - **Run**: search the scope's controls in **Add a control**. Each one shows the posture it held at the start of the period, the level the line is priced at.
6. Click **Create**.
:::

When controls moved during the period and no build line holds them, the section says how many, with an **Assign** button that opens a new build line. A review with every move assigned tells the board where all the work of the year went.

Adding or deleting a line, or changing a line's nature or controls, dates the review's figures: they show **Expired**, or **Not priced** for a line or a review never priced yet, until the scenarios are simulated again. Click **Re-simulate scenarios** in the **What it returned** section. Renaming a line or correcting its spend or amortization needs no simulation.

Next to the lines, **Spend by NIST CSF function** draws the spend on the six functions of the National Institute of Standards and Technology (NIST) Cybersecurity Framework (CSF) 2.0. A line counts in every function its controls map to, so the amounts do not add up to the review's spend.

## Read what the spend returned

::screenshot{id=opencrq-budget-review alt="The budget review Security budget 2026 of Grand Meridian France, with its Identity card, the In brief card stating $225K spent, $528K of loss avoided per year, of which $78K by the build work, and 0 of 2 scenarios outside tolerance, and the What was funded section."}

The top of the review answers in one sentence: **From** the period's start **to** its end, the scope spent this much, and the build work removed this much annual loss and brought this many scenarios back within tolerance. Beside it, **In brief** gives the three figures: **Spent**, **Loss avoided per year** (the **Overall** figure, with the part of the build work below it) and the scenarios outside tolerance.

The **What it returned** section is a table read left to right: **Spent**, then **Loss avoided per year** with its p10 to p90 band (added up over the scenarios, as [How a line is priced](#how-a-line-is-priced) explains), then **Return** with its prudent and amortized readings, for the **Build**, **Run** and **Overall** rows. Each line also shows its own chain of figures in **What was funded**.

::screenshot{id=opencrq-budget-reviews-returned alt="The What it returned table of a budget review: Build, $85K spent for $78K of loss avoided per year, a -8.5% return; Run, $140K for $265K, +89.6%; and Overall, $225K for $528K, +134.6%, each with its band and its prudent reading."}

Under it, **Risk over the period** draws the scope's **Simulated ALE** day by day. With a build line, a second curve, **Without the period's work**, shows what the Annualized Loss Expectancy (ALE) would have been without the build work done by each day, and the gap between the two is the **Build work of the period**. Past threats and loss estimates are not kept, only postures: every day is priced on the latest simulation's model, so the curve is an approximation, and it says so.

## Check the risk tolerance

The **Are we within our risk tolerance?** section reads, for each scenario, the probability that a year's loss exceeds its tolerance amount:

- **Outside tolerance today**: the scenarios whose probability exceeds what their tolerance accepts, at the review's posture. Once the period is over, it reads **Outside tolerance at the fiscal year end**, and on a review closed before its end, **Outside tolerance at closing**.
- **Outside tolerance without the period's build work**: the same, with the build lines' controls set back to the start of the fiscal year.
- **Brought back within tolerance by the build work**: the difference, the argument a board remembers.

The table gives each scenario's **Threshold**, its own or else its scope's, then both probabilities. Expand a row to see its loss exceedance curve. A tolerance applies to each scenario on its own, never to the sum of the scope's scenarios. When the scope has none, **Set a risk tolerance** opens the scope's tolerance.

## Close the review

When the fiscal year is presented, close the review to keep its figures as they were shown.

:::steps
1. Click **Close the review**. The button stays disabled, with a hint, while the review has no line, while a figure is not priced, or while a scenario changed since the review was priced: re-simulate first.
2. Confirm. The figures are frozen at that day: later simulations no longer change them, and the lines can no longer be edited.
:::

A closed review shows **Closed** and a **Reopen** button. Reopening drops the frozen figures: the review shows its last figures as **Expired** until you click **Re-simulate scenarios**, which prices it on today's model. The new figures may differ from the ones presented. To delete a review, its lines and its figures, use **Delete** in the menu of its page or of its row in the list.

## Compare the years

The list of reviews shows each review's **Status**, **Period**, **Owner**, **Spent**, **Loss avoided** and **Return**. Above it, four counters read the latest review:

- **Outside tolerance**: the scope's scenarios outside their tolerance, on each scenario's latest simulation.
- **Loss avoided per year**: the latest review's **Overall** figure, with its band and, when an earlier review exists, the change since it.
- **Build return**: the return of the build work alone, with its prudent reading. The run spending is left out: its return is measured against removing every existing control.
- **Spent**: the latest review's total, with the part of build work.

The list of every scope sums the loss avoided of the latest review of each scope, without a band, since the bands of several scopes do not add up.

:::example[Present Grand Meridian France's security budget for 2026]
Grand Meridian France declared its controls on 12 January 2026, and its fiscal year starts on 1 February. On 2 March, once MFA covers every front-desk account, the risk manager opens the review for the year.

In :menu[Budget Reviews], **Create**:

- **Risk scope**: Grand Meridian France
- **Name**: Security budget 2026
- **Fiscal year start**: February; **Year**: 2026
- **Context**: The spend of 2026 against the annual loss it avoids.

Then two lines:

| Field | Build line | Run line |
| --- | --- | --- |
| **Name** | MFA for the front-desk accounts | Endpoint and email security licences |
| **Nature** | **Build** | **Run** |
| **Spent over the period** | 85,000 | 140,000 |
| **Amortized over (years)** | 3 | |
| Controls | IDP Identity Platform (IdP, SSO, MFA, Conditional Access) | EDR Endpoint Detection and Response, SEG Email Security |

**Re-simulate scenarios**, then read the review. On 12 April, **In brief** says the scope spent $225K, of which $85K build and $140K run, and that the build work removes $78K of annual loss. In **What it returned**:

| Row | Spent | Loss avoided per year | Return |
| --- | --- | --- | --- |
| **Build** | $85K, $28K a year amortized | $78K | −8.5%, amortized +174.5% |
| **Run** | $140K | $265K | +89.6% |
| **Overall** | $225K | $528K | +134.6% |

The MFA project does not pay for itself in one year, but it does over the three years it is amortized on. The **Overall** figure, $528K, is more than the build and run figures together, $343K: removing the controls together costs more than removing them one by one. The scope's tolerance is an annual loss of $2.0M exceeded with a probability of 10% at most: **Outside tolerance today** reads 0 of 2 scenarios, and 0 without the period's build work, so Data Breach on Guest Records and Ransomware on the Booking Platform are both **Within tolerance**, with or without the build work.
:::

## What's next

- [Choose the treatment plan that removes the most loss](/opencrq/use/decide/treatment-plans/), to choose next year's build work.
- [Track cyber risk in money with dashboards](/opencrq/use/decide/dashboards/), for the trends of the year.
- [How OpenCRQ quantifies risk](/opencrq/concepts/how-risk-is-quantified/), for the model behind every figure.
