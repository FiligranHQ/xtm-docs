---
title: Stop every agent at once
description: Press the XTM One emergency stop to cancel every agent's work and pause every assignment, for your own activity or the whole platform, prepare a token for when nobody can sign in, and lift the stop.
doc_type: how-to
products: [xtm-one]
personas: [admin, ai-builder, soc-manager]
owner: docs-xtm-one
last_reviewed: 2026-11-08
migrated_from:
  - xtm-one:/administration/emergency-stop/
---

The emergency stop halts agent activity at once. It cancels the runs, background tasks and chat answers in progress, pauses the assignments so their triggers stop firing, and, for the whole platform, switches AI off so nothing new starts. Press it when an agent misbehaves, when costs run away, or when an incident requires that no AI activity continues. Every user can stop their own activity; an administrator can stop the whole platform. The stop stays in effect, visible on every page, until someone lifts it with a reason. This page presses the stop, explains what it reaches, prepares a token to press it without signing in, and lifts it. The same actions over HTTP are in the [XTM One emergency stop API](/develop/apis/emergency-stop/).

::diagram{id=xtm-one-emergency-stop-flow}

## Before you start

- Any user can stop their own activity. To stop the whole platform, or to create the emergency token, you are an XTM One administrator.
- No edition and no AI state is required: the stop works in every case, including when AI is already off.

## What the stop reaches

| You choose | Reaches |
| --- | --- |
| **My activity only** | Everything that runs for you: your runs and the work they handed to other agents, your background tasks, your chat answers being written, the commands your agents sent to computers and not yet dispatched, and the assignments you own or that run as you |
| **Whole platform** (administrators) | The same for every user, plus the autonomous attack paths XTM One drives for OpenAEV, plus the AI switch: AI goes off for everyone |

Work in progress is cancelled and is never resumed: lifting the stop rearms the triggers, it does not replay what was cut. A run cancelled by the stop reads "Stopped by an emergency stop." on its card and its page.

Two kinds of work cannot be called back and finish on their own:

- a command already running on a computer through XTM One Desktop runs to its end, but no command is sent again;
- an answer an agent is already writing for a messaging bot, an MCP client, the browser extension or the assistant of OpenCTI, OpenAEV or OpenCRQ finishes under its own time limit. With **Whole platform**, no new one starts.

What is not agent work keeps running: document processing, knowledge-base synchronization, integration imports and exports. Your data is untouched: agents, knowledge bases, runs and traces stay readable.

## Press the emergency stop

:::steps
1. Open your profile menu, at the top right, and click **Emergency stop**.

   ::screenshot{id=xtm-one-emergency-stop-dialog alt="The Emergency stop dialog of an administrator: the scopes My activity only and Whole platform, the six-character code to type, and Stop everything still disabled."}
2. As an administrator, choose the **Scope**: **My activity only** or **Whole platform**. Other users stop their own activity, with no choice to make.
3. Type the six-character code the dialog shows, under "To confirm, type the code below:". **Stop everything** stays disabled until the code matches, so a slip of the mouse is never enough.
4. Click **Stop everything**. The stop runs at once, and a notification lists what was cancelled and paused: **Emergency stop done**, or **Emergency stop partly applied** when some running work could not be cancelled at once. The assignments are paused and AI is off anyway, and follow-up passes cancel what they reach; if work still runs, press the stop again.
5. The dialog then asks **Everything is stopped. Why?** The stop does not wait for the answer: it is already done. Type the reason and click **Save reason**, or **Skip**. The reason is written in the logs next to the stop and on every assignment the stop paused.
:::

## What you see while the stop is in effect

- **A red banner** on every page reads **Emergency stop in effect**, with how many assignments are paused, whether AI is switched off for everyone, who pressed it, when and why. A stop of the whole platform shows it to every user; only an administrator can lift it, so the others read "Only an administrator can lift it.". A stop of the whole platform stays in effect until it is lifted, even when the account that pressed it, or that generated the token, is deleted: the banner then no longer says who pressed it or why.

  <!-- shot: xtm-one-emergency-stop-banner -->
- **Assignments** the stop paused carry a red **Emergency stop** badge instead of **Paused**, on their page, in the agent's assignments and on the status tile. Their page says "Paused by the emergency stop — lift the stop to resume it." Their usual on/off switch cannot turn them back on, and neither can an agent through its tools.
- **Runs** the stop cancelled read "Stopped by an emergency stop.".
- **Chat**, after a stop of the whole platform, shows AI as switched off.
- **:menu[Administration > Audit logs]** holds one entry per stop and per resume, with the counts and the reason, and one pause or resume entry per assignment.

