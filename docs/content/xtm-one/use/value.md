---
title: Measure what your agents' work is worth
description: Read the Value page of XTM One to see the hours your AI agents saved, what that time is worth once the AI cost is paid, the return and the payback, by fleet, by agent or by unit of work.
doc_type: how-to
products: [xtm-one]
personas: [executive, soc-manager, admin]
owner: docs-xtm-one
last_reviewed: 2026-11-10
edition: ee
---

The **Value** page answers the question a committee asks about AI agents: what is their work worth? It shows the hours of work your agents saved people, and what those hours are worth at the rate of the role that would have done them. It subtracts the AI cost, then gives the net value, the return on investment (ROI) and the payback. You read it for your own agents and fleets or for the whole organization, by fleet, by agent or by unit of work. Every figure opens on how it is computed, so you can defend it.

::screenshot{id=xtm-one-value-page alt="The Value page of the whole organization over the last 30 days: the cards from Hours saved to Payback, then the By agent tab, where Velvet Harbor watcher shows its runs, units, hours saved, value and net value."}

## Before you start

- **The Enterprise Edition**: on the Community Edition, the page says "Value is part of Enterprise Edition." with the ways to upgrade. See [What each edition of XTM One brings](/xtm-one/administer/editions/).
- **Every user** opens it: from :menu[Work > Value], from **See the value** on the **Value** card of the [home dashboard](/xtm-one/use/dashboard/), or from **Open Value** in [Observability](/xtm-one/administer/observability/). It stays open when AI is switched off: reading it never starts a run nor calls a model.
- **Only declared work counts**: an agent or a fleet shows once it declares a unit of work, such as a threat report triaged, and the minutes a person would spend on it. See [Create an agent](/xtm-one/build/agents/) and [Run agents as a team with fleets](/xtm-one/build/fleets/).

## Read the value of your agents

:::steps
1. Open :menu[Work > Value].
2. Pick the scope, next to the line that names it:
   - **Mine** ("Your agents and fleets"): the agents and fleets you can see, and the fleets of the workspaces shared with a group you manage.
   - **Whole organization**: every agent and fleet. An administrator opens the page on it. Anyone else gets it while the organization totals are shown to everyone, which is the default; they open the page on **Mine**.
3. Pick the period: **Last 7 days**, **Last 30 days** (the default), **Last 90 days**, **This quarter** or **All time**.
4. Read the cards. Click the info button of a figure, **How this is computed**, to see its formula with this period's numbers.
5. Pick a tab, **By fleet**, **By agent** or **By unit of work**, to see where the value comes from.
:::

The scope, the period, the tab, the search and the sort stay in the address: a colleague who opens your link gets the same view, within what they may read.

## What the figures say

| Card | What it says |
| --- | --- |
| **Hours saved** | The time people did not spend producing the units, review time deducted |
| **Value** | What those hours are worth, at the hourly rate of the role that would do the work |
| **AI cost** | The cost of the model calls of the agents in view |
| **Net value** | The value minus the AI cost |
| **ROI** | The net value divided by the AI cost, as a multiple: "5 x" means the net value is five times what the AI cost |
| **Payback** | How long the net value takes to pay back the setup cost. With no setup cost, it reads **Under a month**, with "No setup cost" under it |

Under a figure, a range such as "96 to 170 h" goes from a low to a high estimate: the catalog gives each unit a low, likely and high number of manual minutes. A figure that reads "-" has a **Why there is no figure** tip: "Not enough data in this period to compute it." The ROI has none when no model cost was recorded, and the payback none when the net value is not positive.

