---
title: Connect Microsoft Teams
description: Register the app in Entra ID, create its Azure Bot, connect it in XTM One and install the Teams app package, so your users ask agents questions in Teams.
doc_type: how-to
products: [xtm-one]
personas: [admin, ai-builder]
owner: docs-xtm-one
last_reviewed: 2026-11-08
edition: ee
migrated_from:
  - xtm-one:/administration/channels/microsoft-teams/
---

Connecting Microsoft Teams lets your users ask an agent a question in a team channel, a group chat or a personal chat, and lets agents post messages, cards and files to Teams. Teams has the longest setup of all apps, because it takes three Microsoft pieces: an Entra ID app registration for the identity and permissions, an Azure Bot resource that delivers the messages, and a Teams app package that installs the bot in your tenant. Read [Talk to agents from your messaging apps](/xtm-one/administer/channels/) first for what every app shares.

::diagram{id=xtm-one-channels-teams-setup}

## Before you start

- An Entra ID administrator can grant admin consent for application permissions.
- You can upload a custom app to your Teams tenant, or a Teams administrator does it.
- You are an administrator of XTM One, which runs the Enterprise Edition with AI switched on.
- XTM One is reachable from the internet over HTTPS at its `BASE_URL`: Teams posts messages to it, and Microsoft Graph sends it notifications.

## Register the application in Entra ID

:::steps
1. In the Azure portal, open **Entra ID** > **App registrations** > **New registration** and register the application.
2. On its overview page, copy the **Application (client) ID** and the **Directory (tenant) ID**.
3. Under **Certificates & secrets**, create a client secret and copy its value at once: Entra ID shows it only once. Note its expiry date.
4. Under **API permissions**, add the Microsoft Graph permissions below as **application** permissions, not delegated ones, then click **Grant admin consent**.
:::

| Application permission | Why the bot needs it |
| --- | --- |
| `Channel.ReadBasic.All` | List the channels the bot can work with |
| `Chat.Create` | Start a chat with a user |
| `Chat.Read.WhereInstalled` | Read messages in chats where the app is installed |
| `Chat.ReadWrite.WhereInstalled` | Read and write messages in those chats |
| `ChatMember.Read.WhereInstalled` | Read the members of those chats |
| `Team.ReadBasic.All` | List the teams the bot can work with |

## Connect the bot in XTM One

:::steps
1. Open :menu[Administration > Settings > Integrations > Channels & Bots] and click **Configure** on the **Microsoft Teams** card.
2. Paste the application ID in **App ID / Client ID**.
3. Paste the client secret in **App Password / Client Secret**.
4. Paste the directory ID in **Tenant ID**.
5. Click **Save Channel**. The card shows **Connected**, and its **Webhook URL**, ending with `/api/channels/teams/messages`.
:::

::screenshot{id=xtm-one-channels-teams-card alt="The Configure Microsoft Teams form inside its card: App ID / Client ID and App Password / Client Secret side by side, the Tenant ID, and the messaging endpoint ending with /api/channels/teams/messages."}

**App ID / Client ID** and **App Password / Client Secret** are required: with one of them empty, **Save Channel** does not create the bot and the field reads "This field is required." **Tenant ID** is optional, but fill it in: left empty, the bot signs in through Microsoft's common endpoint and skips the files shared in Teams.

XTM One checks the signed token of every Teams request against this application ID: a request that does not come from the Bot Framework for this application is rejected.

## Create the Azure Bot resource

:::steps
1. In the Azure portal, create an **Azure Bot** resource.
2. Choose **Single Tenant** and **Use existing app registration**, with the application ID of the registration.
3. In **Settings** > **Configuration**, set the messaging endpoint to the webhook URL of the Teams card.
4. In **Channels**, add the **Microsoft Teams** channel.
:::

## Build and install the Teams app package