## Lift the emergency stop

:::steps
1. Click **Resume the whole platform** or **Resume my activity** in the banner, or in your profile menu. A user who is not an administrator finds **Resume after emergency stop** in the profile menu.
2. In **Resume the whole platform?** or **Resume your activity?**, type why you lift the stop. The reason is required: it is recorded on the resume entry and on every assignment re-enabled.
3. Click **Resume**. The notification **Emergency stop lifted** says how many assignments resumed and whether AI was switched back on.
:::

Lifting the stop:

- re-enables only the assignments the stop paused: one somebody paused by hand stays paused;
- sets the AI switch back to what it was before a stop of the whole platform, **Enabled**, **Disabled** or **Deployment default**, so a platform that had AI off keeps it off; a change made in :menu[Administration > Settings > Platform] during the stop is kept;
- for an assignment that two people's stops cover, one owning it and the other running it, waits until both have lifted theirs;
- while the whole platform is stopped, only lifting that stop resumes anything: a user cannot lift their own stop underneath it, and a user's own stop stays in place when the platform stop is lifted.

## Prepare a stop without a session

The stop may be needed when nobody can sign in: the on-call engineer has only a terminal, or a monitoring system must press it itself. For that, an administrator creates the emergency stop token in advance:

:::steps
1. Open :menu[Administration > Settings > Access and security] and click the section **Emergency stop**. The card **Emergency stop API token** shows its **Status**: **Configured** or **Not configured**.
2. Click **Generate token**. The token is shown once: XTM One keeps only a hash of it and can never display it again.
3. Click **Print the procedure**. A page opens for paper or PDF, in the language of your interface: when to use the stop, what it does, the exact command with the token and the platform's address, how to lift the stop, and a space for the people to call. Keep it with the on-call material.
:::

The card then shows the **Current token prefix**, who generated it and when, and when it was last used. **Rotate token** replaces the token, and every printed procedure with the old one stops working. **Revoke** disables the token until a new one is generated; the profile menu keeps working.

::screenshot{id=xtm-one-emergency-stop-token alt="The Emergency stop API token card once the token exists: Configured, the token prefix, generated by Hélène Marchetti and never used, with the Rotate token and Revoke buttons."}

The token does one thing: stop the whole platform. It cannot read or change anything else. A stop pressed with it is attributed to the administrator who generated it, and every refused attempt is recorded in the logs as a security event. Treat the printed page like a physical key, and rotate the token as soon as a copy is lost.

:::example[Stopping a looping agent on the night shift]
At 02:40 on 12 April 2026, the SOC of Grand Meridian Hotels sees **Night shift triage** create Jira tickets in a loop, one per alert of the same Velvet Harbor indicator, and the agentic quota drops fast.

1. Paul Garnier, on the night shift, opens his profile menu, clicks **Emergency stop**, types the code and clicks **Stop everything**. Only his own activity stops: the agent's runs that were started for him.
2. The agent's assignment runs as Sophie Laurent, so it keeps firing. Paul calls Marc Dupuis, on call for IT operations, who has no session open. Marc runs the command of the printed procedure from his terminal:

   ```bash
   curl -X POST https://xtm-one.grand-meridian.example/api/v1/panic/emergency \
     -H "Authorization: Bearer $XTM_ONE_EMERGENCY_TOKEN" \
     -H "Content-Type: application/json" \
     -d '{"reason": "Night shift triage creating Jira tickets in a loop"}'
   ```

3. The whole platform stops: every run is cancelled, every assignment paused, AI off. The banner names Hélène Marchetti, who generated the token, and the reason.
4. In the morning, Sophie fixes the agent's instructions. Hélène clicks **Resume the whole platform** with the reason "Triage loop fixed in the agent's instructions". The assignments the stop paused resume, and AI goes back to what it was before the stop.
:::

## What's next

- [XTM One emergency stop API](/develop/apis/emergency-stop/): every route, for a monitoring hook.
- [Run XTM One with AI switched off](/xtm-one/administer/running-without-ai/): to keep AI off without cancelling anything.
- [Limit what an agent run may spend](/xtm-one/use/run-budgets/)
