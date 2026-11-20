---
title: Set up your profile and API keys
description: Set your XTM One profile and two-factor authentication, choose the agent that answers you, review what agents remember or keep it from Platform Health, and create API keys and your MCP endpoint.
doc_type: how-to
products: [xtm-one]
personas: [cti-analyst, developer, ai-builder]
owner: docs-xtm-one
last_reviewed: 2026-11-19
migrated_from:
  - xtm-one:/usage/my-profile/
---

**My Profile** is where you shape XTM One for yourself, apart from the platform's shared **Settings**: your name and time zone, a second factor at sign-in, the AI agent that answers you by default, what agents remember about you, your quota, and the API keys and MCP endpoint that let your own scripts and AI clients work with XTM One as you. Open it from your user menu, at the top right: **My Profile**. The most useful first changes are your time zone, your default chat agent and the General Assistant's tools.

::diagram{id=xtm-one-profile-tabs}

## Your account

The **Profile** tab holds your name, title, email, time zone, theme, language and how you sign in.

- **Time zone**: the agents and the schedule cards use it, so that "tomorrow at 9:00" means 9:00 where you are. Set it first.
- **Email**: another address must not belong to another account. The platform administrator's account, the one `ADMIN_EMAIL` names, keeps that address: saving another one fails with "The platform admin account's email is set by ADMIN_EMAIL and cannot be changed". Change `ADMIN_EMAIL` instead (see the [XTM One configuration](/reference/xtm-one/configuration/)).
- **Password**: shown with a local account. With single sign-on, your identity provider manages it.
- **Work items waiting for you**: the notification bell lists each work item waiting for your decision until you answer it. Turn off **Notify me of work items waiting for me** to take them out of the bell only: the notice of the home dashboard and the **Waiting for me** view of :menu[Work > Operations] still list them.
- **Companion**, for administrators only: turn on **Show the companion**, off by default, and pick a pet. The pet sits in the bottom right corner of every page and shows what Platform Health, the platform's built-in supervisor, and your runs are doing: "A run is waiting for your approval.", "A proposal is ready for your review.", or a run that did not go through. Click it to open its bubble: the open findings of Platform Health ("No open finding.", or how many critical findings or warnings are open), a button to :menu[Administration > Platform Health], named after Platform Health ("Open Platform Health" unless your administrator renamed it), and **Hide**, which turns the companion off until you turn it on again here.

### Turn on two-factor authentication

:::steps
1. On the **Profile** tab, under the password area, click **Set up**. A QR code appears.
2. Scan it with your authenticator app. If you cannot scan, choose **Enter the key manually** and type the key it shows.
3. Type the six-digit code your app shows, and click **Enable two-factor**.
4. Save the 10 recovery codes XTM One shows. This is the only time they appear: each one signs you in once, in place of your app, and nobody can retrieve them for you.

::screenshot{id=xtm-one-profile-two-factor alt="The Two-factor area of the Profile tab: Not enabled, with the Set up button that starts it."}
:::

From then on, signing in asks for the code after your password, or after your identity provider: the second factor applies even when your company's provider does not enforce one. **Use a recovery code**, on the code screen, signs you in without your phone. **Generate new codes** replaces your recovery codes, and **Disable two-factor** asks for your password and a current code. If you lose both your phone and your codes, ask an administrator to reset your enrollment.

### The General Assistant's tools

Under **General Assistant Tools**, override for yourself the built-in tools, integrations, MCP servers, custom tools and knowledge bases the General Assistant uses, or reset them to the platform default. You can add only what you could connect to an agent of your own.

## Choose the agent that answers you

On **Channels & Bots**:

- **Default Chat Agent**: the agent that answers your web chat conversations by default. On XTM One, it starts as the **CTEM Assistant**.
- **External Channels**: for each messaging bot your administrator connected, the agent that answers your private conversations with it, or **Default routing**. Group channels follow the routing an administrator set: see [Talk to agents from your messaging apps](/xtm-one/administer/channels/).

**Agent Accounts** lists the accounts that let agents act in a channel under their own name: see [Give an agent its own account in a channel](/xtm-one/administer/channels/agent-accounts/). **Tool approvals** appears when tool approvals apply to you: it lists the choices you made, so you can take one back.

## Review what agents remember

The **Memory** tab shows what XTM One remembers about you, in two parts:

- **Consolidated Memory**, your cross-agent profile: facts about you only, such as your name, role, time zone, language, writing preferences and key contacts. Every agent reads it.
- **Per-Agent Memory**: what each agent noted while working for you, such as a pending review or a triage rule. No other agent reads it. A note about something that can change is shown to its agent as unverified, with its date, and dropped after 72 hours.

**Purge Profile** clears the cross-agent profile; clearing an agent's memory also removes the notes its work left in your profile; **Purge All** resets everything. Check this tab when an agent remembers something you no longer want it to.

### Opt out of Platform Health

Platform Health, the platform's built-in supervisor, improves the agents their managers opted in. To do so, it reads their memories and the content of their conversations, yours included. Excerpts become evaluation cases, unredacted, that the agent's managers and the administrators can see. You can keep your own content out of it.

:::steps
1. On the **Memory** tab, find the **Platform Health** card. Under **Opt out of Platform Health**, it reads "Your memories and conversations with opted-in agents may be read."
2. Turn on **Opt out of Platform Health**.
3. Confirm with **Opt out**.

::screenshot{id=xtm-one-profile-health-opt-out alt="The Platform Health card of the Memory tab: Opt out of Platform Health, still off, with the line saying your memories and conversations with opted-in agents may be read, and the notice on what Platform Health reads and who sees it."}
:::

