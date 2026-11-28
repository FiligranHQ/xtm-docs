---
title: Give an agent its own account in a channel
description: Let an agent answer from a real Microsoft Teams account or Outlook mailbox, yours as a delegate or a dedicated AI user, under rules you set and with your approval if you want it.
doc_type: how-to
products: [xtm-one]
personas: [ai-builder, admin]
owner: docs-xtm-one
last_reviewed: 2026-11-16
edition: ee
no_screenshot: "An agent account needs a Microsoft integration signed in as a work account, which the capture stack does not have: the diagram shows how an account decides, and the screenshots this page needs are proposed for capture."
migrated_from:
  - xtm-one:/administration/channels/agent-accounts/
---

A channel bot is an app people talk to. An agent account is different: the agent operates a real user account, through that account's own Microsoft sign-in, and answers what the account hears in Microsoft Teams chats and channels or in an Outlook mailbox. You choose whose account it is, which messages it answers, what it may use for each audience, and whether a person approves each reply. This page creates an agent account, sets its rules and replies, and follows what it does. For bots, see [Talk to agents from your messaging apps](/xtm-one/administer/channels/).

::diagram{id=xtm-one-channels-agent-accounts-rules}

There are two modes. Choosing one only sets the starting values; every setting stays editable.

| Mode | Whose account | Typical use | Starts with |
| --- | --- | --- | --- |
| **Personal delegate** | Yours | "When someone mentions me or messages me on Teams, let my agent answer." "Let my agent answer the emails it can, and flag the ones I must answer." | **Ask me first**, the AI label, a 2-minute grace period, 60 minutes of silence after you write, 10 replies per conversation and hour |
| **AI user** | A dedicated account everyone knows is an AI, with its own mailbox and calendar | An AI colleague that answers direct messages and follows channels, or runs a shared mailbox | **Send automatically**, no label, no grace period, 30 replies per conversation and hour |

Agent accounts are in **My Profile**, from your user menu at the top right, on the **Agent Accounts** tab. Each user manages their own; administrators see every account with **All accounts**.

## Before you start

- **A Microsoft integration signed in as the account.** Add one in :menu[Tools & Integrations > Integrations], signed in as the account the agent operates: yourself for a delegate, the service account for an AI user (see [Connect agents to your tools with integrations](/xtm-one/build/integrations/)). It must be your own integration, or one shared with you with its owner's consent. It already carries the scopes the agent needs: `Chat.Read`, `ChatMessage.Send`, `ChannelMessage.Read.All`, `ChannelMessage.Send` and `User.ReadBasic.All` for Teams; `Mail.Read`, `Mail.ReadWrite` and `Mail.Send` for Outlook.
- **A work or school Microsoft account.** Microsoft Graph does not offer these APIs for personal accounts.
- **An agent you trust to run for other people.** Messages come from others, who can write anything: prefer an agent with read-only tools and no destructive ones, and keep approval on for a delegate until you trust it. See [Create an agent](/xtm-one/build/agents/).
- **A public HTTPS address for real time.** Microsoft notifies only an HTTPS address built from `BASE_URL`. Without one, chats and the inbox are checked every minute, and channels cannot be heard.
- The Enterprise Edition with AI switched on: otherwise the account hears messages and records each one as `AI unavailable`.

One platform account is operated by one enabled agent account at a time. A Microsoft integration can therefore run a Teams account and an Outlook mailbox together, but not two Teams accounts: saving a second one is refused. Put the extra behavior in the rules of the first one.

## Create an account

