---
title: Let an agent act later, at a time you choose
description: Ask any XTM One agent to post a message or run a task once, at a later time, then follow, cancel or reschedule that scheduled work item.
doc_type: how-to
products: [xtm-one]
personas: [cti-analyst, soc-manager]
owner: docs-xtm-one
last_reviewed: 2026-11-08
edition: ee
migrated_from:
  - xtm-one:/usage/scheduled-work-items/
---

Ask any AI agent to do something once, later: "post this summary in the CTI team's channel tomorrow at 9:00", "check the open incidents on Friday at 17:00 and send me a recap". The agent stores a **scheduled work item**, and XTM One runs it at that time by itself, as you. There is nothing to set up. Work that repeats, every Monday or on every new alert, is not a scheduled work item but an assignment of the agent, added with **Add Assignment** on the agent's page: see [Create an agent](/xtm-one/build/agents/) and [Start an agent from another tool with a webhook](/xtm-one/use/webhook-triggers/).

::diagram{id=xtm-one-work-items-lifecycle}

## Before you start

- You can chat with the agent: see [Get answers from an agent in the chat](/xtm-one/use/chat/).
- Your time zone is set in your profile, so that "9:00" means 9:00 where you are: see [Set up your profile and API keys](/xtm-one/use/profile/).
- For a message to post, the agent has the integration that posts it, connected and usable by you (a Slack or Microsoft Teams integration, for example).

Every agent can schedule, list, cancel and reschedule its own one-off actions, whatever tools it was given: these tools cannot be removed from an agent. Only an administrator can turn scheduling off, for the whole platform (see [When scheduling is off](#when-scheduling-is-off)).

## Schedule an action from a chat

:::steps
1. In a conversation with the agent, say what to do and when, with the day and the time: "post this in #soc-night tomorrow at 07:30".
2. Read the card the agent shows in the conversation: the time in your time zone and in UTC, the action and where it goes, and a preview of the content.
3. Check the content and the time. Click **Cancel** to drop it, or **Reschedule** to pick another time.

   ::screenshot{id=xtm-one-work-items-schedule-card alt="Paul Garnier asks for a reminder at 7:30 Paris time, and Night shift triage answers with the card of the action: Check FD-PAR-03 before the front desk opens, Scheduled on Monday 13 April 2026 at 05:30 UTC, an Instruction for the SOC morning shift with its text, and the Reschedule and Cancel buttons."}
:::

The agent stores the action in one of two forms. It picks the one that fits your request; the card says which.

| Form | What runs at that time | Fits |
| --- | --- | --- |
| **Exact** | One of the agent's tools, exactly as prepared when you asked: the message, its mentions and its target are sent as stored, with no model involved | Content that is final, such as a message you approved |
| **Instruction** | The agent runs the instruction in the background, with its tools, as if you asked it then | Work that depends on what is true at that time, such as "check the open incidents and post a summary" |

A time always carries its time zone. A time without one is refused rather than read as UTC. When the time an agent wrote is not your time zone on that date, for example across a daylight saving change, the agent is told, so it can correct it.

## What happens at the time

XTM One checks every minute for actions that are due, and runs each one once:

1. It checks everything again: scheduling is on, the agent exists and is enabled, your account is active and can still use the agent, AI is available, and, for an exact action, the tool is still available to the agent for you (the integration connected and shared with you, the tool not denied or waiting for approval).
2. It runs the action, as you: with your rights on the agent and the integrations the agent uses for you.
3. It records the outcome. **Done** keeps what the tool or the agent returned as proof, such as the link of the posted message. **Blocked** keeps the reason: a check that failed, or the error of the tool.

The outcome is posted back in the conversation the action came from, and you receive a notification. The run shows in :menu[Work > Operations], labelled **Scheduled**.

An action is never sent twice. A failure is retried, at most a few times, only when the action provably did not reach its target (the model provider could not be reached, the target answered "too many requests"). If XTM One restarts while sending an action, the item is blocked as interrupted: check the target, and reschedule it if needed.

An action runs up to 15 minutes late, when the platform is busy. Past that, it is **Missed**: it is not run at a time nobody chose, and you are notified. Reschedule it to run it, or cancel it.

## Follow, cancel or reschedule

A scheduled action is private to the person who asked for it, whatever the agent. You find it in four places:

| Where | What you see and do |
| --- | --- |
| The conversation | The card, then the outcome. Ask the agent "what did you schedule for me?" to list them |
| :menu[Orchestration > Agentic Flow], **Schedule** view of the **Agent-to-agent map** | Every action waiting, marked **One-off**, on the same timeline as the assignments, which are **Recurring**: see [Follow how agents hand work to each other](/xtm-one/use/agentic-flow/) |
| :menu[Work > Operations], **Work Items** tab | Each action as a work item marked **Scheduled**: its time, its tool and target, a preview, then its outcome and **View the run**. The **Scheduled** filter shows them alone |
| The agent's **Overview** | The **Coming up** card: your actions on that agent and its next assignment runs |

**Cancel** and **Reschedule** work everywhere the action shows, and from the chat ("move it to 08:00"). Cancelling before the time runs nothing. A running action can no longer be moved, and one that ran cannot be moved: schedule a new one.

Every scheduled action, each cancel and reschedule, and the run that executed it are recorded in the audit log.

## When scheduling is off

An administrator turns scheduling off for the whole platform in :menu[Administration > Settings > Access and security], under **Security options** (**Scheduled work items**), or with `SCHEDULED_WORK_ITEMS_ENABLED=false`, which wins: see the [configuration reference](/reference/xtm-one/configuration/). While it is off, agents answer that scheduling is disabled by the administrator, actions that come due are marked missed, and listing and cancelling keep working. The [emergency stop](/xtm-one/administer/emergency-stop/) also stops an action being run.

:::example[Post the morning handover in the SOC channel]
On 12 April 2026 at 23:10, Paul Garnier, SOC analyst on the night shift, has finished a summary of the night's Velvet Harbor alerts with **Night shift triage**. Julien Bernard, the SOC manager, wants it in the `#soc-night` Slack channel when the day shift arrives. Paul writes in the same conversation:

> Post this summary in #soc-night tomorrow at 07:30, and mention @julien.bernard.

The card shows:

| Field | Value |
| --- | --- |
| Time | Monday 13 April 2026, 07:30 (Europe/Paris), 05:30 UTC |
| Action | **Exact: send_slack_message** to the Slack channel `#soc-night` |
| Preview | The summary, with the mention of Julien Bernard |

At 07:30, XTM One checks that **Night shift triage** is still enabled and that Paul's Slack integration still reaches `#soc-night`, then posts the message as stored. The card in Paul's conversation turns to **Done**, with the link of the Slack message, and Paul gets a notification when he wakes up.
:::

## What's next

- [Follow how agents hand work to each other](/xtm-one/use/agentic-flow/), to see every one-off action and assignment on one timeline.
- [Review what agents did](/xtm-one/use/work-history/), to open the run of an action.
- [Create an agent](/xtm-one/build/agents/), to give an agent work that repeats.
