---
title: Connect Slack
description: Create a Slack app, connect it in XTM One and subscribe it to events, so your teams ask agents questions in Slack channels and direct messages.
doc_type: how-to
products: [xtm-one]
personas: [admin, ai-builder]
owner: docs-xtm-one
last_reviewed: 2026-11-08
edition: ee
migrated_from:
  - xtm-one:/administration/channels/slack/
---

Connecting Slack lets your teams ask an agent a question in a channel or a direct message, and lets agents post to Slack on their own, such as a triage summary in an incident thread. You create a Slack app in your workspace, paste its token and signing secret in XTM One, give Slack the webhook URL, then turn Slack on for an agent. Whitelists, mentions and anonymous access work as on every app: read [Talk to agents from your messaging apps](/xtm-one/administer/channels/) first.

::diagram{id=xtm-one-channels-slack-setup}

## Before you start

- You can create and install an app in the Slack workspace, or a Slack administrator approves the installation.
- You are an administrator of XTM One, which runs the Enterprise Edition with AI switched on.
- XTM One is reachable from the internet over HTTPS at its `BASE_URL`: Slack posts events to it.

## Create the Slack app

:::steps
1. Open [api.slack.com/apps](https://api.slack.com/apps), click **Create New App** > **From scratch**, name the app and pick your workspace. The name is the one people @mention.
2. Open **OAuth & Permissions** and add every scope of the table below under **Bot Token Scopes**.
3. Open **Agents & Assistants** and turn on the **App Agent** toggle. It lets the bot show that it is thinking while the agent works.
4. Install the app to the workspace, then copy the **Bot User OAuth Token**. It starts with `xoxb-`.
5. Open **Basic Information** and copy the **Signing Secret**.
:::

| Scope | Why the bot needs it |
| --- | --- |
| `app_mentions:read` | Receive messages that @mention the bot |
| `assistant:write` | Show the typing indicator while the agent works |
| `channels:history` | Read messages in the public channels the bot joined |
| `channels:join` | Join public channels |
| `channels:read` | Read basic information about public channels |
| `chat:write` | Send messages as the bot |
| `groups:history` | Read messages in the private channels the bot was added to |
| `groups:read` | Read basic information about those private channels |
| `im:history` | Read direct messages with the bot |
| `im:read` | Read basic information about direct messages |
| `im:write` | Start a direct message with someone |
| `users:read` | Read workspace profiles |
| `users:read.email` | Match a Slack user to their XTM One account by email |

A scope added after installation needs the app reinstalled. Without `users:read` and `users:read.email`, XTM One cannot tell who writes: senders are then served only if anonymous access is allowed.

## Connect the bot in XTM One

:::steps
1. Open :menu[Administration > Settings > Integrations > Channels & Bots] and click **Configure** on the **Slack** card.
2. Paste the bot token in **Bot Token / Access Token**.
3. Paste the signing secret in **Signing secret**.
4. Click **Save Channel**. The card shows **Connected**, and its **Receive** column shows the **Webhook URL**, ending with `/api/channels/slack/events`.
:::

Both fields are required: with one of them empty, **Save Channel** does not create the bot and the field reads "This field is required."

::screenshot{id=xtm-one-channels-slack-card alt="The Configure Slack form inside its card: the bot token, the signing secret, and the events URL ending with /api/channels/slack/events."}

Save the signing secret before the next step. XTM One refuses every Slack request it cannot check against the signing secret, Slack's own URL verification included.

## Subscribe the app to events

:::steps
1. Back in the Slack app, open **Event Subscriptions** and turn on events.
2. Paste the webhook URL of the Slack card in **Request URL**. Slack checks it at once and shows it as verified.
3. Under **Subscribe to bot events**, add `message.im`, then one of the two channel events below, and save.
4. In each Slack channel where the bot should work, type `/invite @` followed by the app's name. The bot sees only the channels it belongs to.
:::

| Event | What it delivers | When to add it |
| --- | --- | --- |
| `message.im` | Direct messages with the bot | Always |
| `app_mention` | Messages that @mention the bot in a channel | The usual setup: the bot answers mentions |
| `message.channels` | Every message of the public channels the bot joined | A channel dedicated to the agent, where it answers everything |

Choose `app_mention` or `message.channels`, not both: one channel event is enough.

## Turn Slack on for an agent

:::steps
1. Open the agent in :menu[Orchestration > Agents], go to **Configuration** > **Channels & Bots** and click **Edit**.
2. Turn on **Slack**.
3. Turn on **Receive** so the agent answers Slack messages. Under **Restrict to specific channels**, pick the channels it answers in; the list shows the channels the bot can see.
4. Leave **Publish** on if the agent may post to Slack on its own, and pick the channels it may post to.
5. Adjust **Allow anonymous users**, **Allow private conversations** and **Only when mentioned** for this agent, within what the bot card allows, and click **Save**.
:::

With **Publish** on, the agent can send a message, reply in a thread, read recent messages or a thread, list channels and users, look up a user, add or remove a reaction, pin or unpin a message, set a channel topic, and delete one of its own messages. Its answers are converted to Slack formatting, and a long answer is split into several messages.

## Check that it works

1. Send the bot a direct message. The bot shows that it is thinking, and the agent answers.
2. In an invited channel, @mention the bot. The agent answers in a thread.
3. On the Slack card, **Routing** under **Receive** lists the agent.

| What you see | What to do |
| --- | --- |
| Slack rejects the **Request URL** | The signing secret is missing or wrong in XTM One. Save it on the card with **Edit credentials**, then retry the verification. |
| The bot ignores channel messages | Invite it to the channel, check the subscribed event, the receive whitelist and the agent's **Receive**. |
| No typing indicator | The **App Agent** toggle is off or `assistant:write` is missing. Everything else works. |
| The answer appears in the channel, not the thread | Slack refuses thread replies on some message types; the answer is posted in the channel rather than lost. |

:::example[Paul asks "Night shift triage" from #soc-night]
Hélène Marchetti creates a Slack app named **XTM One** in the Grand Meridian Hotels workspace, with the thirteen scopes and **App Agent** on, and installs it. On the **Slack** card of `https://xtm-one.grand-meridian.example` she pastes the **Bot User OAuth Token** in **Bot Token / Access Token** and the **Signing Secret** in **Signing secret**, then clicks **Save Channel**.

In **Event Subscriptions** she sets **Request URL** to `https://xtm-one.grand-meridian.example/api/channels/slack/events`, subscribes to `message.im` and `app_mention`, and types `/invite @XTM One` in `#soc-night`. On the card she sets the **Receive** whitelist to `#soc-night` and turns on **Only when mentioned**. Sophie Laurent turns on **Slack** for the "Night shift triage" agent, with **Receive** on and restricted to `#soc-night`.

At 02:14 Paul Garnier writes in `#soc-night`: "@XTM One is 203.0.113.10 linked to Velvet Harbor?" The agent answers in the thread, as Paul, from what OpenCTI knows of Velvet Harbor.
:::

## What's next

- Decide when the bot answers and who it serves: [Talk to agents from your messaging apps](/xtm-one/administer/channels/).
- Connect another app: [Connect Microsoft Teams](/xtm-one/administer/channels/microsoft-teams/).