:::steps
1. Open **My Profile** from your user menu, at the top right, then the **Agent Accounts** tab, and click **New account**.
2. Choose **Personal delegate** or **AI user**.
3. In **General**, set **Name**, pick the **Platform** (**Microsoft Teams** or **Microsoft Outlook**; it cannot change later), the **Account (integration)** signed in as the account, and the **Agent** that answers. Under the integration, the editor shows who it is signed in as, and warns when another agent account already operates that account.
4. Review **Rules**, **Replies** and, if needed, **Schedule**, as [Rules](#rules) and [Replies](#replies) describe.
5. In **Test**, type a message and a sender to see what the account would do with it. Nothing is sent.
6. Click **Create**.

   <!-- shot: xtm-one-channels-agent-account-editor -->
:::

The account then reads who it is, subscribes to the conversations its rules reach, and starts listening. Its card shows **Setting up**, then how it listens: **Real time** or **Polling every minute**. Microsoft confirms each subscription by calling XTM One back, which can take a few seconds per conversation.

## Operate a service account

You can operate a service account you hold the credentials of from your own XTM One account. Connect a second Microsoft integration signed in as the service account, name it after it, create an **AI user** account with it, and check under **Account (integration)** that the editor shows the service account, not you. To give the agent the account's mailbox and calendar, bind Microsoft on the agent.

In that account's replies the agent works as the service account:

- Its Microsoft tools act as the account, never as you. An agent without a Microsoft binding gets no Microsoft tools: the account changes who a tool acts as, not which tools the agent has.
- The agent is told who wrote, as Microsoft reports it, not that it talks to you. Your personal memory and your default knowledge bases and MCP servers are left out.
- Two agent accounts never answer each other, so two of your accounts cannot reply to each other in a loop.

Every reply still counts against your agentic quota, and its tools run with your permissions.

## Rules

Rules are read top to bottom, and **the first rule that matches a message decides**. A message no rule matches is left alone. A new account starts with two rules: direct messages, always answered, and mentions in groups and channels.

| Setting | What it decides |
| --- | --- |
| **Where** | **Direct messages**, **Group chats**, **Channels**, in any combination |
| **Specific conversations** | Chosen from the account's own chats and channels. Empty: every conversation of the kinds above |
| **Only these people** | Focus the rule on some people, from the directory or typed as an email |
| **Members of these groups** | XTM One groups: the rule also covers a sender whose Microsoft address is the email of an active member. With no people and no group, the rule covers anyone |
| **Except these people** | People this rule never answers |
| **Trigger** | **Only when the account is mentioned** or **Every message (full monitoring)**. A direct message always counts as addressed. In a channel, a rule waiting for a mention hears only the mentions, unless another rule answers every message there |
| **Action** | **Answer**, or **Ignore (stop here)**: an ignore rule placed first excludes a conversation or a person entirely |
| **Reply** | **Always reply**, or **AI decides whether to reply**: the agent reads the message and speaks only when its persona says it should |
| **Follow the thread** | Once the agent answered in a thread, later messages there count as addressed and the agent decides. In a channel heard for mentions only, the thread is checked every minute for a day, so a follow-up can take up to a minute to be seen |

A rule can override the account's response mode, AI label and grace period, and add **Guardrails and instructions for this rule** ("for my manager, be formal and never commit to a date"). The editor offers ready-made rules: **Monitor a channel (AI decides)**, **Focus on some people**, **Ignore a conversation**, **Ignore some senders**, **Public channels, non-confidential only** and **Leadership groups, full access**.

### Limit what the agent uses for each audience

An agent with deep reach should not answer everyone with everything. In **Access for this rule**, each rule decides what the agent may use for the messages it covers:

- **Integrations, knowledge bases, MCP servers, built-in tools, custom tools and skills.** Everything the agent has is on by default; switch off what this audience must not reach. XTM One enforces it: a knowledge base switched off is not searched, an integration switched off gives no tool. A capability the agent gains later is on for a rule that left the list untouched, never for one that narrowed it.
- **Use the agent's memory.** Off: the agent neither reads nor writes its memory for these messages, so what it learned in a board chat cannot surface in a public channel.
- **Guardrails.** Words that govern what the agent says to that audience. The model follows them; access limits are enforced by the platform. Put what must not leak behind an access limit, and use guardrails for tone and what to decline.

Because the first matching rule decides, **order is the policy**: put channel rules first. Someone asking in a public channel then gets that channel's access, whoever they are, because everyone in the channel reads the answer. In **Test**, type an email as the sender to see which rule applies, whether it limits access, and which of the rule's groups the sender belongs to.

## Mailboxes

A **Microsoft Outlook** account watches the account's inbox. Its kinds of conversation are **Emails to the account alone** and **Emails with other recipients**, and its triggers **Only when the account is in To** or **Every email, Cc included**. A thread of replies is one conversation. **Only these people** and **Except these people** also take a whole domain, written `@example.com`.

For each email a rule hands it, the agent answers, hands it to you, or leaves it, and says why in one line:

| The agent | What happens | Default in the mailbox |
| --- | --- | --- |
| Answers | The reply goes out as the response mode says | Marked as read, filed in the `Answered by AI` category |
| Hands it to you | Nothing is sent; the notification bell tells you why | Flagged, filed in `Needs your attention` |
| Leaves it (a thank-you, a confirmation) | Nothing is sent | Filed in `No reply needed` |

Change these in **Replies** > **Mailbox triage**. With **Always reply**, the agent answers or hands over, never leaves an email. The agent replies in the email's thread, to the sender or to everyone, with the original quoted below. A reply that would reach more people than **People a reply may reach** (10 besides the sender, by default) is handed to you instead, and a mailbox account starts with **Agent turns per hour, whole account** at 60. **Replies** > **Signature** chooses the signature: the account's own (copied from its latest sent emails, the default), one read from a signature integration, or none when your mail server signs in transit.

One email gets one answer from an agent: when an assignment of the same agent that runs on incoming emails already took it, the account leaves it, as **Handled elsewhere**.

The account leaves alone its own emails, drafts, meeting invitations, emails from any agent account, and automated mail: newsletters, mailing lists, notifications, auto-replies and out-of-office messages.

## Replies

| Setting | What it does |
| --- | --- |
| **Response mode** | **Send automatically**; **Ask me first**: replies wait for your approval; **Shadow (record only)**: nothing is posted, so you see what the agent would say while you tune the rules |
| **AI label** | A line added before or after every reply, by XTM One, never by the model. `{name}` stands for the account's name |
| **Grace period (seconds)** | Wait before answering; if the account's human answers meanwhile, the agent stays silent |
| **Step back after the owner writes (minutes)** | Once the human writes in a conversation, the agent stays out of it this long. `0`: never |
| **Replies per conversation and hour** | A safety limit against loops, for instance between two AI accounts. `0`: no limit |
| **Agent turns per hour, whole account** | Every message the agent takes up costs a turn, answered or not, in any conversation. Past the limit, the account leaves the next messages until the hour ends, with the reason `Hourly limit of the account reached`. `0`, the default of a Teams account: no limit |
| **People a reply may reach** | Outlook only. How many people a reply may reach besides the sender (10 by default). A reply that would reach more, a reply-all to a long list, is handed to you instead of sent. `0`: no limit |
| **Context messages** | How many earlier messages of the thread the agent reads before answering (20 by default) |
| **Instructions** | Added to the agent's persona for every reply of this account |
| **Reactions as feedback** | On by default: a reaction on an answer rates it in the agent's **Feedback** tab (thumbs up positive, thumbs down negative), for 14 days after the answer. Only a keyed fingerprint of who reacted is kept |
| **Notify me when a reply is sent** | On by default: each reply the account posts on its own reaches your notification bell, with where and what was said |

Teams and Outlook show none of the cards of the XTM One chat. When a reply includes a deck, a report, a video or a dashboard, the account posts instead a link to its page in XTM One, named after its title; a link to a deck opens on the slide the agent pointed to. A file goes out as an attachment. Something that is not yours is left out of the reply, never linked.

In a Teams chat or channel thread, the agent answers one turn at a time. It waits a few seconds before it starts (6 by default, `CHAT_EVENT_COALESCE_WINDOW_SECONDS` in the [configuration reference](/reference/xtm-one/configuration/)), and messages written meanwhile are **Queued**. It then answers them in one reply, on the latest one: the others show **Answered together**. A burst of five lines from Paul gets one answer, not five.

With **Ask me first**, each reply waits in **Replies waiting for your approval** at the top of **Agent Accounts**, and the notification bell tells you. Send it as written, edit it first, or drop it. A reply nobody approves expires after a day.

**Schedule** sets weekly windows with a **Timezone**: **During these hours** answers only inside them, **Outside these hours (when I am away)** only outside them, for instance evenings and weekends for a delegate. Type each window's start and end on a 24-hour clock, such as `20:00`, or pick them from the clock beside the field. A time that does not exist, such as `25:99`, shows "Enter a time as HH:MM, from 00:00 to 23:59." and the window keeps its previous time. A window that ends before it starts runs past midnight.

## Follow what the account does

The account's name, or its **Activity** button, opens its page, with what it heard over the last 30 days:

- **Overview**: over 24 hours, 7 days or 30 days, how many messages it heard, what it did with them, the share it answered, the time to answer and to approve, and the ratings of its answers. A **Handled elsewhere** card appears when another path of the same agent took some of them. The **Rules**, **Conversations** and **People** tables can be searched, sorted and exported; a conversation or a person opens **Decisions** on it.
- **Decisions**: every message, newest first, with why it was handled that way (`No rule covers it`, `The account is not mentioned`, `The account owner took over`, `Hourly reply limit reached`, `Hourly limit of the account reached`...), the rule, and the reply. Filter on one or more **Outcome** and **Kind of conversation**, or search.
- **Messages sent**: every message the AI sent in the account's name. **Kind** separates **Replies to a message** from **Proactive sends**; **Status** keeps **Sent**, **Waiting for approval**, **Recorded (shadow)**, **Failed**, **Rejected** or **Expired** messages. Click a row to read the whole message, and send or drop a draft waiting for you there.

In the tables of these three tabs, **Columns** hides the columns you do not need. Your browser keeps the hidden columns and the page size you choose for each table.

### See what the agent sent on its own

The agent can also write through the account's Microsoft sign-in outside its rules: a chat where you ask it to post in a channel, an assignment that emails a summary, a tool call. XTM One logs each of these in the enabled accounts on that integration, so you see everything the AI sent in the account's name. In **Messages sent** and **Decisions**, such a message carries **Proactive send**. Opened, it says "Sent by the agent on its own, not in answer to a message.", lists its recipients after **To:**, and links to what produced it: **Agent**, **Conversation**, **Run** and **Assignment**, when known. A proactive send that failed shows **Failed**.

You can also manage your accounts from a chat: ask the General Assistant, or an agent with the agent builder built-in tool, to "pause my Teams delegate until Monday" or "why didn't you answer Julien yesterday?". An agent that operates one of your accounts can change that account the same way. These tools are never offered while the agent answers someone writing to the account. To keep an agent from changing its accounts, add `update_agent_account`, `edit_agent_account_rule` and `delete_agent_account` to its tool deny list.

## When the account does not hear a conversation

The account card says how many conversations it cannot hear (**Not listening to 2 of 14 conversations**), why, and what to do, with a button that does it; **Technical details** shows Microsoft's answer for support. Once fixed, click **Synchronize now** or wait for the next synchronization, within the hour. The account keeps answering everywhere else.

| The card says | What to do |
| --- | --- |
| The account's integration cannot be used | Sign the integration in again as the account, or pick another one |
| Microsoft Teams rejected the integration's sign-in | The sign-in expired or was revoked: sign the integration in again |
| Microsoft Teams refused to notify the account | Get the permission the card names granted in Entra ID (`Chat.Read` for chats, `ChannelMessage.Read.All` for channels), or add the account back to the conversation, then synchronize |
| Another agent account already follows it | Microsoft allows one subscription per application and conversation: remove the conversation from one account's rules, or connect one of them through another Microsoft application |
| Microsoft Teams could not reach the platform to confirm the subscription | Make `BASE_URL/api/v1/agent-accounts/notifications/microsoft_teams` reachable from the internet over HTTPS |
| Channels cannot be heard without a public HTTPS address | Set `BASE_URL` to a public HTTPS address (see the [configuration reference](/reference/xtm-one/configuration/#urls)) |
| The conversation no longer exists, or the account left it | Remove it from the rules, or add the account back |
| The rules cover it, but the account does not listen to it | A rule answering in every channel follows the first 50 channels of the account, and none in a team whose channels Microsoft Teams does not list to the account. Name the channel in a rule, or pick it again in the rule if the rule names it without its team |
| Microsoft Teams asked the platform to slow down, or did not answer | Nothing: the next synchronization tries again |

## Keep it safe

- Messages from other people are untrusted input to an agent that acts as you. The prompt-injection checks of XTM One apply, and a tool that needs approval stops the reply instead of posting. The safest setup is a delegate with **Ask me first** and an agent with read-only tools.
- Narrow each rule's **Access for this rule** for the audiences that must not reach a knowledge base, an integration or the agent's memory.
- Group membership is read at every message: someone removed from a group stops reaching what the group could at their next message.
- Every created, changed, deleted, approved and rejected account or reply is recorded in the audit log.

:::example[Paul asks the SOC's AI colleague in the Teams night channel]
Grand Meridian Hotels gives its night shift an AI colleague in Teams: the service account `soc-night-assistant@grand-meridian.example`, shown in Teams as **SOC Night Assistant**. Sophie Laurent connects a Microsoft integration signed in as that account, named `[SOC Night Assistant] Microsoft`, then in **My Profile**, on the **Agent Accounts** tab, clicks **New account** and chooses **AI user**:

- **General**: **Name** `SOC Night Assistant`, **Platform** **Microsoft Teams**, **Account (integration)** `[SOC Night Assistant] Microsoft` (the editor shows **Signed in as** the service account), **Agent** "Night shift triage".
- **Rules**: first **Public channels, non-confidential only** on the SOC team's **Night shift** channel, with the "Grand Meridian security policies" knowledge base switched off and **Use the agent's memory** off; then the default direct messages and mentions rules.
- **Replies**: **Send automatically**, **Replies per conversation and hour** `30`.
- **Schedule**: **During these hours**, **Timezone** Europe/Paris, every day from 20:00 to 07:00.

At 02:14 Paul Garnier writes in the **Night shift** channel: "@SOC Night Assistant is 203.0.113.10 linked to Velvet Harbor?" The agent answers at once from the account, from what OpenCTI knows, and the **Decisions** tab of the account shows the rule that answered.
:::

## What's next

- Choose the tools and knowledge of the agent that answers: [Create an agent](/xtm-one/build/agents/).
- Connect a bot people talk to instead: [Connect Microsoft Teams](/xtm-one/administer/channels/microsoft-teams/).
