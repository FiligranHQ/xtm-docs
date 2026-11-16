---
title: How XTM One computes the value of agents' work
description: The model behind the Value page of XTM One - the four formulas, where each input comes from, what a figure counts, who sees which figures, and where each default is changed.
doc_type: explanation
products: [xtm-one]
personas: [executive, soc-manager, admin]
owner: docs-xtm-one
last_reviewed: 2026-11-10
---

Before you quote the value of your AI agents to a committee, you need to know where each number comes from. XTM One computes every figure of the **Value** page from the runs it recorded and from a catalog of minutes and hourly rates. No figure comes from a language model, and reading one never starts a run. This page explains the model, the label each input carries, what is counted and what is not, and how to replace the catalog's defaults with your own numbers. In the product, the same explanation is the **How value is computed** page.

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
| **AI cost** | The measured model usage of the counted runs in the period. |
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

- A rejected outcome counts no unit.
- A run nobody reviewed is counted, and shown apart. The review rate says how much of the figure a person confirmed.
- A unit without manual minutes counts its AI cost and no time saved.
- The period is 7, 30 or 90 days, this quarter or all time. The trend is weekly.

What an agent finishes in a conversation (a deck, a report) is counted on its own card of the Value page and never added to these figures: see [Count what agents finished in a chat](/xtm-one/use/value/#count-what-agents-finished-in-a-chat).

## Who sees what

You see the figures of the agents and fleets you can open. An administrator sees the whole organization, and decides whether every user also sees the organization totals. They do by default. The interface has no switch for it: an administrator sets it through the API, with `PUT /api/v1/roi/settings` and the body `{"roi_org_totals_visible": false}`, and the change is audited.

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
