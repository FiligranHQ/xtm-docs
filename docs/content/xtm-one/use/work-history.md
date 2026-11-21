---
title: Review what agents did
description: Follow the runs of XTM One agents in Operations, answer a run that waits for you, read and rate a result, and follow delegated work, partners' tasks and the work items agents queue.
doc_type: how-to
products: [xtm-one]
personas: [ai-builder, soc-manager, auditor]
owner: docs-xtm-one
last_reviewed: 2026-11-19
migrated_from:
  - xtm-one:/usage/work-history/
---

**Operations** is the record of what AI agents did on their own: every run an assignment started on a schedule, a webhook or an event, every task an agent handed to the background, the work items agents queue for each other and the state they share. Open it to check whether a task still runs, read a result before acting on it, answer an agent that waits for your decision, or understand why a run failed. It is a read surface over work already done, so it keeps working when AI is switched off.

::diagram{id=xtm-one-work-history-operations}

## Before you start

You see the runs of the agents you can see: your own agents, the agents shared with you and company-managed agents. Deleting runs and the administrator's clean-up depend on your rights.

## Find a run

:::steps
1. Open :menu[Work > Operations]. The page opens on the **Executions** tab.
2. Choose whose runs you see: **Mine**, **Company**, **Shared** or **All**. Each segment shows how many runs it holds, and the tab opens on the first one that holds a run. When **Mine** is empty while others are not, **Show all** moves to every run you can see.
3. Narrow the list. **Running**, first on the toolbar, keeps the runs under way in one click. **Status** next to it takes one or more of **Running**, **Awaiting input**, **Completed**, **Failed** and **Cancelled**, each with how many runs it holds under your other filters; **Clear** empties it. Type in **Search runs...**, or open **Filters** for the run type (**All** or **Assignments**), the view, the fleet and the agent. The badge on **Filters** counts the filters in use.
4. Read the runs. In **Grid view**, where the tab opens, each row gives the run's title, usually its assignment, its status, one line with its result, its question or its error, its agent, and when it last changed. Hover the title for the whole record: the assignment, the agent that delegated the run, the tokens it used, and when it was created, started and ended. **List view** shows a table of **Run**, **Status**, **Agent**, **Started** and **Duration**, newest first: click a column header to sort, and choose the columns shown under **Columns**. In grid view, **Sort by** sorts the rows.

::screenshot{id=xtm-one-work-history-executions alt="The Executions tab: the Running chip and the Status filter, the Search runs field, the Mine, Company, Shared and All segments with their counts, and the Filters button. Below, the row of Weekly Velvet Harbor brief, Completed, with the start of its summary and its agent, Velvet Harbor watcher."}
:::

**Rows per page** shows 10, 25, 50 or 100 runs at a time, 25 by default, and keeps your choice. Click a run to open it. The view under **Filters** offers **All**, **Notifications** and **Digests**: **Notifications** and **Digests** keep the runs that sent updates or digests apart from the other tasks, and hide **Status** while chosen. A run cut by the [emergency stop](/xtm-one/administer/emergency-stop/) says **Stopped by an emergency stop**.

The fleet filter shows when you can open a fleet. With a fleet chosen, the list holds the assignment runs of the fleet's own members, and **ROI review** set to **Unreviewed** keeps the runs whose outcome nobody reviewed yet. The fleet and the agent you choose stay selected when you move to another tab of **Operations**. Fleets are described in [Run agents as a team with fleets](/xtm-one/build/fleets/).

The other tabs of **Operations** hold what agents coordinate through:

| Tab | What it lists |
| --- | --- |
| **Background Tasks** | The work agents delegated to the background, from chats and runs |
| **Work Items** | The durable items agents queue for each other, and the actions people scheduled for later |
| **Shared State** | The values agents keep across runs |
| **Worksets** | The corpora agents build to reason over more material than fits in one answer |
| **Workspaces** | The places that group conversations by topic: see [Share a workspace with your team](/xtm-one/use/workspaces/) |

