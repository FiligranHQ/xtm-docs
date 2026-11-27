---
title: Talk to agents from your messaging apps
description: How a channel bot lets your teams ask XTM One agents questions from Slack, Microsoft Teams, WhatsApp, Telegram, Discord or Google Chat, and what decides when it answers.
doc_type: explanation
products: [xtm-one]
personas: [admin, ai-builder]
owner: docs-xtm-one
last_reviewed: 2026-11-08
edition: ee
migrated_from:
  - xtm-one:/administration/channels/index/
---

A channel bot lets people talk to an XTM One agent from the messaging app they already work in, instead of opening XTM One: a SOC analyst asks the triage agent about an alert in a Slack channel, and the agent answers in the thread. The same bot also lets agents post on their own, for example a summary at the end of a scheduled run. You connect one bot per messaging app in :menu[Administration > Settings > Integrations > Channels & Bots], then choose on each agent whether it answers messages, posts messages, or both.

::diagram{id=xtm-one-channels-message-flow}

::screenshot{id=xtm-one-channels-settings alt="The Channels & Bots section of the Integrations tab of Settings before any bot is connected: Web Chat Always Active, then Slack, Microsoft Teams, WhatsApp, Telegram, Google Chat and Discord, each with Setup guide and Configure."}

## What it is good for

| Goal | Example |
| --- | --- |
| Answer where the question is asked | Paul Garnier asks the "Night shift triage" agent about an IP address in Slack #soc-night at 2 a.m. |
| Post without being asked | The "Velvet Harbor watcher" agent posts its summary of a new report to the CTI team's Teams channel. |
| Reach people without an XTM One account | A dedicated agent answers questions in a public Telegram group, with a narrow set of tools. |
| Keep one conversation across devices | Paul starts a conversation in the web chat and continues it in a direct message with the Slack bot. |

## Supported messaging apps

Web chat is always there: the **Web Chat** card shows **Always Active**, needs no setup and cannot be turned off. The other apps each need a bot, connected on its own page.

| App | What you set up on its side | How it reaches XTM One |
| --- | --- | --- |
| [Slack](/xtm-one/administer/channels/slack/) | A Slack app with a bot token and a signing secret | Webhook |
| [Microsoft Teams](/xtm-one/administer/channels/microsoft-teams/) | An Entra ID app registration, an Azure Bot resource and a Teams app package | Webhook |
| [WhatsApp](/xtm-one/administer/channels/whatsapp/) | A Meta app on the WhatsApp Business Cloud API | Webhook |
| [Telegram](/xtm-one/administer/channels/telegram/) | A bot created with `@BotFather` | Webhook |
| [Discord](/xtm-one/administer/channels/discord/) | A Discord application with a bot token | A connection XTM One opens when it starts |
| [Google Chat](/xtm-one/administer/channels/google-chat/) | A Google Chat app and a service account key | Webhook |

A bot speaks as itself: people address the bot. To let an agent answer from a real user account instead, yours or a dedicated AI account, see [Give an agent its own account in a channel](/xtm-one/administer/channels/agent-accounts/).

## What every connection needs

