---
title: Follow what agents do and what they cost
description: Read Model usage, Outcomes, Traces and Sessions in XTM One to see what the AI costs and what the agents' work is worth, and to follow a failing or costly agent's run step by step.
doc_type: how-to
products: [xtm-one]
personas: [admin, soc-manager, ai-builder]
owner: docs-xtm-one
last_reviewed: 2026-11-09
---

XTM One records what every AI agent does: each answer in a chat, each assignment run, each piece of work one agent hands to another is a trace, with every model call and tool call inside it. :menu[Administration > Observability] reads that activity in four tabs. **Model usage** says what the AI costs, for which agents, people and models, how fast it answers and how often it fails. **Outcomes** says what the agents' work is worth: the hours it saved and its value once the model cost is paid. **Traces** lists the traces one by one, to find the run that went wrong, and opens each one step by step. **Sessions** groups them by conversation. This page reads the four tabs, then follows a costly agent from the dashboard to the call that explains it. What a trace keeps and for how long is set in [Settings](/xtm-one/administer/settings/#usage-and-quotas).

::screenshot{id=xtm-one-observability-analytics alt="The Model usage tab over the last 30 days: the six key figures with their change against the previous 30 days, then the Usage over time chart and the Top agents ranking that lists the agents by cost."}

## Before you start