**Background Tasks** has the same toolbar as **Executions**: **Running**, then **Status** with **Pending**, **Running**, **Completed**, **Incomplete**, **Failed** and **Cancelled**, then **Search tasks...**. **Filters** holds the fleet and the execution: **No live execution** keeps the tasks still marked in flight that no execution runs. **List view** shows **Task**, **Status**, **Delegated by**, **Delegated to**, **Created** and **Updated**. Click a task to open the run it belongs to; a task delegated from a chat has no page. On its row, **Kill task** stops a pending or running task and **Delete** removes it; once your own chat task is finished, rate it with a thumbs up or down. Select tasks to delete several at once.

**Shared State** opens on its **Active** entries. **Status** on the toolbar switches to **Stale**, **Resolved** or **All**, each with its count. Search with **Search key or label...**, and filter under **Filters** by **Owner**, the agents that wrote the entries, and by fleet. **List view** shows **Key**, **Label**, **Status**, **Owner**, **Size** in characters and **Updated**, newest first. Click an entry to unfold its value and its **Notes**. Hover its title in the grid view for its **Contents**, its **Size** with the length of its longest list, its latest note and its dates. On its row, **Resolve** closes an active entry and **Delete** removes it; only administrators manage the company-managed ones. **Export CSV** downloads the rows as filtered and sorted.

An entry is **Stale** when its value has not changed for 14 days, notes aside, or its end date has passed; a banner counts the active ones, and **Show stale** lists them. **Oversized** marks an entry over 32,768 characters: the agent that writes it is warned at each write. A write is refused when it takes the value over 65,536 characters, or a list in it over 500 items and longer than it was, unless it makes the value smaller. Ask its owner agent to bound its longest list or archive its old items: an archive that would go past these limits drops the oldest items instead of failing, and archived items are kept 14 days.

Every hour, XTM One tidies shared state without calling a model. It resolves the active entries whose work item is done, cancelled or deleted, and those past their end date. It shortens an entry over 65,536 characters or 500 list items by dropping its oldest list items. Each of these leaves a note on the entry. For each fleet whose entries are oversized or stale, it files one low-priority work item for the fleet's orchestrator, titled "Shared-state hygiene" with the counts; no other is filed while that one stays open.

**Worksets** is a table you read as a list or as cards (**List view**, **Grid view**). Search it with **Search worksets...**, filter it under **Filters** by **Status** (**Active**, **Resolved**), by **Scope** (**This run**, **This flow**, **Global**) and by fleet, and sort it by name, items, size or last update. In the list view, **Columns** picks the columns shown. **Export CSV** downloads the rows as filtered and sorted. Click a workset to unfold its coverage and its items. An active workset says how its collection stands: **Still collecting** while more items may come, **Rolling** for the corpus XTM One keeps open for an agent's chat or assignment, and **Sealed** once closed.

## Answer a run that waits for you

Some runs pause to ask a person: an approval before an action, a clarification. Such a run shows under **Awaiting input**, its row saying **Approval needed** or the question. Click **Answer** on the row: the question or the pending action unfolds under it, and you answer it there, or **Discard** it when it is no longer needed. The run's page offers the same. This is where the approvals and questions of autonomous work reach you in XTM One; the ones asked in Slack and Teams are in [Approve agent actions from Slack and Teams](/xtm-one/administer/chat-approvals/).

Someone who cannot answer a run can ask the people who can, from the card of its link in Microsoft Teams: its owner, or the administrators when the run is company-managed or its owner is deactivated. They get a notification, and the run's page lists the requests under **Requests to act on this run**. Answering or discarding the run closes them; **Dismiss** closes one request and leaves the run waiting.

## Read a result

Click a run to open its page: its title, its state, its agent, the trigger that started it, when it was created, started and completed, and links to its agent, its assignment or its parent run. A finished run shows, depending on what it did, its final summary, the actions of its tools, its steps, what it sent to another system, and the runs it started by delegating. Under **Generated Files**, the files it delivered show as cards; its working files, such as scratch files, stay folded under one line.

