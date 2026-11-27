---
title: Connect WhatsApp
description: Create a Meta app on the WhatsApp Business Cloud API, connect it in XTM One and register its webhook, so people reach an agent from a WhatsApp conversation.
doc_type: how-to
products: [xtm-one]
personas: [admin, ai-builder]
owner: docs-xtm-one
last_reviewed: 2026-11-08
edition: ee
migrated_from:
  - xtm-one:/administration/channels/whatsapp/
---

Connecting WhatsApp lets people reach an agent from an ordinary WhatsApp conversation, and lets agents send messages, templates and reactions back. It runs on Meta's WhatsApp Business Cloud API: you create a Meta app, paste its credentials in XTM One, and point the app's webhook at XTM One. WhatsApp gives no email address, so read [Recognize your colleagues](#recognize-your-colleagues) before you open it to anyone. What every app shares is in [Talk to agents from your messaging apps](/xtm-one/administer/channels/).

::diagram{id=xtm-one-channels-whatsapp-setup}

## Before you start

- You have a Meta developer account and a WhatsApp Business account with a phone number registered for the Cloud API.
- You are an administrator of XTM One, which runs the Enterprise Edition with AI switched on.
- XTM One is reachable from the internet over HTTPS at its `BASE_URL`: Meta posts messages to it.

## Create the Meta app

:::steps
1. Open [developers.facebook.com](https://developers.facebook.com/apps/), go to **My Apps** > **Create App** and choose the **Business** type.
2. Add the **WhatsApp** product to the app.
3. Open **WhatsApp** > **API Setup** and copy the **Phone Number ID**.
4. Generate a permanent access token. The temporary token of that page expires after 24 hours: use it for a first test only.
5. Copy the app secret from the app's basic settings.
:::

The app needs the `whatsapp_business_messaging` permission to send and receive messages, and `whatsapp_business_management` to manage numbers and templates.

## Connect the bot in XTM One

:::steps
1. Open :menu[Administration > Settings > Integrations > Channels & Bots] and click **Configure** on the **WhatsApp** card.
2. Paste the access token in **Bot Token / Access Token**.
3. Paste the phone number ID in **Phone number ID**: the ID Meta gives the number, not the phone number itself.
4. Choose a verify token, any random string, and type it in **Verify token**. You type the same value in Meta.
5. Paste the app secret in **App secret**. XTM One then checks that every incoming request is signed by Meta. Always fill it in.
6. Click **Save Channel**, and copy the **Webhook URL** of the card. It ends with `/api/channels/whatsapp/webhook`.
:::

::screenshot{id=xtm-one-channels-whatsapp-card alt="The Configure WhatsApp form inside its card: the access token, the Phone number ID, the Verify token you choose, the App secret, and the webhook URL to give Meta."}

**Bot Token / Access Token**, **Phone number ID** and **Verify token** are required: with one of them empty, **Save Channel** does not create the bot and the field reads "This field is required."

## Configure the webhook in Meta

:::steps
1. In the Meta app, open **WhatsApp** > **Configuration** > **Webhook**.
2. Set the callback URL to the webhook URL of the WhatsApp card.
3. Type the same verify token as in XTM One. Meta checks the URL at once: save the credentials in XTM One first.
4. Subscribe to the `messages` webhook field.
:::

## Turn WhatsApp on for an agent

:::steps
1. Open the agent in :menu[Orchestration > Agents], go to **Configuration** > **Channels & Bots** and click **Edit**.
2. Turn on **WhatsApp**, then **Receive** so the agent answers incoming messages.
3. Leave **Publish** on if the agent may send messages on its own.
4. Decide on **Allow private conversations** (direct conversations) and **Allow anonymous users** (people XTM One cannot match to an account), and click **Save**. Leave **Only when mentioned** off.
:::

Whitelist entries are phone numbers for direct conversations and group identifiers for groups. With **Publish** on, the agent can send a text message, send an approved template, mark a message as read, and react with an emoji.

## What is different on WhatsApp

- **No mentions.** The Cloud API does not report @mentions, so with **Only when mentioned** on, the bot stays silent in groups. Leave it off, on the bot and on the agent, and scope groups with the receive whitelist. For the same reason `/stop` mutes a group for the full seven days.
- **The 24-hour window.** Meta allows free-form messages only within 24 hours of the person's last message. Outside it, the agent must send a template approved in Meta Business Manager.
- **No typing indicator.** A long answer arrives with no sign of activity before it.
- **Files.** Photos, documents, audio, voice notes and videos reach the agent as files of the conversation, with their caption as the message. A file over the 40 MB upload limit of XTM One is not passed on, but its caption is. Stickers are ignored.

## Recognize your colleagues

WhatsApp gives a phone number, not an email, so XTM One cannot match a sender to an account by itself. Without a match, the sender is served anonymously when **Allow anonymous users** is on, and otherwise reads that their account could not be identified.

To let a colleague use an agent with their own access, an administrator links their WhatsApp number to their XTM One account once, through the API, with an API key from the **API Keys** tab of **My Profile** (the user menu, at the top right). The number is written as WhatsApp sends it: country code and number, digits only. The same call links a Telegram user ID (`telegram`) or a Discord user ID (`discord`).

```bash
curl -X POST "https://xtm-one.grand-meridian.example/api/v1/users/$PAUL_USER_ID/channel-identities" \
  -H "Authorization: Bearer $XTM_ONE_ADMIN_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"channel_type": "whatsapp", "platform_id": "33639980112"}'
```

`GET` on the same path lists a user's linked identities, and `DELETE` on `/api/v1/users/$PAUL_USER_ID/channel-identities/whatsapp/33639980112` removes one.

## Check that it works

Send a message to the business number from WhatsApp. The agent answers in the conversation.

| What you see | What to do |
| --- | --- |
| Meta rejects the callback URL | The verify token differs between Meta and XTM One, or the credentials were not saved first. |
| Messages arrive but nothing answers | Check the `messages` field subscription, the agent's **Receive**, and the receive whitelist. |
| The bot ignores a group | **Only when mentioned** is on somewhere, or the group is not in the whitelist. |
| A message the agent sends on its own fails | The 24-hour window has closed: send a template. |
| The sender is told their account could not be identified | Link their number, or turn on **Allow anonymous users** on the bot and on the agent. |

:::example[Paul asks "Night shift triage" from the on-call phone]
At night the SOC's on-call analyst is away from a desk. Hélène Marchetti connects the Grand Meridian Hotels WhatsApp Business number on the **WhatsApp** card, with the verify token `grand-meridian-whatsapp-2026` and the Meta app secret, and points the Meta webhook at `https://xtm-one.grand-meridian.example/api/channels/whatsapp/webhook`. She leaves **Allow anonymous users** off and links Paul Garnier's number, `33639980112`, to his account with the call above.

Sophie Laurent turns on **WhatsApp** for the "Night shift triage" agent, with **Receive** on, **Allow private conversations** on and **Only when mentioned** off. From the hotel car park, Paul writes to the business number: "Front-desk proxy in Lyon flagged 203.0.113.10, is it Velvet Harbor?" The agent answers as Paul, with his access to OpenCTI. A number nobody linked reads that its account could not be identified.
:::

## What's next

- Decide when the bot answers and who it serves: [Talk to agents from your messaging apps](/xtm-one/administer/channels/).
- Connect another app: [Connect Telegram](/xtm-one/administer/channels/telegram/).
