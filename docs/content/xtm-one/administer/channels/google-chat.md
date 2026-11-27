---
title: Connect Google Chat
description: Configure a Google Chat app and its service account, connect them in XTM One and add the app to spaces, so Google Workspace users talk to an agent.
doc_type: how-to
products: [xtm-one]
personas: [admin, ai-builder]
owner: docs-xtm-one
last_reviewed: 2026-11-08
edition: ee
migrated_from:
  - xtm-one:/administration/channels/google-chat/
---

Connecting Google Chat lets Google Workspace users talk to an agent in a space or a direct message, and lets agents post messages back into spaces. You configure a Chat app in Google Cloud that sends its messages to XTM One, give XTM One a service account key to answer with, then add the app to spaces. Google Chat gives the sender's email, so each person is answered with their own access. What every app shares is in [Talk to agents from your messaging apps](/xtm-one/administer/channels/).

::diagram{id=xtm-one-channels-google-chat-setup}

## Before you start

- You have a Google Cloud project and may enable APIs and create service accounts in it.
- Your Google Workspace domain lets you make the Chat app available to your users.
- You are an administrator of XTM One, which runs the Enterprise Edition with AI switched on.
- XTM One is reachable from the internet over HTTPS at its `BASE_URL`: Google Chat posts messages to it.

## Configure the Chat app in Google Cloud

The connection settings need the webhook URL of XTM One: `BASE_URL` followed by `/api/channels/googlechat/webhook`. The **Google Chat** card shows it once connected; you can also read it in its **Setup guide**.

:::steps
1. In the Google Cloud console, enable the **Google Chat API** for your project.
2. Open the Chat API configuration page and set the app name, description and avatar: what your users see in Google Chat.
3. Turn on the interactive features, and choose to receive messages in spaces and direct messages.
4. Under the connection settings, select **HTTP endpoint URL** and paste the webhook URL.
5. Set the visibility to the people or groups who may use the app.
:::

## Create the service account

:::steps
1. Open **IAM & Admin** > **Service Accounts** and create a service account for the Chat app.
2. Create a JSON key for it and download the file. XTM One sends its answers to Google Chat with this key.
3. Copy the project number from the project's overview page: the number, not the project ID.
:::

## Connect the bot in XTM One

:::steps
1. Open :menu[Administration > Settings > Integrations > Channels & Bots] and click **Configure** on the **Google Chat** card.
2. Paste the whole content of the JSON key file in **Bot Token / Access Token**.
3. Paste the project number in **Project number**. XTM One then accepts only requests carrying a token Google signed for your project. Always fill it in.
4. Click **Save Channel**.
:::

::screenshot{id=xtm-one-channels-google-chat-card alt="The Configure Google Chat form inside its card: the service account key in Bot Token / Access Token, the Project number, and the URL to give Google, ending with /api/channels/googlechat/webhook."}

## Turn Google Chat on for an agent

:::steps
1. Open the agent in :menu[Orchestration > Agents], go to **Configuration** > **Channels & Bots** and click **Edit**.
2. Turn on **Google Chat**, then **Receive** so the agent answers incoming messages.
3. Leave **Publish** on if the agent may post on its own.
4. Adjust **Allow anonymous users**, **Allow private conversations** and **Only when mentioned** within what the bot card allows, and click **Save**.
:::

Whitelist entries are space names, in the form `spaces/` followed by the space's identifier. With **Publish** on, the agent can send a message to a space, list the spaces it belongs to, read basic information about a space, and delete one of its own messages. It sends text only: Google Chat lets an app upload a file only on behalf of a signed-in user, which a service account is not.

## Add the app to spaces

In Google Chat, open a space and add the app from the space's menu, or start a direct message with it. The app sees only the spaces it belongs to.

## What to expect on Google Chat

- Each sender is matched to their XTM One account by email, and gets their own permissions, preferences and memory.
- **Only when mentioned** works in spaces, and Google Chat has real threads: once the bot has answered in a thread it follows it without a new mention, and `/stop` mutes that thread.
- There is no typing indicator: a long answer arrives with no sign of activity before it.
- Files uploaded with a message reach the agent, and a file sent without text is still answered. A file over the 40 MB upload limit of XTM One is not passed on, but the message is. A Google Drive file shared in a message reaches the agent as a link only: to read it, give the agent a Google integration (see [Connect agents to your tools with integrations](/xtm-one/build/integrations/)).

## Check that it works

Send the app a direct message in Google Chat. The agent answers, as you.

| What you see | What to do |
| --- | --- |
| Google rejects the endpoint, or never delivers | Check that the Chat API is enabled, the app uses **HTTP endpoint URL**, and the URL is the card's. |
| Every request is rejected | The project number is wrong: use the numeric project number, not the project ID. |
| Answers in direct messages, not in a space | Add the app to the space, check the receive whitelist and whether a mention is needed. |
| The agent cannot post | The service account key is missing or invalid: it signs every answer, separately from the incoming webhook. |

:::example[Paul asks "Night shift triage" in the SOC night space]
The Spanish hotels of Grand Meridian Hotels work in Google Workspace. Hélène Marchetti enables the **Google Chat API** in the `grand-meridian-soc` project, names the app **XTM One**, sets its **HTTP endpoint URL** to `https://xtm-one.grand-meridian.example/api/channels/googlechat/webhook` and makes it visible to the SOC group. She pastes the service account's JSON key and the project number on the **Google Chat** card, then adds the app to the space "SOC night".

Sophie Laurent turns on **Google Chat** for the "Night shift triage" agent, with **Receive** on and **Only when mentioned** on. Paul Garnier writes in the space: "@XTM One which OpenAEV simulation last tested detection of the Velvet Harbor PowerShell loader?" The agent answers in the thread as Paul, and answers his follow-up there without a new mention.
:::

## What's next

- Decide when the bot answers and who it serves: [Talk to agents from your messaging apps](/xtm-one/administer/channels/).
- Connect another app: [Connect Slack](/xtm-one/administer/channels/slack/).