The card then reads "Platform Health reads none of your memories and conversations.", and the evaluation cases already built from them are deleted at once. Turning the switch off again lets Platform Health read your content from then on, but does not bring the deleted cases back. Whether an agent is opted in is decided on its page: see [Let Platform Health improve it](/xtm-one/build/agents/#let-platform-health-improve-it).

:::example[Keep guest records out of the evaluation cases]
In May 2026, Sophie Laurent opts the **Velvet Harbor watcher** in to Platform Health. Camille Roux, CTI analyst at Grand Meridian Hotels, has pasted the names of guests whose records Velvet Harbor took into her conversations with the watcher. She opens **My Profile**, tab **Memory**, turns on **Opt out of Platform Health** and confirms with **Opt out**. Platform Health no longer reads her memories and conversations, and the cases built from them are gone. It goes on improving the watcher from the conversations of the colleagues who did not opt out.
:::

## Check your quota

The **Quotas** tab shows your agentic executions against your limit for the period, and where the limit comes from: a rule for you, a rule of your group, or the platform default. When your limit is the platform's shared pool, the gauge shows the whole platform's usage, including requests with no account behind them.

## Create an API key

An API key lets a script or a tool call XTM One as you. It can never do more than your account, and loses what your account loses.

:::steps
1. Open the **API Keys** tab and click **Create Key**. The **New API key** dialog opens.
2. Give it a **Name** that says where it is used.
3. Under **Surfaces**, choose where it may be used: **Platform API** (the REST API under `/api/v1`) and **MCP endpoints** (your agents as MCP tools, and the platform MCP servers). A request elsewhere is refused with `403`.
4. Under **Access**, choose **Read-only** for a key that must never change anything: on the REST API it sends only `GET`, `HEAD` and `OPTIONS` requests; on the MCP endpoints it calls only the tools that read.
5. Optionally, **Limit to specific agents**: the key then reaches only those agents, and nothing else of the platform.
6. Under **Expiry**, choose 7, 30 or 90 days, a year, or a date. **No expiry** is offered only while your administrator has set no maximum lifetime.
7. Optionally, under **Limits**, set its own **Requests per minute** and **Quota** per day or month, on top of your own quotas.
8. Click **Create key**, and copy it now: it is never shown again.

::screenshot{id=xtm-one-profile-api-key alt="The top of the New API key dialog: the Name, and the two Surfaces, Platform API (the REST API under /api/v1) and MCP endpoints, both on."}
:::

Each key in the list shows its prefix, what it reaches (**Full access**, or its surfaces and agents), its expiry, its use of its own quota and its last use. The pencil edits it; deleting it stops everything that uses it at once. You are notified 7 days and 1 day before a key expires. A key created before a maximum lifetime was set and outliving it shows **Beyond the maximum lifetime**: give it an expiry within the limit. A key can create other keys through the API, never one that reaches further than itself.

## Connect your AI client to XTM One

The **MCP Endpoint** tab connects an external MCP client, such as Cursor or Claude Desktop, to XTM One:

- **Endpoint URL**, at `/mcp/user`, exposes your agents as tools. Choose which agents; with none chosen, every agent you can see is exposed.
- **Platform MCP Servers** gives the MCP server of each connected OpenCTI, OpenAEV or OpenCRQ platform, with its address, status and client configuration. A platform not registered yet shows **Not connected**.
- **Platform Tools MCP Server**, when an administrator enabled it, exposes XTM One's own tools (agents, assignments, runs, traces, knowledge bases) to read and fix an agent from your client.

The client authenticates with one of your API keys. See [Use XTM One's own MCP servers from other clients](/xtm-one/build/platform-mcp-servers/).

## Stop your agents at once

Your user menu also holds **Emergency stop**: it cancels every agent activity that runs for you and pauses your assignments. You confirm by typing the code the dialog shows. **Resume my activity**, in the same menu, or **Resume** on the red banner, lifts it. See [Stop every agent at once](/xtm-one/administer/emergency-stop/).

:::example[Set up Camille's profile and a read-only key for her notebook]
On 12 April 2026, Camille Roux, CTI analyst at Grand Meridian Hotels, opens **My Profile**:

| Where | Value |
| --- | --- |
| **Profile**, time zone | `Europe/Paris` |
| **Profile**, two-factor | **Set up** with her authenticator app, recovery codes saved in the company password manager |
| **Channels & Bots**, **Default Chat Agent** | **Velvet Harbor watcher** |
| **Channels & Bots**, agent for Slack | **Velvet Harbor watcher** |

She then creates a key for her analysis notebook, which reads her conversations with **Velvet Harbor watcher**:

| Field | Value |
| --- | --- |
| **Name** | `CTI notebook, read-only` |
| **Surfaces** | **Platform API** |
| **Access** | **Read-only** |
| **Limit to specific agents** | **Velvet Harbor watcher** |
| **Expiry** | 90 days |
| **Requests per minute** | `30` |

She copies the key once into her notebook's secret store. The notebook reads the conversations with the agent; any `POST` it sends, and any request about another agent, is refused.
:::

## What's next

- [Get answers from an agent in the chat](/xtm-one/use/chat/), with your default agent.
- [Use XTM One's own MCP servers from other clients](/xtm-one/build/platform-mcp-servers/), to connect Cursor or Claude Desktop.
- [Configure single sign-on](/xtm-one/administer/sso/), for how you sign in.