- An administrator of XTM One: only administrators create, edit or delete a channel bot.
- The Enterprise Edition, with AI switched on: saving a new bot is refused otherwise. See [What each edition of XTM One brings](/xtm-one/administer/editions/).
- A public HTTPS address. Every app but Discord sends its messages to a webhook URL built from `BASE_URL` (see the [configuration reference](/reference/xtm-one/configuration/#urls)). If `BASE_URL` changes, update the URL in every messaging app.
- Someone with administrator rights in the messaging app: most of the steps happen in its own console.

## How a channel goes live

Every connection page follows the same four steps.

1. **Create the app in the messaging app's console.** This is where you get the credentials and grant the permissions.
2. **Connect the bot in XTM One.** In :menu[Administration > Settings > Integrations > Channels & Bots], click **Configure** on the app's card, paste the credentials and click **Save Channel**. Each field has a line under it saying what to paste, and an **i** next to its name saying where to find it in the messaging app; a new bot is not saved until its required fields are filled. XTM One encrypts every value before storing it, and the card shows **Connected**.
3. **Give the messaging app the webhook URL.** The connected card shows it under **Receive**, as **Webhook URL**, with a copy button. Save the credentials first: XTM One checks the signature of the app's first verification request.
4. **Turn the channel on for an agent.** A connected bot answers nothing on its own. Open the agent in :menu[Orchestration > Agents], go to **Configuration** > **Channels & Bots**, click **Edit**, turn on the app, then turn on **Receive**, **Publish** or both, and click **Save**. A channel turned on starts with **Receive** off and **Publish** on.

Each card has a **Setup guide** button: it lists the same steps with the permissions to grant and the real webhook URL of your platform, and for Teams a manifest to copy.

## Receive and Publish

A bot works in two directions, shown side by side on its card and again on each agent.

- **Receive** covers incoming messages: someone writes in a channel or sends a direct message, and an agent answers.
- **Publish** covers outgoing messages: the agent uses the bot's tools (send a message, reply in a thread, add a reaction, upload a file, depending on the app) on its own initiative. The tool count on the card lists them, each one with a **Test** button to try it against the live connection.

They are separate on purpose: an agent can post a daily summary to a channel without listening to it, or answer in a channel without being allowed to post anywhere else. On the agent, **Publish** also chooses which bot tools the agent gets (**All bot tools** by default).

## Which agent answers

Several agents can share one bot. For each message, XTM One looks for the agent in this order:

1. In a direct message, the agent the sender chose for that app in **My Profile** (the user menu), on the **Channels & Bots** tab, under **External Channels**.
2. An agent the sender can use (their own, shared with their group, or company-managed) with **Receive** on for that app and allowed in that channel.
3. Any enabled agent with **Receive** on for that app and allowed in that channel: an agent an administrator set up answers everyone.
4. The General Assistant, when it is turned on for that app. Otherwise the message gets no answer.

Give each agent its own channels with the receive whitelists below, so that one channel has one agent. The bot card shows the result under **Routing** (who answers there) and **Binding** (who posts there), with a button to remove an agent from the channel.

## Limit the channels a bot reaches

A bot can reach every channel it has been invited to, private channels included. Two whitelists on the bot card narrow that inside XTM One, for every agent that uses the bot:

- The **Receive** whitelist: the channels XTM One accepts messages from. Messages from anywhere else are ignored, without an answer.
- The **Publish** whitelist: the channels agents can post to. A post anywhere else fails with an access-denied error instead of landing in the wrong place.

Click **Edit** next to **Allowed Channels**, pick channels from the list the bot can see or type a **Channel ID**, and click **Save**. The identifier depends on the app: a Slack channel ID, a Teams team and channel, a WhatsApp phone number or group, a Telegram chat ID, a Discord channel ID, a Google Chat space name.

An empty whitelist allows every channel, and a bot with no whitelist at all is reserved for company-managed agents: users see it on their own agents only once a whitelist exists, so a personal agent never gets workspace-wide reach by accident. On each agent, **Restrict to specific channels** narrows further; the agent gets the overlap of its choice and the bot's whitelist, never more. An administrator can turn on **Bypass whitelist** on one agent to let it receive and publish on every channel: a message the whitelist would have dropped then reaches that agent only.

## When the bot answers

Four switches on the bot card decide when and how the bot answers. The first three exist again on each agent, under **Receive** and with the same names, and the bot always wins: on the agent, a switch the bot holds says so under its name, for example "Turned off on the bot".

| Switch | On the agent too | Effect | Default on the bot |
| --- | --- | --- | --- |
| **Allow anonymous users** | Yes | People with no XTM One account can talk to the agent. With it off on the bot, no agent can turn it on. | Off |
| **Allow private conversations** | Yes | Direct messages reach the agent even when the receive whitelist would drop them. With it off on the bot, no agent can turn it on; only an administrator changes it on an agent. | On |
| **Only when mentioned** | Yes | In group channels, the bot answers only when it is @mentioned. With it on on the bot, no agent can turn it off. | Off |
| **Ask before running tools** | No | A tool that needs approval stops the conversation instead of running. Off: conversations through this bot, and its own send, edit and delete tools, run without asking. | On |

What follows from them:

- A direct message never needs a mention.
- With **Only when mentioned** on, once the bot has answered in a thread it follows that thread for 24 hours without a new mention, on Slack, Microsoft Teams and Google Chat. Telegram and Discord have no threads to follow: each question needs its own mention.
- With **Only when mentioned** off, in a group channel the agent reads messages that do not mention it and stays silent unless they are for it.
- WhatsApp reports no mentions: with **Only when mentioned** on, a WhatsApp bot stays silent in groups. Scope it with the receive whitelist instead.
- A messaging app cannot show an approval prompt. With **Ask before running tools** on, a tool that needs approval ends the turn with a message saying so; the sender continues in the web chat or asks for something else. Turn it off only for a bot whose agents have tools you trust to run unattended: XTM One asks you to confirm, and records the change in the audit log. How approvals work is in [Govern how AI is used](/xtm-one/administer/ai-governance/).

### People without an account

With **Allow anonymous users** on, the agent answers people XTM One cannot match to an account, and gets their display name and platform identity as context. The General Assistant never answers them: only agents do.

Such a conversation uses what the agent is configured with: its knowledge bases, integrations and handover agents, whatever their **Restrict access** setting. What the agent browses across the platform is limited to what every user sees. Anonymous messages count against the platform-wide agentic limits of your license; at the limit, the sender reads `This service is at capacity right now. Please try again later.`

:::warning
**Allow anonymous users** lets anyone who can reach the channel use the agent, its tools and its knowledge bases. Use a dedicated agent with read-only tools and only the knowledge it may share.
:::

## How the bot knows who is talking

XTM One matches the sender to an account by email address. A matched sender gets their own permissions, preferences and memory, as in the web chat.

| App | Email given | Without a match |
| --- | --- | --- |
| Slack, Google Chat | Yes | Anonymous, if allowed |
| Microsoft Teams | Yes, or read from the sender's Entra ID identity | Anonymous, if allowed |
| WhatsApp, Telegram, Discord | No | Anonymous if allowed; otherwise the sender reads that their account could not be identified |

Once a sender is matched by email, XTM One remembers their identity on that app. For WhatsApp, Telegram and Discord, an administrator links a person's phone number or user ID to their account through the API: see [Recognize your colleagues](/xtm-one/administer/channels/whatsapp/#recognize-your-colleagues).

## Commands in a conversation

Tell your users about two commands when you announce a bot:

- `/reset` or `/new` archives the conversation and starts a fresh one, to clear a context that drifted. In a direct message that continues a web conversation, it ends the continuation and keeps the web conversation.
- `/stop` mutes the bot in the current thread for seven days. Mentioning the bot lifts the mute. It works where threads exist: Slack, Microsoft Teams, Google Chat and WhatsApp groups. On WhatsApp the mute lasts the full seven days, since a mention cannot lift it.

## One conversation on the web and in Slack or Teams

A user can continue a web conversation in a direct message with the Slack or Teams bot, and back (see [Ask an agent in the chat](/xtm-one/use/chat/)). It stays one conversation, with its history, files and pending approvals.

- Only direct messages are shared with the web. Channel threads and group chats are never shown on the web, because other people read them.
- The direct message continues the conversation only when its sender is the conversation's owner.
- The agent must have the app turned on with **Receive** and **Publish**. The continuation is offered only where the bot would answer the direct message.
- One turn runs at a time. A Slack or Teams direct message sent while a turn runs waits (up to five messages, for an hour at most) and is answered next.
- On Teams, the user must have written to the bot once in a personal chat: Teams lets a bot write first only in a chat it already has.

## Files and the bot's own messages

When an answer includes a file, the message carries a link to it that works without signing in for 24 hours: treat the link like the file. PDFs, text, Markdown, CSV, JSON, images, audio and video open in the browser; HTML, SVG, XML and every other type are downloaded, never opened.

Besides the agents' answers, the bot sends short notices of its own (a message is queued, a quota is reached, an error). They are in English, French, Spanish or Japanese: the language of the sender's profile, else the platform's **Default language** in :menu[Administration > Settings > Platform], else English. The agents' answers follow the conversation.

## Good habits

- Connect one bot per app and share it between agents, each with its own channels.
- Set the receive whitelist before inviting the bot to busy channels; widen it once the behavior is what you expect.
- Turn on **Only when mentioned** in shared team channels; leave it off in a channel dedicated to the agent.
- Keep publish whitelists to the channels an agent reports into.
- Rotate credentials in the messaging app, then paste the new value with **Edit credentials** (the pencil on the card) and click **Update Channel**. Fields left blank keep their stored value.

## When the bot does not answer

Check, in this order:

1. The card shows **Connected**.
2. An agent has the app turned on with **Receive** on, or the General Assistant is turned on for it.
3. The channel is in the receive whitelist, or there is none.
4. The bot is a member of the channel.
5. In a group channel, **Only when mentioned** is on and the message has no mention.

The bot answers in direct messages but not in a channel: almost always the receive whitelist, a missing invitation or a missing mention. An agent cannot post: the bot's publish whitelist or the agent's own **Publish** selection, since the agent gets their overlap. The messaging app rejects the webhook URL: save the credentials in XTM One first, and check that `BASE_URL` is the public HTTPS address you pasted.

:::example[Paul asks the night shift agent about an alert from Slack]
Grand Meridian Hotels runs XTM One at `https://xtm-one.grand-meridian.example`. Hélène Marchetti, the CISO, connected the Slack workspace on the **Slack** card, then set the **Receive** whitelist to `#soc-night` and turned on **Only when mentioned**. Sophie Laurent opened the "Night shift triage" agent, went to **Configuration** > **Channels & Bots**, turned on **Slack** with **Receive** on, **Restrict to specific channels** set to `#soc-night`, and **Publish** on.

At 02:14 the front-desk proxy of the Lyon hotel flags 203.0.113.10. Paul Garnier, on the night shift, writes in `#soc-night`: "@XTM One is 203.0.113.10 linked to Velvet Harbor?" Slack gives XTM One his email, so the agent answers as Paul, with his access, and reads OpenCTI: the address is an Indicator of Velvet Harbor, seen in the 9 January 2026 intrusion. Paul's colleagues ask follow-up questions in the same thread without mentioning the bot again, and Paul types `/stop` when the alert is closed.
:::

## What's next

- Connect your first app: [Connect Slack](/xtm-one/administer/channels/slack/), [Connect Microsoft Teams](/xtm-one/administer/channels/microsoft-teams/), [Connect WhatsApp](/xtm-one/administer/channels/whatsapp/), [Connect Telegram](/xtm-one/administer/channels/telegram/), [Connect Discord](/xtm-one/administer/channels/discord/) or [Connect Google Chat](/xtm-one/administer/channels/google-chat/).
- Choose the tools and knowledge of the agent that answers: [Create an agent](/xtm-one/build/agents/).
- Let an agent answer from a real account: [Give an agent its own account in a channel](/xtm-one/administer/channels/agent-accounts/).
