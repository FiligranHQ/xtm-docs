---
title: Review what Platform Health finds and proposes for your agents
description: Approve or reject the improvements Platform Health proposes for your agents, and read the cycles it ran and the signals that started them.
doc_type: how-to
products: [xtm-one]
personas: [admin, ai-builder]
owner: docs-xtm-one
last_reviewed: 2026-11-19
edition: ee
---

Platform Health is the supervisor built into XTM One: a fleet of AI agents that consolidates memories, diagnoses the agents and flows their managers opted in, measures a variant on sandbox copies, and proposes the change that earned it. Its page, :menu[Administration > Platform Health], is where an administrator reviews that work: the proposals to approve or reject on **Improvements**, what each run found and spent on **Cycles**, and the measures that started a run on **Signals**.

::screenshot{id=xtm-one-administer-platform-health alt="The Platform Health page under Administration: its title with the About Platform Health info button, Platform Health settings at the top right, and the Improvements, Cycles and Signals tabs, Improvements open."}

## How Platform Health works

1. An administrator turns Platform Health on, and the manager of an agent or a flow opts it in.
2. A cycle starts: on a schedule, when a signal crosses its threshold, or on demand from the agent's page.
3. Its specialists read the target's runs, ratings and memories, and record each finding once, with its evidence.
4. The Experimenter runs the current configuration and a variant as sandbox copies, on the same production cases, and measures both.
5. The change becomes a proposal, with its diff and its measured result. Nothing changes until a person who manages the target approves it.
6. After an approval, Platform Health watches the target, and rolls the change back when it regresses.

## Before you start

