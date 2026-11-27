---
title: Connect Telegram
description: Create a Telegram bot with BotFather, connect it in XTM One and register its webhook, so people chat with an agent in a private chat or a group.
doc_type: how-to
products: [xtm-one]
personas: [admin, ai-builder]
owner: docs-xtm-one
last_reviewed: 2026-11-08
edition: ee
migrated_from:
  - xtm-one:/administration/channels/telegram/
---

Connecting Telegram lets people chat with an agent in a private chat or a group, and lets agents post messages, photos and polls back. It is the quickest app to set up: you create the bot in a few messages with `@BotFather`, paste its token in XTM One, and register the webhook with one API call. Telegram gives no email address, so senders are anonymous unless an administrator links them to their account. What every app shares is in [Talk to agents from your messaging apps](/xtm-one/administer/channels/).

::diagram{id=xtm-one-channels-telegram-setup}

## Before you start

- You have a Telegram account.
- You are an administrator of XTM One, which runs the Enterprise Edition with AI switched on.
- XTM One is reachable from the internet over HTTPS at its `BASE_URL`: Telegram posts updates to it.

## Create the bot

:::steps
1. In Telegram, start a conversation with `@BotFather`.
2. Send `/newbot` and answer its prompts: a name, then a username ending in `bot`.
3. Copy the token `@BotFather` returns. It has the form of a number, a colon and a string of letters.
4. If the bot must read group conversations, send `/setprivacy` to `@BotFather`, choose the bot and disable privacy mode. By default a bot sees only the group messages addressed to it.
:::

## Connect the bot in XTM One

:::steps
1. Open :menu[Administration > Settings > Integrations > Channels & Bots] and click **Configure** on the **Telegram** card.
2. Paste the token in **Bot Token / Access Token**. The field is required: empty, **Save Channel** does not create the bot.
3. Choose a webhook secret of 1 to 256 letters, digits, `_` and `-`, and type it in **Webhook secret token**. Telegram sends it back with every update, and XTM One refuses any update without it. Always set it.
4. Click **Save Channel**, and copy the **Webhook URL** of the card. It ends with `/api/channels/telegram/webhook`.
:::

::screenshot{id=xtm-one-channels-telegram-card alt="The Configure Telegram form inside its card: the BotFather token in Bot Token / Access Token, the webhook secret you choose, and the webhook URL to register with setWebhook, ending with /api/channels/telegram/webhook."}

## Register the webhook with Telegram

Telegram has no console for this: one call to the Bot API registers the URL and the secret. Pass the same secret as in XTM One, or Telegram's updates are rejected.

```bash
curl -X POST "https://api.telegram.org/bot$TELEGRAM_BOT_TOKEN/setWebhook" \
  -d "url=https://xtm-one.grand-meridian.example/api/channels/telegram/webhook" \
  -d "secret_token=$TELEGRAM_WEBHOOK_SECRET"
```

Telegram answers `"ok":true`. `getWebhookInfo` on the same API shows the registered URL and the last delivery error.

## Turn Telegram on for an agent

:::steps
1. Open the agent in :menu[Orchestration > Agents], go to **Configuration** > **Channels & Bots** and click **Edit**.
2. Turn on **Telegram**, then **Receive** so the agent answers incoming messages.
3. Leave **Publish** on if the agent may send messages on its own.
4. Decide on **Allow private conversations** (private chats), **Allow anonymous users** (people XTM One cannot match to an account) and **Only when mentioned** (groups), and click **Save**.
:::

Whitelist entries are Telegram chat IDs. With **Publish** on, the agent can send a message, send a photo by URL, edit or delete one of its own messages, forward a message, pin or unpin a message, create a poll, and read basic information about a chat, group or channel. Pinning, and deleting other people's messages, needs the bot to be an administrator of the chat.

## What is different on Telegram

- **No email.** Turn on **Allow anonymous users** to serve unknown senders, who reach the agent with their username and display name as context; or link a colleague's Telegram user ID to their account, as in [Recognize your colleagues](/xtm-one/administer/channels/whatsapp/#recognize-your-colleagues), with `"channel_type": "telegram"`.
- **Mentions, but no threads.** **Only when mentioned** works in groups, but the bot does not follow a thread: each new question needs its own mention. `/stop` is not available; `/reset` works.
- **Files.** Photos, documents, audio, video, voice messages and round video messages reach the agent with their caption as the message. Telegram lets a bot download up to 20 MB: a larger file is not passed on, but its caption is. Stickers are ignored.

## Check that it works

Send a message to the bot in Telegram. The agent answers.

| What you see | What to do |
| --- | --- |
| Nothing arrives | Call `getWebhookInfo`: it shows the URL Telegram uses and its last delivery error. |
| Every update is rejected | The `secret_token` of `setWebhook` is not the one stored in XTM One. Run `setWebhook` again with the right value. |
| The bot sees only messages that mention it | Privacy mode is on in `@BotFather`, or **Only when mentioned** is on. |
| The bot cannot pin or delete | It is not an administrator of that chat. |

:::example[Paul asks "Night shift triage" from the SOC's Telegram group]
The Grand Meridian Hotels SOC keeps a Telegram group, "GM SOC night", for the analysts on call. Hélène Marchetti creates `@gm_soc_night_bot` with `@BotFather`, pastes its token on the **Telegram** card of `https://xtm-one.grand-meridian.example`, and types `gm-soc-night-webhook-2026` in **Webhook secret token**. She runs `setWebhook` with the same secret, adds the bot to the group, and links Paul Garnier's Telegram user ID to his account.

Sophie Laurent turns on **Telegram** for the "Night shift triage" agent, with **Receive** on and **Only when mentioned** on. Paul writes in the group: "@gm_soc_night_bot did Velvet Harbor use booking-themed lures before January?" The agent answers as Paul; the next question needs a new mention.
:::

## What's next

- Decide when the bot answers and who it serves: [Talk to agents from your messaging apps](/xtm-one/administer/channels/).
- Connect another app: [Connect Discord](/xtm-one/administer/channels/discord/).