- **The four tabs** are for administrators. A user who is neither an administrator nor a group manager does not see the **Observability** row, and its addresses take them back home.
- **A group manager** sees one row, **Model usage**, narrowed to the current members of the groups they manage, administrators excepted. They cannot open **Outcomes**, **Traces**, **Sessions** or a team member's trace. See [Group managers](/xtm-one/concepts/access/#group-managers).
- **An agent's owner**, and the people a personal agent is shared with as **Full access (incl. chats)**, read that agent's traces on its **Observability** tab and open each trace from there. Nobody but an administrator reads the traces of a company-managed agent.
- **Traces exist** while **Enable local trace storage** is on. You set it in :menu[Administration > Settings], tab **Usage and quotas**, section **Observability**, with **Configure**. With **Store full payloads** off, a trace keeps timings, tokens, cost, model, status and failure cause, but no prompt, answer or tool result. Traces are deleted after **Trace retention (days)**, 90 by default.
- **Costs** need a price for each model, set in the same tab, section **Model pricing**. When the deployment sets `OBSERVABILITY_COST_DATA_ENABLED=false`, every cost shows as `--$`: see the [configuration reference](/reference/xtm-one/configuration/#advanced-tuning).
- **Outcomes** counts only the agents and fleets that declare a unit of work: see [Count what an agent produces](/xtm-one/build/agents/#count-what-an-agent-produces).

## Read Model usage

Open :menu[Administration > Observability], tab **Model usage**. Everything on the tab answers for the window and the filters at the top.

- **The window**: **24H**, **7D**, **30D** (the default), **90D**, **6M**, **1Y** or **Custom** dates. A window longer than 7 days is read from daily totals that XTM One keeps after the traces are deleted, so **6M** and **1Y** still answer with a 90-day retention.
- **The filters**: **All groups**, **All agents**, **All models** and **All sources** narrow every card at once. A group means its current members: a person who changes group takes their past usage along. **Clear all filters** resets them, with the count of filters in use. The window and the filters stay in the address, so you can send a view to a colleague.
- **Refresh Analytics**, for administrators, rebuilds the missing daily totals of the last 30 complete days in the background. "Analytics refresh started" confirms it, and you are notified when it ends. Use it when the tab says "Stats may be incomplete", with the age of the last build: the daily totals have not been built for more than 48 hours.

### The key figures

Each figure carries a badge with its change against the previous window of the same length (**vs previous 30 days**). On cost, latency and errors, a rise shows as bad news.

| Figure | What it measures | What to do with it |
| --- | --- | --- |
| **Total Cost** | What the model calls cost, in US dollars, with the cost per trace under it | A rise with the same number of traces means heavier runs: look at **Top agents** by **Cost / trace** |
| **Traces** | How many pieces of agent work ran | Follows adoption. A sudden rise often comes from one assignment that runs too often |
| **Total Tokens** | The tokens sent to and received from the models | Explains a cost on a model without a price |
| **Avg Latency** | How long a trace takes from start to end, on average | Read **Latency** for the slow tail, which an average hides |
| **Error Rate** | The share of traces that failed | Open **Top errors**, then the failed traces |
| **Avg Feedback (0-1)** | The average rating people gave the answers, from 0 to 1 | A drop after a change of prompt or model points at that change |

**N traces with unknown cost**, under the figures, counts the traces of a model without a price: every cost total leaves them out. Add the price in **Model Pricing**, then **Reprice $0 traces** to fill their cost.

### The cards

- **Usage over time**: the cost per model, or the traces per source, by **Day**, **Week** or **Month**. **Stacked** draws the six largest series and groups the others as **Other**; **Total** draws one bar. Click a series under the chart to hide it.
- **Top agents**, **Top models** and **Top users** rank by **Cost**, **Traces**, **Tokens**, **Cost / trace** or **Avg latency**, over the **Top 5**, **Top 10** or **Top 25**. A group manager reads **Top team members**. Click an agent to open its page, when you may open it. A deleted agent carries a **Deleted** chip, a disabled one **Disabled**. To read the whole tab for one agent, pick it in **All agents**.
- **Traces by source**: where the work came from. **Agent Chat** is a conversation with an agent, **General Chat** with the General Assistant, **Assignment** a scheduled or triggered run; the other sources gather the platform's own model calls. Click a source to filter the page on it.
- **Latency**: **Percentiles** (P50 to P99 over the window), **Daily median**, or **By model** (the median and P95 of the ten busiest models). The headline reads "Median 4.2s, 95% under 12s": the second number is what the slowest answers cost your users.
- **Cost breakdown**: one table, **By agent**, **By user**, **By group** or **By model**. Click a row to open the agent, or, for an administrator, the person or the group.
  - **By agent** shows each agent's owner, or a **Company** badge for a company-managed agent. Its **Status** filter keeps the **Active**, **Disabled** or **Deleted** agents.
  - **By user** shows each person's groups and how many agents they used, with a **Groups** filter.
  - **By group** shows the active users out of the members, and the average cost per active user.
  - **By model** shows input and output tokens and cost, P50 and P95.

  Runs with no account behind them (an anonymous channel sender, a public widget visitor, the global MCP key) show apart as **No user account**. A person in several groups counts in each, so the groups can add up to more than the total. When a view cannot be read, the card says "Could not load this breakdown." with **Retry**.
- **Top errors**: the failed traces of the window, grouped by their message (its first 200 characters), agent and model, with the **Count** and when each was **Last Seen**. Its **Agent** and **Model** filters narrow the list.

The tables sort on every column and search. They show 10, 25, 50 or 100 rows a page, hide columns with **Columns**, and **Clear all filters** resets their filters. Your browser keeps each table's page size and hidden columns for your next visit. In a narrow window, a few columns step aside (**Agents** and **Avg cost / trace** of **By user**, **Input Cost**, **Output Cost** and **P50** of **By model**): **Columns** brings them back. Large figures read short, such as $61.2k. **Export CSV** downloads every row that matches the search, with the exact figures.

## Read what the agents' work is worth in Outcomes

Open :menu[Administration > Observability], tab **Outcomes**. It says what the agents and fleets delivered over the window: the hours of work they saved people, and what that time is worth once the model calls are paid. It counts every agent or fleet that declares a unit of work, such as a threat report triaged or a detection rule written. Reading it never starts a run nor calls a model.

::screenshot{id=xtm-one-observability-outcomes alt="The Outcomes tab over the last 30 days: the Hours saved, Net value and Review rate cards, then the table of the agents and fleets that count a unit of work, with Velvet Harbor watcher and its runs, units, hours saved, cost and net value."}

- **The window**: **7D**, **30D** (the default), **90D** or **All**. **Export CSV** downloads the table, **Export PDF** a one-page value report.
- **Open Value** opens the Value page at the organization's scope. Every user reads there the value of the agents and fleets they can open.
- **An estimate**: while some minutes or rates are still the catalog's defaults, a note says "Estimated: some minutes or rates are catalog defaults. Set your own to replace them." **Units and rates** opens the place to set them, in [Settings](/xtm-one/administer/settings/#organization).

| Card | What it measures |
| --- | --- |
| **Hours saved** | The time people did not spend producing the units, review time deducted. Under it, the runs counted, the work items when a fleet counts them, and the chat deliverables when agents finished decks, reports or documents in conversations. A range from a low to a high estimate shows when the units carry one |
| **Net value** | What those hours are worth at the hourly rate of the role that would do the work, minus the model cost. Under it, the cost, or "no priced model or no run cost recorded" when the runs cost nothing |
| **Review rate** | The share of records a person reviewed, such as "Reviewed 3 of 12 records (optional)". Review is optional: a run nobody reviewed still counts |

When the deployment hides costs, every cost and net value reads **Hidden**.

The table lists the fleets and agents, the largest net value first: **Name**, **Type** (**Fleet** or **Agent**, with a **Type** filter), **Runs**, **Units**, **Hours saved**, **Cost**, **Net value** and **Reviewed**. Click a fleet's row to open the fleet's page, an agent's row to open the agent's **Value** tab, with its return on investment (ROI) and its run budget. Search it with **Search fleets or agents...**, and hide the columns you do not need with **Columns**.

The table ends with summed rows, so that it adds up to the cards; hover the icon next to each name for its meaning:

- **No agent (general chat)**: the value and cost no agent carries, such as the General Assistant's chats.
- **Other agents and fleets**: the agents and fleets you cannot open, or used by too few people to be shown apart.

Under the table, two lists say where each figure comes from, with this window's numbers:

- **How this is computed**: one line per source, such as "Automations: 12 h = $1,065", with **Automations**, **Chat (estimated)**, **Other AI usage** and **Other sources**. The lines add up to the total, which turns the minutes into hours, and the value minus the model cost into the net value. Then come the records reviewed, and the rejected runs, which count no time saved, only their cost.
- **Assumptions**: the manual and review minutes of each unit and the rate of each role, each marked as an assumption, a published estimate or your organization's own figure. "No baseline for" names the units without minutes: their cost counts, but no time saved.

With nothing to show, the tab says why:

- **On 7D, 30D or 90D**: "Nothing was counted in this period." **Widen the window** moves to the next longer window.
- **On All**: "No agent is measured yet." Pick a unit of work on an agent or a fleet: **Open agents** takes you to the agents.

For example, the Velvet Harbor watcher counts one **Threat report triaged for relevance** per run. The catalog puts it at 20 minutes of manual work and 3 of review, at the analyst's rate of $88.73 an hour. Each weekly brief saves 17 minutes, worth about $25 before its model cost.

## Find a trace in Traces

Open :menu[Administration > Observability], tab **Traces**. The newest traces come first, 50 a page.

::screenshot{id=xtm-one-observability-traces alt="The Traces list: the search and the Filters pill in the toolbar with the total at its end, and the traces of Night shift triage, Crisis briefer, Velvet Harbor watcher and the CTEM agents, each with its timestamp, name, agent, user, model, duration and tokens."}

- **Filters**: in the table's toolbar, **Search trace name...**, then the **Filters** pill, which opens **Status** (**OK** or **Error**), **Time range** (**From** and **To**), a switch per provider type under **Sources** (`openai`, `anthropic`, `azure_openai`, `ollama`...), **Groups**, **Agents**, **Models** and **Failure cause**. The pill shows how many filters are in use, and **Clear all filters** resets them and keeps the sort. The total at the end of the toolbar then reads "n of N": the traces that match out of every trace you may read.
- **Columns**: **Timestamp**, **Name**, **Agent**, **User**, **Model** (**Platform default** when the agent uses the platform's default model), **Duration**, **Tokens (in/out)**, **Cost**, **Status** and **Score**, a check when someone rated the answer. Every column but **Agent**, **User** and **Score** sorts. **User**, **Tokens (in/out)** and **Score** step aside in a narrow window, and **Columns** hides the ones you do not need. The list shows 10, 25, 50 or 100 traces a page.
- **A failed trace** names its cause in a tooltip on its status.
- **View Traces**, on an assignment or one of its runs, opens this list scoped to it: the scope shows as a chip after **Scoped to:**, which you remove to see every trace again.

Click a row to open the trace in a side panel, and **Open full page** for its own page.

### Failure causes

The **Failure cause** filter groups the causes by who the failure points at. The option reads "Provider · Provider rate limited".

| Group | Examples | Who acts |
| --- | --- | --- |
| **Configuration** | Agent missing or disabled, AI disabled, Integration resource missing, Pipeline tool refused or failed, Platform quota exceeded | You: the platform's setup stopped the run before or around the model call |
| **Request** | Context overflow, Request too large, Model declined to answer | The agent's builder: shorten what the agent sends, or pick a model with a larger context |
| **Provider** | Provider rate limited, Provider overloaded, Provider rejected credentials, Provider timed out, All providers failed | You or the provider: check the API key, the provider's limits, or add a fallback provider |
| **Run control** | Run budget exhausted, Cancelled | The run stopped as asked: raise the [run budget](/xtm-one/use/run-budgets/) if the work is legitimate |
| **Orchestration** | Never claimed by a worker, Attempts exhausted, Deadline exceeded | The platform engineer: the workers did not take or finish the work. See [Monitor and troubleshoot XTM One](/deploy/operate/xtm-one/) |
| **Unattributed** | Unattributed, No cause recorded | Read the trace's error message |

## Follow one trace step by step

A trace's page, `/traces/` followed by its identifier, shows the trace whole.

::screenshot{id=xtm-one-observability-trace-detail alt="The trace of Night shift triage testing a Sigma rule: the header with its latency, agent, provider, cost and tokens, the tree of its iterations and tool calls on the left, and the code interpreter call selected on the right with its arguments and output."}

- **The header**: the trace's name, **Trace ID** and status, then badges: **Latency**, the **Session** (a link to the conversation) or the **Run** it belongs to, the **Agent**, the **Provider** and the **Provider config** that answered (marked `failover` when a fallback provider took over, `downgraded` when the run's budget moved it to another model), the cost, the tokens ("1.2K prompt to 350 completion", the exact count on hover), the model, and the platform actions it counted against the [quotas](/xtm-one/administer/settings/#usage-and-quotas).
- **Session focus**, under the header, lists the other traces of the same conversation: click one to move to it. The **Conversation KPIs** then add up the whole conversation: traces and errors, tokens, cost, wall time and latency.
- **The left pane** draws the trace's steps as a **Tree** (each iteration of the agent, the model calls, the tool calls inside), a **Timeline**, a **Search** through the names, inputs, outputs and arguments, or a **Graph**. A failed iteration is marked.
- **The right pane** inspects the step you select: its type and level, latency, model, cost and tokens, then **Tool Arguments**, **Input**, **Output** and **Reasoning**, and the files it used or produced. A large payload shows a preview with **Load full payload**. On a failed trace, **Agent View** names the **Failing iteration** and the **Trace Error**: the cause first, then the provider's message.

## Read a whole conversation in Sessions

Open :menu[Administration > Observability], tab **Sessions**, to start from a conversation rather than a trace. Each row is a conversation, named by its title in the **Conversation** column ("Untitled conversation" without one), with its **User**, **Last Activity**, **Traces**, **Agents**, **Models**, **Tokens**, **Cost** and **Errors**. It filters like **Traces**: **Search conversation, model, agent...**, then the **Filters** pill with **Status**, **Time range**, **Sources**, **Groups**, **Agents** and **Models**, and **Clear all filters**. Click a row to open its first trace with **Session focus** on, and **Back to Sessions** to return.

::screenshot{id=xtm-one-observability-sessions alt="The Sessions tab of Observability: one row per conversation, with its user, last activity, traces, agents, models, tokens, cost and errors."}

Use it when a person reports "the agent went wrong somewhere in our conversation": the **Errors** column finds the conversation, and **Session focus** walks through its turns.

:::example[Find why the SOC's AI bill went up in April]
On Monday 13 April 2026, Hélène Marchetti, CISO of Grand Meridian Hotels, sees **Total Cost** at $412 on **30D**, up 38 % **vs previous 30 days**, while **Traces** only rose 6 %. Heavier runs, not more of them.

1. In **Top agents**, she switches the metric to **Cost / trace**: **Crisis briefer** comes first at $1.90 a trace, ahead of **Night shift triage** at $0.31.
2. She picks **Crisis briefer** in **All agents**: every card now reads that agent alone. **Usage over time**, **By model**, shows the jump on 2 April. That day, Sophie Laurent moved the agent from the platform default to a larger model for the crisis cell's decks.
3. **Top errors** lists 14 failures of **Night shift triage**, all "Provider rate limited", last seen at 02:04. She opens the **Traces** tab and sets, in **Filters**:

   | Filter | Value |
   | --- | --- |
   | **Agents** | Night shift triage |
   | **Status** | **Error** |
   | **Failure cause** | Provider · Provider rate limited |
   | **From** | 6 April 2026 |

4. She opens the trace of 02:04: **Agent View** names the failing iteration, after a burst of five parallel calls during Paul Garnier's alert on FD-PAR-03.

She asks Sophie to keep the larger model for the decks only, sets a [run budget](/xtm-one/use/run-budgets/) on **Crisis briefer**, and adds a fallback provider for the night. The next week, **Cost / trace** of **Crisis briefer** is back under $0.80, and **Top errors** shows no rate limit.
:::

:::xtm-one[Ask instead of reading]
An agent with the **Platform Insights** tool answers the same questions in the chat, from the same figures: "graph the cost of the SOC's agents over the last 90 days". A group manager gets their team's figures, as on this page. See [Get answers about the platform's own activity](/xtm-one/use/platform-insights/).
:::

## What's next

- [Set up the platform in Settings](/xtm-one/administer/settings/#usage-and-quotas), to choose what traces keep and for how long.
- [Count what an agent produces](/xtm-one/build/agents/#count-what-an-agent-produces), so that its work shows in **Outcomes**.
- [Set what one agent run may spend](/xtm-one/use/run-budgets/), to bound the runs this page shows as costly.
- [Review who changed what on the platform](/xtm-one/administer/audit-logs/), to find who changed the agent behind a rise.
- [Monitor and troubleshoot XTM One](/deploy/operate/xtm-one/), for the health of the platform itself.
