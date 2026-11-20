---
title: Measure what your agents' work is worth
description: Read the Value page of XTM One for the hours your AI agents saved, what that time is worth once the AI cost is paid, the return and the payback, by fleet, agent or unit of work, and export them.
doc_type: how-to
products: [xtm-one]
personas: [executive, soc-manager, admin]
owner: docs-xtm-one
last_reviewed: 2026-11-16
edition: ee
---

The **Value** page answers the question a committee asks about AI agents: what is their work worth? It shows the hours of work your agents saved people, and what those hours are worth at the rate of the role that would have done them. It subtracts the AI cost, then gives the net value, the return on investment (ROI) and the payback. You read it for what you did or for the whole organization, by fleet, by agent or by unit of work, and export it as a CSV file or a PDF report. Every figure opens on how it is computed, so you can defend it.

::screenshot{id=xtm-one-value-page alt="The Value page of the whole organization over the last 30 days, with Export CSV and Export PDF next to the period: the cards from Hours saved to Payback, then the By agent tab, where Velvet Harbor watcher shows its runs, units, hours saved, value and net value."}

## Before you start

- **The Enterprise Edition**: on the Community Edition, the page says "Value is part of Enterprise Edition." with the ways to upgrade. See [What each edition of XTM One brings](/xtm-one/administer/editions/).
- **Every user** opens it: from :menu[Work > Value], from **See the value** on the **Value** card of the [home dashboard](/xtm-one/use/dashboard/), or from **Open Value** in [Observability](/xtm-one/administer/observability/). It stays open when AI is switched off: reading it never starts a run nor calls a model.
- **Only declared work counts**: an agent's runs count once it declares a unit of work, such as a threat report triaged, and the minutes a person would spend on it. Decks, reports, videos and documents an agent finishes in a chat count on their own. See [Create an agent](/xtm-one/build/agents/) and [Run agents as a team with fleets](/xtm-one/build/fleets/).

## Read the value of your agents

:::steps
1. Open :menu[Work > Value].
2. Pick the scope, next to the line that names it:
   - **Mine** ("What you did"): the work that counts for you, on any agent and in the general chat. A run counts for the person it ran as, otherwise the owner of its assignment, otherwise the owner of its agent. A work item counts for the person it works for, and a deliverable finished in a chat for the person who asked for it. The AI cost is the cost of your own model calls.
   - **Whole organization**: everyone's work, the same totals for every reader. An administrator opens the page on it. Anyone else gets it while **Organization totals for every user** is on, which is the default; they open the page on **Mine**.
3. Pick the period: **Last 7 days**, **Last 30 days** (the default), **Last 90 days**, **This quarter** or **All time**.
4. Read the cards. Click the info button of a figure, **How this is computed**, to see its formula with this period's numbers.
5. Pick a tab, **By fleet**, **By agent** or **By unit of work**, to see where the value comes from.
6. To take the figures away, click **Export CSV** or **Export PDF**, next to the period.
:::

The scope, the period, the tab, the search and the sort stay in the address: a colleague who opens your link gets the same view, within what they may read.

## What the figures say

| Card | What it says |
| --- | --- |
| **Hours saved** | The time people did not spend producing the units and the chat deliverables, review time deducted |
| **Value** | What those hours are worth, at the hourly rate of the role that would do the work |
| **AI cost** | The cost of every model call in view: agents' runs, chats and any other AI usage |
| **Net value** | The value minus the AI cost |
| **ROI** | The net value divided by the AI cost, as a multiple: "5 x" means the net value is five times what the AI cost |
| **Payback** | How long the net value takes to pay back the setup cost. With no setup cost, it reads **Under a month**, with "No setup cost" under it |

Under a figure, a range such as "96 to 170 h" goes from a low to a high estimate: the catalog gives each unit a low, likely and high number of manual minutes. A figure that reads "-" has a **Why there is no figure** tip: "Not enough data in this period to compute it." The ROI has none when no model cost was recorded, and the payback none when the net value is not positive.