**How this is computed** opens on the period, the formula with its numbers, and the **Inputs**. Each input carries its origin: **Default**, **Customized**, **Assumption**, **Published estimate** or **Measured**, with its source. It also says what was counted and what was not. **Read the methodology** opens the full model: see [How XTM One computes the value of agents' work](/xtm-one/concepts/value-methodology/).

While some minutes or rates are still the catalog's, a note says "Estimated: some minutes or rates are catalog defaults. Set your own to replace them." **Units and rates** opens the units of work in Settings, for an administrator. Present the figures as an estimate until your own minutes and rates replace the defaults.

Under the cards:

- **The review line** says how much of the work a person checked: "Reviewed: 3 of 12 runs (25%)". When work items count units, a second count follows, such as "Work items: 2 of 5 reviewed (40%)". Review is optional: a run nobody reviewed counts like an accepted one. **Review the rest**, while some runs wait, opens the runs in :menu[Work > Operations]; **Open the work items** opens the work items. On the **Whole organization** view, someone who is not an administrator gets no link: "Totals of the whole organization; you can open only your own runs."
- **Value over time** draws the value and the net value per week, with the range shaded ("Range: low to high estimate"). Under two weeks of data, it says "Not enough history yet".
- **The cost note** says what the AI cost covers: the model cost of the agents in view. Usage that no agent carries, such as the general chat and the LLM proxy, is not counted.

## See where the value comes from

The three tabs list the same figures, the highest value first: **Name**, **Runs**, **Units**, **Hours saved**, **Value**, **AI cost**, **Net value** and **Reviewed**. Every column sorts, and the search finds a row by name (**Search fleets...**, **Search agents...**, **Search units of work...**). **Columns** hides the **Runs**, **Units** or **AI cost** column. A fleet opens its page. An agent opens the **Value** tab of its page, which shows its ROI and its run budget: see [Create an agent](/xtm-one/build/agents/).

| Tab | One row per | Use it to |
| --- | --- | --- |
| **By fleet** | Fleet | Compare the teams of agents |
| **By agent** | Agent | Find the agent that delivers the most, or one that costs more than it saves |
| **By unit of work** | Kind of work, such as a threat report triaged | See which work the agents take off people's hands |

The rows add up to the cards. What no row of the tab holds is summed in a last row, with a tip that says what it is:

- **Agents outside a fleet**, under **By fleet**: the runs of the agents that belong to no fleet.
- **Other agents and fleets**: the agents and fleets you cannot open, on the **Whole organization** view.
- **Cost not tied to a unit**, under **By unit of work**: the AI cost of the runs that produced no unit of work. The cost note gives its amount.

## Count what agents finished in a chat

The **From chat, estimated** card counts the decks, reports, videos and documents an agent finished in a conversation. It is kept apart from the figures above and never added to them. A deliverable is counted once it is used, or 7 days after its last edit; until then it is **Waiting for use**. Its hours saved are the low estimate of the time by hand, minus the time spent in the conversation plus 25%.

- The card says how many deliverables are counted and how many wait. Its table lists each **Deliverable**, its **Agent**, the **Person** who asked for it, its **Status** (**Waiting for use**, **Counted** or **Corrected**), its **Hours saved** and **Value**. Filter it by **Kind** (**Deck**, **Report**, **Video**, **Document**) and **Status**, or search it.
- **Adjust**, on a counted line, opens **Adjust the hours saved**. Type the **Hours saved**, between 0 and 720, then **Save**: your figure replaces the estimate for everyone who reads it, and nothing waits on it. **Back to the estimate** removes your figure.

## When costs are hidden

When the deployment hides cost data, your own view, and the organization's for someone who is not an administrator, show three cards: **Hours saved**, **Value** and **AI cost**. The cost reads **Hidden**, captioned "Costs hidden on this deployment", never 0: the value is shown before the AI cost. An administrator hides costs at install, with `OBSERVABILITY_COST_DATA_ENABLED`: see the [configuration reference](/reference/xtm-one/configuration/#advanced-tuning).

## When the page is empty

- **"Nothing was counted in this period."**: no counted run in the period. **Widen the window** moves to the next longer period.
- **"No agent is measured yet."**, on **All time**: no agent or fleet declares a unit of work yet. An administrator gets **Pick a unit on an agent**, which opens the agents. Anyone else reads "Your administrator or an agent's owner declares what an agent produces."
- **"The value report could not be read."**: click **Retry**.

The page has no export. The CSV file and the PDF value report are on the **Outcomes** tab of [Observability](/xtm-one/administer/observability/), for administrators.

:::example[Bring the agents' value to the quarterly committee]
On Monday 13 April 2026, Hélène Marchetti, CISO of Grand Meridian Hotels and XTM One administrator, prepares the quarterly security committee. She opens :menu[Work > Value]: the page opens on **Whole organization**. She picks **Last 90 days**.

On **By agent**, **Velvet Harbor watcher** comes first. Its assignment **Weekly Velvet Harbor brief** ran 13 times, and each run counts one **Threat report triaged for relevance**. On the **Value** card, **How this is computed** shows 13 units x (20 - 3) minutes at $88.73 an hour: about $327. Its **Inputs** mark the 20 minutes as an **Assumption** and the analyst's rate as a **Published estimate**.

The page also says "Estimated: some minutes or rates are catalog defaults." Hélène follows **Units and rates** to Settings and, in **Rates by role**, gives the CTI analyst Grand Meridian's own loaded rate. Every figure of the page follows. She quotes **Hours saved** and **Net value** in her deck, with the range under each.
:::

## What's next

- [How XTM One computes the value of agents' work](/xtm-one/concepts/value-methodology/): the model, where each input comes from and what is counted.
- [Review what agents did](/xtm-one/use/work-history/): accept, modify or reject a run, so the review line tells the truth.
- [Follow what agents do and what they cost](/xtm-one/administer/observability/): the model cost behind **AI cost**, and the exports.
