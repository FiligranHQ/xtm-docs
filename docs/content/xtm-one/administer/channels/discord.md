---
title: Connect Discord
description: Create a Discord application and its bot, invite it to your server, connect it in XTM One and restart, so people talk to an agent in Discord.
doc_type: how-to
products: [xtm-one]
personas: [admin, ai-builder]
owner: docs-xtm-one
last_reviewed: 2026-11-08
edition: ee
migrated_from:
  - xtm-one:/administration/channels/discord/
---

Connecting Discord lets people talk to an agent in a server channel or a direct message, and lets agents post messages, reactions and threads back. Discord is the one app without a webhook: XTM One opens its own connection to Discord when it starts. So you create the bot, invite it, paste its token in XTM One, then restart XTM One. Discord gives no email address, so senders are anonymous unless an administrator links them to their account. What every app shares is in [Talk to agents from your messaging apps](/xtm-one/administer/channels/).

::diagram{id=xtm-one-channels-discord-setup}

## Before you start

- You can create an application in the Discord developer portal and invite a bot to the target server.
- You are an administrator of XTM One, which runs the Enterprise Edition with AI switched on.
- You, or the platform engineer, can restart XTM One after saving the token.

## Create the Discord application

:::steps
1. Open [discord.com/developers/applications](https://discord.com/developers/applications) and create a new application.
2. Open **Bot**, click **Reset Token** and copy the token.
3. Under **Privileged Gateway Intents**, turn on **Message Content Intent**. Without it, Discord delivers messages without their text, and the agent has nothing to answer.
:::

Agents answer messages, not slash commands: XTM One acknowledges a slash command with "Processing your request..." and nothing more. Leave the application's interactions endpoint unset in the developer portal.

## Invite the bot to your server

:::steps
1. Open **OAuth2** > **URL Generator**.
2. Select the `bot` scope.
3. Select the permissions **Send Messages**, **Read Message History** and **Add Reactions**.
4. Open the generated URL and invite the bot to the server.
:::

## Connect the bot in XTM One

:::steps
1. Open :menu[Administration > Settings > Integrations > Channels & Bots] and click **Configure** on the **Discord** card.
2. Paste the bot token in **Bot Token / Access Token**.
3. Leave **Public key** empty. It serves only the interactions endpoint of slash commands, the card's **Webhook URL** ending with `/api/channels/discord/interactions`, which messages do not use.
4. Click **Save Channel**.
5. Restart XTM One. XTM One opens the Discord connection when it starts: a bot saved on a running platform stays offline until the next restart.
:::

::screenshot{id=xtm-one-channels-discord-card alt="The Configure Discord form inside its card: the bot token, the optional public key, and the interactions URL that only slash commands use."}

## Turn Discord on for an agent

:::steps
1. Open the agent in :menu[Orchestration > Agents], go to **Configuration** > **Channels & Bots** and click **Edit**.
2. Turn on **Discord**, then **Receive** so the agent answers incoming messages.
3. Leave **Publish** on if the agent may post on its own.
4. Decide on **Allow private conversations** (direct messages), **Allow anonymous users** (people XTM One cannot match to an account) and **Only when mentioned** (server channels), and click **Save**.
:::

Whitelist entries are Discord channel IDs: turn on developer mode in Discord, right-click a channel and choose **Copy Channel ID**. With **Publish** on, the agent can send a message, read recent messages of a channel, list the channels it sees, create a thread, add a reaction, pin or unpin a message, and delete one of its own messages.

## What is different on Discord

- **No email.** Turn on **Allow anonymous users** to serve unknown senders, who reach the agent with their username as context; or link a colleague's Discord user ID to their account, as in [Recognize your colleagues](/xtm-one/administer/channels/whatsapp/#recognize-your-colleagues), with `"channel_type": "discord"`.
- **Mentions, but no threads.** **Only when mentioned** works in server channels, but the bot does not follow a thread: each new question needs its own mention. `/stop` is not available; `/reset` works.
- **Wide reach.** An invitation exposes every channel of the server the bot can read: set the receive whitelist to the channels where the bot is expected.
- Answers longer than Discord's limit are split into several messages.

## Check that it works

After the restart, the bot shows as online in the server. Send it a direct message: the agent answers.

| What you see | What to do |
| --- | --- |
| The bot shows as offline | XTM One was not restarted since the token was saved, or the token is wrong. Restart and check the XTM One logs for the Discord connection. |
| Messages arrive empty or are ignored | **Message Content Intent** is off in the developer portal. |
| Answers in direct messages, not in a channel | Check the channel's permissions in Discord, the receive whitelist, and whether a mention is needed. |
| A slash command only answers "Processing your request..." | Expected: agents answer messages. Ask the question in a message that mentions the bot. |

:::example[Paul asks "Night shift triage" in the SOC's Discord server]
The Grand Meridian Hotels SOC chats on a private Discord server during exercises. Hélène Marchetti creates the **XTM One** application, turns on **Message Content Intent**, invites the bot with **Send Messages**, **Read Message History** and **Add Reactions**, pastes its token on the **Discord** card and asks Marc Dupuis to restart XTM One. On the card she sets the **Receive** whitelist to the ID of the `#soc-night` channel and links Paul Garnier's Discord user ID to his account.

Sophie Laurent turns on **Discord** for the "Night shift triage" agent, with **Receive** on and **Only when mentioned** on. Paul writes in `#soc-night`: "@XTM One list the Velvet Harbor indicators added this week." The agent answers as Paul.
:::

## What's next

- Decide when the bot answers and who it serves: [Talk to agents from your messaging apps](/xtm-one/administer/channels/).
- Connect another app: [Connect Google Chat](/xtm-one/administer/channels/google-chat/).