A file whose content was deleted keeps its place, greyed out, with its name, its size and why it went: "Deleted on Mar 12, 2026 by the retention policy", "Deleted on Mar 12, 2026 by an administrator", or "Deleted on Mar 12, 2026 with what it belonged to". It no longer opens nor downloads. An administrator sets how long agents' files are kept, under **Storage & retention** in [Settings](/xtm-one/administer/settings/); [Find the files agents created and received](/xtm-one/use/files/) lists them.

The page says how the run ended:

- **Partial result**, next to the run's state: the run completed but stopped before its agent finished, because it used all its **Max iterations**, reached its timeout or its budget, the model did not answer in time, or tasks it delegated were still running. A notice in the **Result** card says which.
- **Interrupted**: the platform restarted or stopped the process running it. **Details** shows the platform's message.
- **Cancelled**: the run was cancelled, with the reason, such as who cancelled it.
- **Error**: the run failed, with its cause.

A run that met obstacles, such as tool errors or retries, has a **Frictions** card: what it struggled with, how many times, and what carries each fix, or **Undisposed** while nothing does. **Evidence** unfolds what was recorded.

Once a run has completed or failed, the **Result** card asks **Was this result useful?** with a thumbs up and a thumbs down. A thumbs down lets you say what went wrong; when administrators have defined feedback criteria, you can also score each one. Rating again replaces your rating. The people who manage the agent read every rating in its **Feedback** tab.

When the run counts a unit of work for its agent, its assignment or its fleet, the **ROI outcome** row under the result records what a person made of it: **Accepted**, **Modified** or **Rejected**, with an optional note in **Note (optional)**. **Modified** asks for the **Review time**, the minutes spent checking or fixing the result, then **Save review**. Click the chosen outcome again to clear it. Until someone reviews it, "Unreviewed: counted as delivered in ROI until someone reviews it." The row shows to administrators, to the run's owner unless the run is company-managed, and to those who read the fleet's ROI. The figures it feeds are in [Follow what agents do and what they cost](/xtm-one/administer/observability/).

## Follow delegated work

One run can start others when its agent hands work to another agent. In the list, such a run counts its delegation steps on its row, and a delegated run reads **Delegation** when it has no assignment. The run's page links its child runs and its parent: follow them to see where the work started, which agent took it over, and which parts succeeded or failed. How agents hand work to each other is drawn in [Follow how agents hand work to each other](/xtm-one/use/agentic-flow/).

## Follow a task a partner's agent sent