- XTM One runs the Enterprise Edition with AI switched on. Otherwise the menu entry is marked unavailable: the page still opens on past work, but no new cycle starts.
- Your account is an administrator. The page lists every proposal, cycle and signal of the platform. A person who manages an agent decides its proposals from the agent's page: see [let Platform Health improve an agent](/xtm-one/build/agents/#let-platform-health-improve-it).
- **Enable Platform Health** is on, in the [Platform Health settings](/xtm-one/administer/settings/#agents-and-automations). It is off by default; the **Platform Health settings** button, at the top right of the page, leads there, and **Open Platform Health** in the settings leads back.
- The agents and flows to supervise are opted in by their managers, and inside the **Supervision scope** of the settings. Until then, the three tabs stay empty.

An administrator who turns on **Show the companion** in their [profile](/xtm-one/use/profile/) also sees the open findings of Platform Health in the companion's bubble, where **Open Platform Health** leads to this page.

## Decide a proposal

:::steps

1. Open :menu[Administration > Platform Health]. **Improvements** opens first, newest proposals first, 25 per page, with the columns **When**, **Type**, **Verdict**, **Status** and **Summary**.
2. Narrow the list: **Search proposals...** searches the summaries, and the **Type** and **Status** filters keep the kinds you want. **Export CSV** downloads the rows of the page shown.
3. Look for the status **Awaiting your decision**: you may decide those proposals. A proposal that another person decides shows **Awaiting approval**.
4. Click the row. It opens in place, with the full summary, the diff of the change, what was **Measured**, and the **Evidence** the finding rests on. **Open the target** opens the agent or the flow.
5. Click **Approve** or **Reject**. The message "Proposal approved" or "Proposal rejected" confirms it; "The decision was not recorded" says why it failed.

:::

An opened proposal shows:

- The diff of the change: lines added start with `+`, lines removed with `-`. A change without text says "This change has no text diff."
- The writes the experiment intercepted, when it ran a flow: "3 writes were simulated during the experiment."
- **Measured**: the figures of the experiment, as it filed them. Once the change is applied and watched, **Re-evaluation** adds the score before and after, with the difference, such as `71.0 → 78.5 (+7.5)`.
- **Evidence**: the traces, ratings, runs, flow runs and evaluation runs the finding comes from.

The **Type** says what the change is: **Persona** or **Skill** of an agent, **Assignment prompt**, **Flow repair**, **Flow optimization**, **New flow**, or **Agent to flow**, an assignment replaced by a flow. The **Verdict** is the result of the experiment: **Measured gain**, **No measured change**, **Measured regression**, or **Unmeasured**. An unmeasured proposal says why, and never reads as a gain.

## What happens after your decision

- **Approve** writes the change as a new version of the target, recorded in the [audit logs](/xtm-one/administer/audit-logs/), such as "Applied an approved persona rewrite to agent 'Velvet Harbor watcher' as v4". The status turns **Watching**.
- For 7 days by default, Platform Health measures the target again. A regression rolls the change back as a new version: **Rolled back**. When a person edited the target in the meantime, their edit stays: **Kept after a human edit**.
- A watch that ends without regression gives **Kept**. When nothing could be measured, the status is **Failed** and the change stays: roll it back from the target's version history if you need to.
- A proposal whose target changed since it was filed cannot be approved, for example "The persona changed since this rewrite was proposed. Reject it and ask for a new one."
- **Reject** leaves the target as it is, and is recorded in the audit logs. Platform Health does not propose a rejected change of an agent again.
- An **Agent to flow** proposal first runs **In shadow**, beside the assignment it replaces, then is watched like the others.

The watch goes on to its end even after Platform Health or AI is turned off.

## Who decides a proposal

- Administrators read every proposal of the platform.
- On a company-managed agent or flow, administrators decide.
- On a private one, its owner decides, and the people it is shared with at **Can configure** or **Full access (incl. chats)**. An administrator sees **Awaiting approval** there, and cannot decide.

## Read a cycle

**Cycles** lists the 50 newest cycles. Each row gives the status, the specialist that ran it, the trigger and the date:

- Status: **In progress**, **Completed**, **Partly completed**, **Budget reached**, **Failed** or **Cancelled**.
- Specialist: Platform Health itself, **Memory Steward**, **Quality Analyst**, **Flow Lab** or **Experimenter**.
- Trigger: **Schedule**, **Signal** with the signal's kind, or **On demand**.

Click a row to read its digest: **Findings**, **Experiments**, **Proposals** ("1 filed · 0 applied · 0 rolled back"), **Consolidations** for the memory work, **Budget spent**, **Writes logged in dry-run, not executed**, **Skipped or not completed**, and the specialist's report. **Open the target** opens the agent or the flow. A cycle still open says "The cycle is still running; its digest is written when it ends."

- A target has one open cycle at a time. A cycle still open after 24 hours closes as **Failed**.
- The schedules run in UTC: memory every day at 02:00, quality on Mondays at 03:00, flows on Mondays at 04:00, and a weekly digest on Mondays at 07:00.
- By default, a cycle spends at most 500,000 tokens or $5, and Platform Health at most 10,000,000 tokens or $100 a month. A cycle that reaches its cap ends as **Budget reached**.
- In dry run, Platform Health files no proposal: the digest counts the writes it would have made.

## Read the signals

**Signals** lists the signals crossed now, with the columns **Signal**, **Target**, **Reading**, **Cycle** and **Fired**. **Reading** gives the baseline, then the value now; a feedback score reads on 100, such as 82 → 64. **Cycle** says **Started**, or why no cycle started. **Search...**, the **Signal**, **Target** and **Cycle** filters, and **Export CSV** work as on **Improvements**. Click a row to open the target.

| Signal | Crosses when | Checked |
| --- | --- | --- |
| **Feedback drop** | The rating falls by 10 points over the last 3 days against the 3 days before, with at least 10 ratings | Every 15 minutes |
| **Failure spike** | More than 5 % of the last 20 runs failed | Every 15 minutes |
| **Cost drift** | The cost rose by 20 % or more over 7 days against the 7 before, with at least 10 runs | Every hour |
| **Model change** | The model of the target changed | Every hour |

These are the default thresholds: an administrator changes them, or turns a signal off, in the [Platform Health settings](/xtm-one/administer/settings/#agents-and-automations). Signals come from figures only, with no call to a model. A signal starts one cycle when it crosses: staying crossed starts nothing more, and clearing it re-arms it. A start refused for the budget, the license, or because Platform Health is off is tried again at the next pass. A start refused because a cycle is already open is not.

The reasons **Cycle** may give:

- "Platform Health is not enabled."
- "Platform Health's fleet is not set up yet."
- "Platform Health's assignment for this work is paused."
- "This target is outside Platform Health's supervision scope."
- "Platform Health is already working on this target."
- "Platform Health's budget per cycle is reached." or "Platform Health's monthly budget is reached."
- "Platform Health could not start this work."

The **Name** setting renames Platform Health in these messages and in the page's content. The menu entry and the page's title keep Platform Health.

:::xtm-one[Ask Platform Health from the chat]
The built-in tool suite **Platform Health** lets an agent list the signals and the cycles, read a cycle's digest, and request a cycle. In an administrator's chat, it acts as Platform Health: every cycle, and the signals of the whole scope. Anyone else reaches only the agents and flows they manage. Ask "which signals are crossed on Velvet Harbor watcher?" or "start a cycle on Velvet Harbor watcher".
:::

:::example[Follow the improvement of the Velvet Harbor watcher]
In May 2026, Hélène Marchetti, CISO of Grand Meridian Hotels and administrator of XTM One, turned on **Enable Platform Health**. Sophie Laurent opted her agent "Velvet Harbor watcher" in, and clicked **Improve with Platform Health** on its page. The agent is private, shared with the group **CTI team** at **Read only**.

1. A few days later, Hélène opens :menu[Administration > Platform Health] and clicks **Cycles**. The top row is **Completed**, Platform Health, **On demand**.
2. She clicks the row. Its digest gives **Proposals** "1 filed · 0 applied · 0 rolled back" and **Budget spent** "212,480 tokens · $1.64".
3. On **Improvements**, the new row is **Persona**, **Measured gain**, **Awaiting approval**. The diff adds a line that asks for the indicators of each report, in a table.
4. Hélène cannot decide: the agent is Sophie's. Sophie opens the agent's page, and approves the proposal from its **Improvements** card.

The status turns **Watching**, then **Kept** 7 days later, when no regression shows.
:::

## What's next

- [Set up the platform in Settings](/xtm-one/administer/settings/#agents-and-automations)
- [Create an agent](/xtm-one/build/agents/#let-platform-health-improve-it)
- [Read Platform Health's fleet](/xtm-one/build/fleets/#read-platform-health-s-fleet)
- [Audit logs](/xtm-one/administer/audit-logs/)