:::steps
1. Open the [Teams Developer Portal](https://dev.teams.microsoft.com/apps) and create the app.
2. Click **Setup guide** on the Teams card of XTM One and copy the **Example app manifest**. It already carries your platform's name.
3. In the manifest, replace `<YOUR_APP_ID>` with the application ID (in `id`, `bots[0].botId` and `webApplicationInfo`), `your-domain.com` with the domain of XTM One, and the `developer` block with your organization's.
4. Keep the three resource-specific permissions of the table below under `authorization.permissions.resourceSpecific`.
5. Add two icons: `color.png`, 192 × 192 pixels in full color, and `outline.png`, 32 × 32 pixels, a white outline on a transparent background.
6. Zip `manifest.json`, `color.png` and `outline.png` at the root of the archive, with no folder inside.
7. Install the package from the Teams admin center, or upload it to Teams if your tenant allows custom apps. When you change the manifest later, raise its `version` so Teams clients take the new package.
:::

| Resource-specific permission | Why the bot needs it |
| --- | --- |
| `ChannelMessage.Read.Group` | Read channel messages in the teams where the app is installed |
| `ChatMessage.Read.Chat` | Read messages in the chats where the app is installed |
| `ChannelMember.Read.Group` | Receive membership events of the shared and private channels the app is added to |

The manifest uses schema 1.29. Its `supportsChannelFeatures: "tier1"` lets members add the app to shared and private channels; do not also declare `supportedChannelTypes`, or Teams refuses the package with "We can't read the manifest file". Its `supportsFiles: true` lets the bot receive files sent in personal chats; files shared in channels and group chats are read through Microsoft Graph.

## Turn Teams on for an agent

:::steps
1. Open the agent in :menu[Orchestration > Agents], go to **Configuration** > **Channels & Bots** and click **Edit**.
2. Turn on **Teams**.
3. Turn on **Receive**, and under **Restrict to specific channels** pick the team channels it answers in. XTM One reads the teams and channels for you.
4. Leave **Publish** on if the agent may post to Teams on its own, and pick its channels.
5. Adjust **Allow anonymous users**, **Allow private conversations** and **Only when mentioned** within what the bot card allows, and click **Save**.
:::

Group chats are never listed, since Teams does not let a bot list them: **Allow private conversations**, **Only when mentioned** and the agent routing govern them, not the whitelists. A personal chat with the bot is never filtered by the whitelist.

With **Publish** on, the agent can send a message or an adaptive card to a channel, a personal chat or a group chat, edit a message it sent, read channel messages, chat messages and thread replies, list teams, channels and chats, upload a file to a channel, download a file it received, and delete one of its own channel messages.

## Standard, private and shared channels

- In a **standard** channel, Teams delivers every message to the bot: **Only when mentioned** decides.
- In a **private** or **shared** channel, Microsoft delivers only the messages that @mention the bot. Once the bot has answered a mention there, XTM One follows that thread through Microsoft Graph, so follow-ups reach the agent without a new mention. It stops after seven days without activity, and the bot posts a short notice.

Those follow-ups arrive as Microsoft Graph notifications at `BASE_URL/api/channels/teams/graph-notifications`, which Microsoft must reach. Turn on **Require verified integration notifications** in :menu[Administration > Settings > Access and security > Security options], or set `INTEGRATION_NOTIFICATIONS_REQUIRE_VERIFICATION=true` (see the [configuration reference](/reference/xtm-one/configuration/#security-switches)): XTM One then acts only on a notification that proves it comes from the subscription it created. After you turn it on, a maintenance pass (shortly after start, then about every hour) replaces the older subscriptions; if Microsoft refuses a new one, that channel's follow-ups stop until a later pass, and the log says `Graph subscription creation failed`.

## Check that it works

1. Open a personal chat with the bot in Teams and write to it. The agent answers.
2. @mention the bot in a channel of a team where the app is installed. The agent answers in the thread.
3. On the Teams card, **Routing** lists the agent.

| What you see | What to do |
| --- | --- |
| The bot is installed but never answers | Check the messaging endpoint of the Azure Bot, that its **Microsoft Teams** channel is added, and that an agent has **Receive** on. |
| It worked, then stopped | The client secret has expired. Create a new one in Entra ID, paste it with **Edit credentials** and click **Update Channel**. |
| Requests are rejected as unauthorized | The application ID in XTM One is not the one the Azure Bot uses. They must be the same registration. |
| No answer in a private channel | Expected until the bot is @mentioned once in the thread. |
| Mentions are not seen in group chats | Teams sometimes strips the mention; XTM One reads the message again through Graph, which needs the resource-specific permissions in the manifest. |
| Files sent in a chat are not read | Check `supportsFiles` in the manifest for personal chats, and the admin consent of the Graph permissions for channels. |

:::example[Paul asks "Night shift triage" in the SOC team's night channel]
Hélène Marchetti registers **XTM One Teams bot** in the Grand Meridian Hotels Entra ID tenant, grants admin consent to the six application permissions, and creates a client secret valid for 12 months, its expiry date in the SOC calendar. On the **Microsoft Teams** card of `https://xtm-one.grand-meridian.example` she fills **App ID / Client ID**, **App Password / Client Secret** and **Tenant ID**, then clicks **Save Channel**. The Azure Bot's messaging endpoint is `https://xtm-one.grand-meridian.example/api/channels/teams/messages`. She installs the package, with `validDomains` set to `xtm-one.grand-meridian.example`, in the **SOC** team.

Sophie Laurent turns on **Teams** for the "Night shift triage" agent, with **Receive** restricted to the **Night shift** channel of the **SOC** team and **Only when mentioned** on. Paul Garnier writes "@XTM One what do we know about the PowerShell loader Velvet Harbor used in January?" The agent answers in the thread as Paul: Teams gives XTM One his email.
:::

## What's next

- Decide when the bot answers and who it serves: [Talk to agents from your messaging apps](/xtm-one/administer/channels/).
- Let an agent answer from your own Teams account: [Give an agent its own account in a channel](/xtm-one/administer/channels/agent-accounts/).