An agent outside the platform, at a partner or on another XTM One, can send a task to one of your agents through an agent-to-agent (A2A) integration: see [Let a partner's agents work with yours over A2A](/xtm-one/build/integrations/#let-a-partner-s-agents-work-with-yours-over-a2a). Each task runs as a run of your agent. Its page shows where the task came from in its overview: **Source** reads **A2A**, with the **Client ID** the partner presented and the **A2A Provider**, the kind of integration the task came through, such as **OAuth A2A**.

Your agent can ask the partner's agent a question before it goes on. Its run then ends, and the task waits for the answer:

- The run reads **Waiting on caller**, in the list and on its page, rather than **Completed**. Under **Status**, it counts among **Completed**.
- Its page shows the card **Waiting on its caller**, with the question under **The agent asked**. The **Result** card says "No result yet: the task is waiting for its caller to answer the question above."
- Nobody answers it from XTM One: the agent that sent the task answers, and the page updates on its own when it does.

The answer continues the task in a new run of the same agent. The run that asked then shows **Asked its caller** with the badge **Answered**, and **View the run that continued it**. The new run shows the card **Continues the task after its caller answered**, with **The agent asked** and **The caller answered**, and **View the run that asked**. When the partner cancels the task before answering, or the task outlives its maximum lifetime, four hours by default, the run that asked shows **Cancelled before an answer**. A linked run you cannot open is not linked.

## Read a work item

Open the **Work Items** tab; **About work items**, the "i" on its toolbar, says what a work item is. Like **Executions**, the tab filters by **Mine**, **Company**, **Shared** or **All**, and searches with **Search title or key...**.

**Status**, on the toolbar, opens on the open work: **Open**, **Claimed**, **In progress** and **Blocked**. Pick any of these and **Done** or **Cancelled**, each with its count; **Open work** goes back to the open ones and **All** shows every status. Hover **Blocked** for how many items wait on other items and how many need an action. **Filters** holds:

- the view: **All items**, **Waiting for me**, **Stale** or **Scheduled**;
- the fleet, and the owner agent (**Any owner agent**).

**List view** shows **Title**, **Status**, **Priority**, **Owner**, **Due**, **Created** and **Updated**, newest first. With only **Done** or **Cancelled** chosen, the items sort by when they ended, until you sort a column. **Export CSV** downloads the rows as filtered and sorted.

**Waiting for me** gathers the decisions agents ask of you, and your own items that no agent owns and are past due. **Stale** lists the items that need a look, each badge saying why: **Past due**, **Never released** (it waits for a deployment the platform does not watch), **Reference quiet** (its pull request, issue or deployment has seen no activity for days) or **Quiet for a day** (taken or blocked, then quiet for over a day). **Scheduled** lists the actions people scheduled for later.

Each row gives the item's title, its status, its priority, its owner agent (the one expected to act on it, or "nobody"), and what it waits for, else its summary. Hover the title for the rest: the agent that created it, the chat or run it came from, the person it acts for, the agents its completion reaches (**Impacts**), the run or agent that holds it (**Held by**), its key and its dates. An agent, chat or run you cannot open is shown without its name. A badge says what will act on it:

| Badge | Meaning |
| --- | --- |
| **Picked up automatically** | An assignment or flow triggered by work items takes it within about a minute. **Picked up once due** is the same, for an item with a due date to come |
| **No automatic pickup** | Nothing takes the owner agent's items: the item waits until someone asks that agent to process its work items, or completes it by hand |
| **Not picked up: filtered out** | The owner agent has an inbox, but its minimum priority or prefix leaves this item out |
| **Unassigned: nothing picks it up** | No owner agent is set |
| **Blocked on other items** | It waits for the items listed under **Waits for**, and goes back to the queue by itself once every one is done. A cancelled one keeps it blocked |
| **Waiting for CI** | The CI of its pull request is running: when it completes, the platform hands the item to its owner agent at once |
| **Being worked on**, **Blocked: not picked up** | An agent holds the item, or it was blocked, for example after repeated failed attempts. Nothing picks up a blocked item until someone puts it back in the queue |

A blocked item reads **Waiting** while items it waits for are unfinished: it reopens by itself. It reads **Blocked** when someone has to act: an item it waited for was cancelled, a restart cut the run that held it, or someone blocked it by hand. Hover the chip for the items, or the reason. A blocked item that does not say what it waits for carries **No reason**; a red count on the toolbar says how many, and lists them when you click it.

An item whose pull request waits in its repository's merge queue on GitHub stays parked until the queue merges it: no run starts only to find the merge still pending. When the queue drops the pull request without merging it, a note at the top of the item's **Details** says so and why. The item goes back to the queue for its owner agent to fix the pull request, or stays blocked while other items it waits for are unfinished.

Click a row, or **View details** in its menu, for the whole item: its **Details**, its **Result**, its payload and its timeline, who created it and where, and its dates. **Files** lists the files its creator attached to it and those an agent added while working on it or completing it, each with its type, its size and **Download**; a line counts the files no longer stored. **Who acts on it** says what acts on it next, the agents its completion reaches, what holds it, its pull requests and what it waits for. When the item counts a unit of work, its **Result** carries the same **ROI outcome** row as a run.

An item that waits for a person's decision is **Blocked** from the moment an agent asks, so its owner agent does not take it before someone answers; under what it waits for, its row reads **a person's decision**. When an agent waits for your decision, the item opens on **Waiting for your decision**: the question, one button per option with what it does, the option the agent suggests marked **Recommended**, and the deadline, if any. Click an option, or write in **Or answer in your own words** and click **Send answer**: the item goes back to the queue, and its owner agent takes it with your answer. Only an administrator answers a company-managed item.

On an unfinished item, **Ask the agent** opens a chat with its owner agent, the message ready for you to complete and send:

| Button | What the agent does |
| --- | --- |
| **Explain** | Says where the item stands and what it waits for |
| **Re-check** | Checks live whether the wait is over, and closes the item if it is |
| **It's solved** | Checks that the problem is gone, then closes the item or says what is missing |
| **Do it** | Acts on the item now and records what it did |

The row's menu offers **Ask the agent to re-check** and **Tell the agent it's solved** as well. The hand actions say what they do to the owner agent, and **Mark done** and **Cancel work item** ask for confirmation and let you record why:

- **Mark done** closes the item: its owner agent does not run it again, and its completion still reaches the agents it impacts. It is not offered while the item waits for unfinished items. When the problem is gone but the agent should check, ask it with **It's solved** instead.
- **Cancel work item** closes it without completion work: nothing reaches the agents it impacts.
- **Put back in the queue** reopens a claimed, blocked or finished item: its owner agent runs it again at its next pickup, and any wait it had is cleared.

Work items and shared state written by a company-managed agent are company-managed: everyone who can use that agent sees them, and only company-managed agents and administrators change them.

## When an assignment stops on its own

- **A missing resource**: when an integration reports the same resource missing in two runs (a Notion database deleted, or no longer shared with the integration), the run fails with the cause **Integration resource missing**, and the assignment's owner is notified once. After a few failed scheduled runs in a row (three by default), the assignment pauses. Share the resource with the integration again, or change the assignment, then click **Resume** on the assignment's page.
- **A deactivated owner**: while an administrator has disabled the account that owns an assignment, nothing starts it, unless it is company-managed. It resumes as it was when the account is enabled again. To keep it running meanwhile, an administrator transfers its agent to an active owner or makes it company-managed.

## Clean up

**Delete this run**, on a run's row, deletes that run. To clear test runs or old records, select runs and click **Delete**, or select every run matching the current filters; flow runs cannot be selected. When the selection holds runs awaiting approval, you can keep only those and approve or reject them together. Administrators have two more buttons, icons whose name shows on hover:

- **Cleanup Stale Runs**, at the end of the filter row, marks as failed the runs a restart or a timeout left showing as running, so the history is accurate again.
- **Cancel runs not started**, next to the agent picker in **Filters**, cancels every pending, delayed or queued run of the agent chosen there; it stays greyed out until you choose one. XTM One counts the runs and asks to confirm with **Cancel runs**, or **Keep them**. Running runs are not touched; the cancelled ones stay in the history, and **Status** switches to **Cancelled** to show them.

With a fleet chosen, neither button shows, and you cannot select every run matching the filters.

:::example[Check the Monday brief, and answer the agent that waits]
On Monday 13 April 2026 at 08:20, Camille Roux, CTI analyst at Grand Meridian Hotels, checks the morning run of the **Weekly Velvet Harbor brief**, an assignment of **Velvet Harbor watcher**. She opens :menu[Work > Operations], keeps **Executions**, chooses **Completed** under **Status**, and picks **Velvet Harbor watcher** in the agent filter. The run of 08:00 shows the trigger **Scheduled**, the summary of two new reports on Velvet Harbor, and the message it posted to the CTI team. She rates it with a thumbs up.

Under **Awaiting input**, a second run of the same agent waits: it asks whether to add the indicators of the new report to the watch list. Camille opens it, approves, and the run goes on. On the **Work Items** tab, the item **Check the Madrid booking platform** that **Night shift triage** created for **Velvet Harbor watcher** shows **No automatic pickup**: Camille asks the agent in the chat to process its work items.
:::

## What's next

- [Follow how agents hand work to each other](/xtm-one/use/agentic-flow/), for the live view of the same activity.
- [Let an agent act later, at a time you choose](/xtm-one/use/work-items/), for the scheduled items of the **Work Items** tab.
- [Find the files agents created and received](/xtm-one/use/files/), for the outputs of runs.
