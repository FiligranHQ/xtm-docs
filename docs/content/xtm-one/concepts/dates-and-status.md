---
title: What the dates and statuses of an object mean
description: The difference between the date an object changed and the date work ran in XTM One, every status of a run, an assignment, a work item and a connected platform, and how to read them together.
doc_type: explanation
products: [xtm-one]
personas: [ai-builder, soc-manager, admin]
owner: docs-xtm-one
last_reviewed: 2026-11-08
migrated_from:
  - xtm-one:/usage/foundations/dates-and-status/
---

XTM One shows two kinds of dates, and confusing them is the most common misreading. **Record dates**, such as **Created** and **Updated**, say when someone changed an object's configuration. **Work dates**, such as a run's start and duration or a work item's **Last activity**, say when the work ran. An agent updated yesterday has not necessarily run since; a run started an hour ago with no end may be waiting for a person. Statuses complete the picture: a run is waiting, working or ended; an assignment is active or paused; a connected platform is reachable or not. This page lists each status, what it means and what to do about it.

::diagram{id=xtm-one-dates-and-status-run}

## Two kinds of dates

| Kind | Examples | Answers | Seen on |
| --- | --- | --- | --- |
| Record dates | **Created**, **Updated**, a version's date in **Versioning** | When was this configured, and by whom? | The **Overview** tab of agents and resources, :menu[Administration > Audit logs] |
| Work dates | A run's start, end and duration, the **Last activity** and due date of a work item | When did the work run, and is it still going? | :menu[Work > Operations], a run's page, an agent's **Activity** tab |
| Health dates | The last heartbeat of a connected platform, the last synchronization of a knowledge base | Is the connection or the content still fresh? | The suite switcher, the knowledge base's page |

On a configuration page, a date describes the object. On :menu[Work > Operations], it describes the work. On a connection, it describes freshness.

## Statuses of a run

A run is one execution of an assignment, of a background task or of a person's request. Its status shows on its card in :menu[Work > Operations], tab **Executions**, and on its page. The tab filters on **Running**, **Awaiting Input**, **Completed** and **Failed**.

::screenshot{id=xtm-one-dates-and-status-executions alt="The Executions tab of Operations for Sophie Laurent: the filters by kind, owner and status (All, Notifications, Digests, Running, Awaiting Input, Completed, Failed), and the run of Weekly Velvet Harbor brief, Completed on 12 April 2026 at 10:24, with the start of its summary and its 2 steps."}

| Status | Meaning | What to do |
| --- | --- | --- |
| **Pending** | Created and handed over, not yet picked up by a worker | Nothing; if it lasts, check that the worker runs |
| **Delayed** | Waits for the delay its assignment sets, or for a retry after a failure | Nothing |
| **Queued** | A webhook delivery that waits for the assignment's current run to end, so deliveries run in order | Nothing |
| **Running** | The agent is working | Open it to follow the tool calls |
| **Awaiting Input** | The agent asked a question, or asks approval for a tool call, and waits for a person. The output is not final | Answer or approve on the run's page; the bell of the top bar lists these runs |
| **Completed** | Finished. A person may also have dismissed its question, and the result then says that no action was taken | Read the result before acting on it outside XTM One |
| **Failed** | Did not finish. The run carries its cause, such as **Provider rate limited**, **Run budget exhausted** or **AI not licensed** | Read the cause; do not use a partial result |
| **Cancelled** | Stopped by a person or by an emergency stop. A cancelled run is never resumed | Start it again if the work is still needed |
| **Skipped** | Ended before it started, with the reason on the run, for example because its owner's account was deactivated. Nothing ran and nothing was charged | Fix the reason; the assignment itself is unchanged |

A background task delegated to another agent can also end **Incomplete**: it stopped before finishing, often because it waited too long for its own sub-tasks.

### Held runs

A run that comes due while AI cannot run, because an administrator switched AI off or the deployment runs the Community Edition, is **Held** instead of failing. It keeps its waiting status with a **Held** mark and the reason, is never charged, and starts by itself within about five minutes of AI being available again. A run held longer than `AI_UNAVAILABLE_HOLD_MAX_SECONDS` (one week by default) fails with the cause **AI disabled on the platform** or **AI not licensed**. See [Run XTM One with AI switched off](/xtm-one/administer/running-without-ai/).

## Statuses of an assignment

An assignment is **Active**, so its trigger fires, or **Paused**. The assignment's page says who paused it and when:

- **Paused** by a person, from its **Pause** button.
- **Paused: the owner's account is deactivated**: nothing starts it until the account is reactivated, then it resumes as it was.
- **Emergency stop**, a red badge instead of **Paused**: an emergency stop paused it, and only lifting the stop resumes it. See [Stop every agent at once](/xtm-one/administer/emergency-stop/).

A recent **Updated** date on an agent means its configuration changed, not that it ran. Its runs are in the agent's **Activity** tab and in :menu[Work > Operations].

## Statuses of a work item

A work item is a piece of work agents track across runs, in :menu[Work > Operations], tab **Work Items**. It moves through **Open**, **Claimed**, **In Progress**, **Blocked**, then **Done** or **Cancelled**. **Stale** is not a status but a warning XTM One computes when you look:

- the item is past its due date, or
- it is **Claimed**, **In Progress** or **Blocked** and nothing happened on it for 24 hours (`WORK_ITEM_STALE_AFTER_HOURS`).

An **Open** item nobody picked up is backlog, never stale for its age alone. The **Stale** view lists the items that need someone to look.

::screenshot{id=xtm-one-dates-and-status-work-items alt="The Work Items tab of Operations with its status filters from Claimed to Cancelled, and the All items or Stale switch."}

## Health of a connected platform

OpenCTI, OpenAEV and OpenCRQ register with XTM One again and again: each registration is a heartbeat. In the XTM platform switcher of the side menu, a platform whose last heartbeat is less than 15 minutes old shows **Connected**; otherwise it shows **No recent heartbeat**. A platform with no recent heartbeat is still registered: check that it runs and reaches XTM One. See [Connect the modules to each other](/deploy/configure/connect-modules/).

:::example[Monday morning in Operations]
On Monday 13 April 2026 at 09:30, Julien Bernard, SOC manager at Grand Meridian Hotels, opens :menu[Work > Operations]:

- The run of **Weekly Velvet Harbor brief** is **Completed**: it started at 08:00 and took 2 minutes. The **Updated** date of **Velvet Harbor watcher** is 2 April: Sophie Laurent last changed the agent then, and it has run every Monday since.
- A run of **Night shift triage** is **Awaiting Input** since 03:12: the agent asks Paul Garnier's approval before creating a Jira ticket. Its output is not final until someone answers.
- The work item "Check the booking platform logs for the Velvet Harbor loader" is **In Progress** and flagged **Stale**: nobody touched it since Saturday.
:::

## What's next

- [Review what agents did](/xtm-one/use/work-history/): the run pages.
- [How XTM One marks what to trust and what to review](/xtm-one/concepts/trust/)
- [Give agents recurring work](/xtm-one/use/work-items/)
