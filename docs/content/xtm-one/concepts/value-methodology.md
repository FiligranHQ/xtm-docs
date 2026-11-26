---
title: How XTM One computes the value of agents' work
description: The model behind the Value page of XTM One - the four formulas, where each input comes from, what a figure counts, who sees which figures, and where each default is changed.
doc_type: explanation
products: [xtm-one]
personas: [executive, soc-manager, admin]
owner: docs-xtm-one
last_reviewed: 2026-11-16
---

Before you quote the value of your AI agents to a committee, you need to know where each number comes from. XTM One computes every figure of the **Value** page from the runs it recorded and from a catalog of minutes and hourly rates. No figure comes from a language model, and reading one never starts a run. This page explains the model, the label each input carries, what is counted and what is not, and how to replace the catalog's defaults with your own numbers. In XTM One, the same explanation is the **How value is computed** page.

::screenshot{id=xtm-one-value-methodology alt="The How value is computed page: the four formulas of The model, each followed by a worked example in numbers, then the inputs from Units to Setup cost, with the index of the sections on the left."}

## Where to find it

Every user opens it, on any edition, from three places:

- **Read the methodology**, at the bottom of every **How this is computed** popover: on the [Value page](/xtm-one/use/value/), and on the **Value** tab of an agent that counts a unit of work, where its **ROI** section comes before its run budget.
- **How value is computed**, on the value card of the home dashboard, while no value is recorded yet.
- The address `/value/methodology`. **Back to Value** returns to the Value page.

The page is static text: it shows the model, not your organization's numbers. Your numbers are in **How this is computed**, beside each figure of the Value page.

## The model

Four formulas give the figures of the Value page:

| Figure | Formula | Example from the page |
| --- | --- | --- |
| **Value** | units x (manual minutes - review minutes) x hourly rate | 10 units x (30 - 5) min x 60 per hour = 250 |
| **Net value** | value - AI cost | 250 - 40 = 210 |
| **ROI** | net value / AI cost | 210 / 40 = 5.25 |
| **Payback** | setup cost / net value per month | 420 / 210 per month = 2 months |

Each formula reads these inputs:

| Input | What it is |
| --- | --- |
| **Units** | Counted from the runs. Each agent, assignment or fleet declares the unit its output counts, such as a threat report triaged. An orchestrator declares none, so the work of a fleet is never counted twice. |
| **Manual minutes** | The time a person needs without the agent, as a low, likely and high range. The range shown under every figure comes from the low and the high minutes. |
| **Review minutes** | The time a person spends checking the output: measured where the run records it, the catalog's value otherwise. |
| **Hourly rate** | The loaded hourly rate of the role that would do the work. |
| **AI cost** | What was spent: the measured cost of each model call in the period. A call counts once, for the person it is attributed to, its agent and that agent's fleet, and is never shared out over other work. Calls of automations, of chat and every other call count. |
| **Setup cost** | Zero unless declared. **Payback** reads "Under a month" when one month of net value covers it. |

## Where each input comes from

Every input carries one label and its source. You read them in **How this is computed**, in the CSV export and on the last page of the PDF value report.

| Label | What it means | Its source |
| --- | --- | --- |
| **Default** | Shipped with the catalog | The publication or the reasoning |
| **Published estimate** | Taken from a public study | The publication |
| **Assumption** | A reasoned figure, until a study exists | The reasoning |
| **Measured** | Read from the platform: the AI cost, a recorded review time | The period measured |
| **Customized** | Changed by an administrator | Who changed it, when, and the default value |

Before you present a figure, read the labels of its inputs. A figure built on **Assumption** and **Default** inputs is an estimate. Once your administrator sets your own minutes and rates, they read **Customized**, and the figure describes your organization.

## What is counted

Three sources make the total, and every breakdown adds up to it:

| Source | What it counts |
| --- | --- |
| **Automations** | The runs and the work items, valued with the formulas above. |
| **Chat (estimated)** | The decks, reports, videos and documents an agent finished in a conversation, counted on the low side: the low manual time, minus the time spent in the chat plus 25%. The **From chat, estimated** card of the Value page shows the same figure, line by line. |
| **Other AI usage** | The cost of every other model call. It adds cost and no value. |

**How this is computed** then reads "Value = the sum of its sources, chat included", followed by each source and its amount. When showing one source would single out a small group, only the total shows, so a hidden source is never found by subtraction.

