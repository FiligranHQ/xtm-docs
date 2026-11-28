---
title: Get answers about the platform's own activity
description: Ask an XTM One agent with the Platform Insights tool what AI costs, who uses which agent, what was created and who changed what, and read its charts and warnings right.
doc_type: how-to
products: [xtm-one]
personas: [admin, executive, auditor]
owner: docs-xtm-one
last_reviewed: 2026-11-08
edition: ee
migrated_from:
  - xtm-one:/usage/platform-insights/
---

**Platform Insights** is a built-in tool that lets an AI agent answer questions about XTM One itself: what the AI costs and for whom, which agents are used, what was created and when, who changed what. Ask in plain words, such as "graph the cost of AI agents for the CTI team over the last 90 days" or "who deleted an agent last week?", and the agent answers with figures, a chart drawn in the conversation, and the warnings that come with them. It reads the same figures as the administration pages (**Model usage** under **Observability**, **Objects** and **Audit logs**), so what it says matches what they show.

::diagram{id=xtm-one-platform-insights-scope}

## Before you start

- The agent you ask carries **Platform Insights**, chosen in the **Built-in Tools** card of its configuration. The General Assistant carries it by default for administrators, and for group managers, who get their team's figures; an administrator can remove it in :menu[Administration > Settings > AI and models > General Assistant].
- What the agent reads depends on who asks:

  | Who asks | Reads |
  | --- | --- |
  | An administrator | The whole platform: every user, group, agent and object |
  | A group manager | Also the usage of the current members of the groups they manage, administrators excepted, and those groups |
  | Anyone else | Their own slice: the objects they own, the usage they generated, their groups, the audit entries of their own actions. A question about another person or group is refused |

  Every answer says which of the three it served.

Every tool of Platform Insights only reads: none asks for approval. Reading the audit log is itself recorded in the audit log, so looking at who did what leaves a trace of who looked.

## Ask a question

:::steps
1. Open :menu[Chat] and choose the General Assistant, or an agent that carries **Platform Insights**.
2. Ask about usage, cost, objects or changes, with the period and the breakdown you want: per department, group, person, agent, model, day, week, month or release.
3. Read the answer: the figures, the chart when one helps, and the warnings under them.
4. Ask for the CSV when you need every row: the agent attaches the file to its answer.

::screenshot{id=xtm-one-platform-insights-chart alt="Crisis briefer answering Hélène Marchetti with Platform Insights: a stacked bar chart of the agents, assignments, conversations, presentations, workspaces and knowledge bases created in January and in April 2026, with the window under it."}
:::

What you can ask about:

| Question | What the agent reads |
| --- | --- |
| Cost, tokens, errors, active users, latency, feedback | The usage of the models: as a total or per department, group, person, agent, model, source, day, week, month or release, split by a second dimension, compared with the previous period, or projected |
| What the platform holds | Agents, assignments, runs, conversations, presentations, files, workspaces, work items and more, listed or counted per day, week, month or release |
| The traces and sessions | The rows of the **Traces** and **Sessions** pages: when, which agent, model and user, status, duration, cost. A row says what a trace is, never what it holds |
| Who changed what | The audit log, filtered by actor, action, object, period and text |
| The releases | The builds the platform has run, and when each was first and last seen |
| An overview | Objects per type, agents by state, assignments by trigger, integrations, storage, the deployed build |

A chart is a bar, stacked bar, line, area, pie or donut chart, with at most 16 categories and 6 series: a 90-day window is charted per week or per month. When you export the conversation, or read it in Slack or Teams, each chart is written as a table.

## Read the figures right

The agent repeats the warnings that come with a figure, the same ones the dashboards show:

- **Departments never overlap; groups do.** A person counts for one department, so department rows add up to the total. A person in several groups counts in each, so group totals can be higher than the total.
- **Missing prices.** Usage of a model with no price has no cost, and cost totals leave it out. The answer names those models: an administrator sets their price in :menu[Administration > Settings > Usage and quotas > Model pricing].
- **Hidden costs.** When costs are hidden on the platform, every cost is reported as unavailable, never as zero, and no chart is drawn of it.
- **Coverage.** A period that starts before the oldest data the platform holds is flagged: the missing days are missing data, not zero activity. When the daily aggregation is more than 48 hours late, recent days may still change.
- **Partial periods.** The first and last week or month of a period cover only the part inside it, and are flagged.
- **Unattributed usage.** Usage with nobody behind it (the platform's own calls, a run whose agent was deleted since) is reported apart, never dropped.

Usage with no user of its own (an assignment run, a consulted agent, a webhook, a channel message) counts for the person behind it: whoever triggered the run, else the identity it runs as, else the assignment's owner; for a consulted agent, the person of the conversation; for a webhook or a bot, the agent's owner.

Departments are set by an administrator, in :menu[Administration > Settings > Organization > Departments]: a person's department comes from their identity provider, else from the first group rule they match, else **Unassigned**. It is recorded when the usage happens: a person who changes team takes no past usage along.

## Plan a budget

- **Per head**: the number of active people, and the cost per active person, per trace and per 1,000 tokens.
- **Projection**: a monthly series projected up to 36 months ahead, with a linear trend, a compound monthly growth or the average of the last months. The fit uses complete months only, and says how well it reads the past.
- **Comparison**: any breakdown compared with the previous period of the same length.

Administrators can download the same rows as CSV from the API, at `/api/v1/insights/analytics/export`, for a spreadsheet or a data warehouse.

:::example[Prepare the AI budget of the security teams]
On 12 April 2026, Hélène Marchetti, CISO and XTM One administrator at Grand Meridian Hotels, prepares next year's AI budget. She opens :menu[Chat] with the General Assistant and asks:

> Graph the monthly cost of AI agents for the groups CTI team, SOC and Crisis cell since January 2026, project it to the end of 2027 with a linear trend, and give me the top five agents of the SOC.

The answer draws a stacked bar chart per month and group, projects the total to December 2027, and lists the SOC's agents, **Night shift triage** first. Under the chart, a warning says that a person in two groups counts in each, and that April 2026 is a partial month. Hélène asks for the CSV, then "who changed the run budget of Velvet Harbor watcher this month?": the audit log shows her own change of 9 April.
:::

## What's next

- [Set what one agent run may spend](/xtm-one/use/run-budgets/), to act on what an agent costs.
- [Govern how AI is used](/xtm-one/administer/ai-governance/), for the quotas and limits behind these figures.
- [Review what agents did](/xtm-one/use/work-history/), for the runs behind the traces.