**How this is computed** opens on the period, the formula with its numbers, and the **Inputs**. The value is the sum of its sources, such as "Automations $327 + Chat (estimated) $120 = $447": **Automations** for the runs and work items of agents, **Chat (estimated)** for the deliverables of the [From chat, estimated](#count-what-agents-finished-in-a-chat) card. A line then says what was counted, such as "Counted: 13 units from 13 runs and 4 chat deliverables". Each input carries its origin: **Default**, **Customized**, **Assumption**, **Published estimate** or **Measured**, with its source. It also says what was counted and what was not. **Read the methodology** opens the full model: see [How XTM One computes the value of agents' work](/xtm-one/concepts/value-methodology/).

While some minutes or rates are still the catalog's, a note says "Estimated: some minutes or rates are catalog defaults. Set your own to replace them." **Units and rates** opens the units of work in Settings, for an administrator. Present the figures as an estimate until your own minutes and rates replace the defaults.

Under the cards:

- **The review line** says how much of the work a person checked: "Reviewed: 3 of 12 runs (25%)". The runs include the work items that count units. Review is optional: a run nobody reviewed counts like an accepted one. **Review the rest**, while some runs wait, opens the runs in :menu[Work > Operations]. On the **Whole organization** view, someone who is not an administrator gets no link: "Totals of the whole organization; you can open only your own runs." A view with no run to review, such as a **Mine** that holds only chat deliverables, has no review line.
- **Value over time** draws the value and the net value per week, with the range shaded ("Range: low to high estimate"). Under two weeks of data, it says "Not enough history yet".
- **The cost note** says what the AI cost covers: every model call in view, the same figure as LLM Analytics, general chat included. **By agent** shows the cost no agent carries in its **No agent (general chat)** row. When some of the cost is tied to no unit of work, the note gives its amount, which has its own row under **By unit of work**.

## See where the value comes from

The three tabs list the same figures, the highest value first: **Name**, **Runs**, **Units**, **Hours saved**, **Value**, **AI cost**, **Net value** and **Reviewed**. Every column sorts, and the search finds a row by name (**Search fleets...**, **Search agents...**, **Search units of work...**). **Columns** hides the **Runs**, **Units** or **AI cost** column. Click a fleet's row to open its page. Click an agent's row to open the **Value** tab of its page, which shows its ROI and its run budget: see [Create an agent](/xtm-one/build/agents/).

| Tab | One row per | Use it to |
| --- | --- | --- |
| **By fleet** | Fleet | Compare the teams of agents |
| **By agent** | Agent | Find the agent that delivers the most, or one that costs more than it saves |
| **By unit of work** | Kind of work, such as a threat report triaged | See which work the agents take off people's hands |

The rows add up to the cards. What no row of the tab holds is summed in a last row, with a tip that says what it is:

- **Agents outside a fleet**, under **By fleet**: the runs of the agents that belong to no fleet.
- **No agent (general chat)**, under **By agent**: the value and the cost no agent carries, such as the general chat.
- **No unit of work**, under **By unit of work**: the value and the cost tied to no unit of work. The cost note gives its cost.
- **Other agents and fleets**, under **By fleet** and **By agent**: the agents and fleets you cannot open, or that too few people used to be shown apart.
- **Other units of work**, under **By unit of work**: the units of work too few people used to be shown apart.

## What other users see of the organization

An administrator reads every row, and so does anyone on **Mine**. Someone who is not an administrator, on the **Whole organization** view, reads the same totals, but not every row:

- the agents and fleets they cannot open are summed in **Other agents and fleets**;
- a fleet, an agent or a unit of work that fewer people used in the period than **Smallest group shown apart**, five by default, is folded into its **Other** row, and so is a source in **How this is computed**. Another row may be folded with it, so that no figure can be found by subtraction;
- a week of **Value over time** with too few people is merged with the next one;
- the **From chat, estimated** card counts the deliverables by kind, without the table: "Each deliverable is listed only to the person who made it and to administrators."

An administrator sets **Organization totals for every user** and **Smallest group shown apart** on the **Value visibility and privacy** card of [Settings](/xtm-one/administer/settings/#organization), under **Rates by role**. See also [How XTM One computes the value of agents' work](/xtm-one/concepts/value-methodology/).

## Count what agents finished in a chat

The **From chat, estimated** card counts the decks, reports, videos and documents an agent finished in a conversation. Its figures are part of the cards above: they are the **Chat (estimated)** line of **How this is computed**. A deliverable is counted once it is used, or 7 days after its last edit; until then it is **Waiting for use**. Its hours saved are the low estimate of the time by hand, minus the time spent in the conversation plus 25%.

- The card says how many deliverables are counted and how many wait. Its table lists each **Deliverable**, its **Agent**, the **Person** who asked for it, its **Status** (**Waiting for use**, **Counted** or **Corrected**), its **Hours saved** and **Value**. Filter it by **Kind** (**Deck**, **Report**, **Video**, **Document**) and **Status**, or search it.
- A line shows only to the person who asked for the deliverable and to administrators. On **Mine**, the table lists your own deliverables.
- **Adjust**, on a counted line of your own (any line, for an administrator), opens **Adjust the hours saved**. Type the **Hours saved**, between 0 and 720, then **Save**: your figure replaces the estimate for everyone who reads it, and nothing waits on it. **Back to the estimate** removes your figure.

## When costs are hidden

When the deployment hides cost data, every view, for every reader including administrators, shows three cards: **Hours saved**, **Value** and **AI cost**. The tables read **Hidden** in **AI cost** and **Net value**, the exports too. The cost reads **Hidden**, captioned "Costs hidden on this deployment", never 0: the value is shown before the AI cost. An administrator hides costs at install, with `OBSERVABILITY_COST_DATA_ENABLED`: see the [configuration reference](/reference/xtm-one/configuration/#advanced-tuning).

## When the page is empty

- **"Nothing was counted in this period."**: no run, no chat deliverable and no value in the period. **Widen the window** moves to the next longer period. A view that holds only chat deliverables is not empty: it shows its cards.
- **"No agent is measured yet."**, on **All time**: no agent or fleet declares a unit of work yet. An administrator gets **Pick a unit on an agent**, which opens the agents. Anyone else reads "Your administrator or an agent's owner declares what an agent produces."
- **"The value report could not be read."**: click **Retry**.

## Export the figures

**Export CSV** and **Export PDF**, next to the period, download the view you read: its scope and its period, with the figures you see, folded rows and hidden costs included. Every user exports **Mine**; the **Whole organization** exports for administrators, and for anyone else while **Organization totals for every user** is on.

- The CSV file holds a total line, then every row of every breakdown, each summing to the total: **By source**, **By agent**, **By fleet**, **By unit of work** and **By week**, then the inputs the figures rest on. It names two residual rows its own way: **No fleet** and **No unit**.
- The PDF value report shows the hours saved, value, AI cost, net value and ROI, the first eight rows of each breakdown but the weeks, how the figures are computed and their assumptions. When the platform cannot render it, export the CSV instead.
- When an administrator has turned on **By person breakdown for administrators**, an administrator's export adds **By person**, and each such read is written to the audit log.

The file is named after the view and the period, such as `xtm-one-value-90d.pdf` for the organization, or `xtm-one-value-mine-90d.csv`. An export that fails says "The export failed." The [Observability](/xtm-one/administer/observability/) **Outcomes** tab exports the organization's figures too.

:::example[Bring the agents' value to the quarterly committee]
On Monday 13 April 2026, Hélène Marchetti, CISO of Grand Meridian Hotels and XTM One administrator, prepares the quarterly security committee. She opens :menu[Work > Value]: the page opens on **Whole organization**. She picks **Last 90 days**.

On **By agent**, **Velvet Harbor watcher** comes first. Its assignment **Weekly Velvet Harbor brief** ran 13 times, and each run counts one **Threat report triaged for relevance**: 13 units x (20 - 3) minutes at $88.73 an hour, about $327. On the **Value** card, **How this is computed** shows it on the **Automations** line. Its **Inputs** mark the 20 minutes as an **Assumption** and the analyst's rate as a **Published estimate**.

The page also says "Estimated: some minutes or rates are catalog defaults." Hélène follows **Units and rates** to Settings and, in **Rates by role**, gives the CTI analyst Grand Meridian's own loaded rate. Every figure of the page follows. She clicks **Export PDF** and puts the value report in the committee pack, quoting **Hours saved** and **Net value** with the range under each.
:::

## What's next

- [How XTM One computes the value of agents' work](/xtm-one/concepts/value-methodology/): the model, where each input comes from and what is counted.
- [Review what agents did](/xtm-one/use/work-history/): accept, modify or reject a run, so the review line tells the truth.
- [Follow what agents do and what they cost](/xtm-one/administer/observability/): the model cost behind **AI cost**, and the exports.