- Each piece of work and each model call counts for one person: the person a run acted for, the person a work item works for, the person of the conversation. Work with no person counts for the organization, its agent and its fleet, and in nobody's own figures.
- A rejected outcome counts no unit.
- A run nobody reviewed is counted, and shown apart. The review rate says how much of the figure a person confirmed.
- A unit without manual minutes counts its AI cost and no time saved.
- The period is 7, 30 or 90 days, this quarter or all time. The trend is weekly.
- Older history that can't be tied to a person isn't presented as complete. The figures then read "since" a date: the first day from which every piece of work carries its person.

Each deliverable made in a conversation is listed only to the person who made it and to administrators: see [Measure what your agents' work is worth](/xtm-one/use/value/).

## Who sees what

- **Mine** is what you did: your own runs, chats and AI cost, on any agent you use and in the general chat. It never includes a colleague's work on an agent you can see.
- An agent or a fleet shows the work of everyone who used it, in aggregate, to whoever can open it.
- The organization total is the same figure for an administrator and for every user, as long as **Organization totals for every user** is on. Agents and fleets you can't open are summed in **Other agents and fleets**.
- Nobody is identifiable. A user never sees another person's name, email or deliverable titles. For a user, a fleet, agent, unit of work or week backed by fewer people than the threshold is summed into an **Other** row. The threshold is 5 people by default. Your own figures and the organization total are never folded, and an administrator sees every row. The exports and the agents' tools apply the same rule.
- The breakdown by person is for administrators only. It is off by default, and every read of it is written to the audit log.

An administrator sets the three rules in :menu[Settings > Organization > Rates by role], on the **Value visibility and privacy** card, with **Edit**:

| Setting | Default | Effect |
| --- | --- | --- |
| **Organization totals for every user** | On | On, every user may switch the Value page to the organization's totals. Off, only administrators see them, and every user still sees their own. |
| **Smallest group shown apart** | 5 people | A whole number from 2 to 50. Smaller groups are folded into **Other** for users who aren't administrators. |
| **By person breakdown for administrators** | Off | On, administrators see the organization's value per person. |

Every change to these settings is audited.

When the deployment hides cost data, **AI cost**, **Net value**, **ROI** and **Payback** read **Hidden**, on screen and in the exports, never 0. See [When costs are hidden](/xtm-one/use/value/#when-costs-are-hidden).

## How to change a default

| Default | Where an administrator changes it | What a change moves |
| --- | --- | --- |
| **Units of work** | :menu[Settings > Organization > Units of work] | The value of every past and future run of that unit |
| **Rates by role** | :menu[Settings > Organization > Rates by role] | The value of every unit valued at that role |
| **Outcomes** | On an agent or a fleet | What its runs count from now on |

For an administrator, **Units of work** and **Rates by role** link to their settings; anyone else reads "Your administrator changes these defaults." A customized value survives a reseed or an upgrade, and **Reset to default** removes it. Every change is audited.

Changing a minute or a rate moves the past too: the figure of last quarter changes when you change the rate today. Note the figures you presented before you change a default, or present them again with the new one.

## Exports

- **CSV**: the figures, then the methodology line and every assumption with its label and source.
- **PDF**: the value report, then a last page with the formula, the report's own numbers and the assumptions.

Both are on the **Outcomes** tab of [Observability](/xtm-one/administer/observability/), for administrators.

:::example[Check a figure before the committee quotes it]
Hélène Marchetti, CISO of Grand Meridian Hotels, prepares the quarterly security committee. On the Value page, she opens **How this is computed** beside **Value**, then **Read the methodology**.

She reads the example of the first formula: 10 units x (30 - 5) min x 60 per hour = 250. Ten reports triaged, each sparing 25 minutes of an analyst paid 60 an hour, are worth 250. With 40 of AI cost, the net value is 210, and the ROI 5.25. A setup cost of 420 is paid back in 2 months.

Back in her own popover, the analyst's rate reads **Published estimate** and the manual minutes **Assumption**. She asks her administrator to set Grand Meridian's loaded rate in **Rates by role**: it then reads **Customized**, and she presents the figure as her organization's, not as an estimate.
:::

## What's next

- [Measure what your agents' work is worth](/xtm-one/use/value/): read the figures for your agents, fleets and units of work.
- [Review what agents did](/xtm-one/use/work-history/): accept, modify or reject a run, so the review rate tells the truth.
- [Follow what agents do and what they cost](/xtm-one/administer/observability/): the model cost behind **AI cost**, and the exports.
