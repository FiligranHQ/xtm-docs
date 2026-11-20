---
title: Connect an MCP server
description: Give your AI agents the tools of another system through the Model Context Protocol, test the connection, and attach the server to an agent with only the tools it needs.
doc_type: how-to
products: [xtm-one]
personas: [ai-builder, admin]
owner: docs-xtm-one
last_reviewed: 2026-11-08
edition: ee
migrated_from:
  - xtm-one:/usage/resources/mcp-servers/
  - xtm-one:/usage/resources/connect-mcp-server/
  - xtm-one:/usage/resources/mcp-catalogs/
---

An MCP server gives your AI agents the tools of another system, through the Model Context Protocol (MCP): an internal API, a sandbox, a search engine, a ticketing tool. You declare the server once in XTM One, test that its tools answer, then attach it to the agents that need it, each with only the tools its job calls for. This page connects a server and attaches it. To let an external MCP client (Claude, Cursor) call XTM One instead, see [Use XTM One's own MCP servers from other clients](/xtm-one/build/platform-mcp-servers/).

::diagram{id=xtm-one-build-mcp-servers-connection}

## Before you start

- Adding or importing a server requires AI to be switched on and the Enterprise Edition. See [What each edition of XTM One brings](/xtm-one/administer/editions/).
- Any user can add a server for themselves. Only an administrator can create a company-managed server, which every user can see and use and only administrators can edit.
- From the provider of the server: its endpoint URL and an API key or an OAuth sign-in, for a remote server; the package or binary to run and the environment variables it reads, for a local one.

## Choose the transport

The **Transport** decides where the server runs and how XTM One talks to it.

| Transport | Choose it when | What XTM One does |
| --- | --- | --- |
| **Streamable HTTP** | The provider hosts the server and gives you a URL, often ending in `/mcp`. The recommended choice for a remote server. | Calls the URL over HTTP, with your headers or OAuth token. |
| **SSE (legacy)** | The provider only offers the older Server-Sent Events endpoint, often ending in `/sse`. | The same, over the older protocol. |
| **Subprocess (stdio)** | The server is a package or binary you run yourself (`npx ...`, `uvx ...`). | Starts the command on the XTM One platform, as a child process of its backend and worker, without a sandbox. |

A local server runs with the permissions of the platform: run only servers you trust, and prefer a remote transport for anything from a third party. A local server cannot go through a relay group.

## Add the server

:::steps
1. Open :menu[Tools & Integrations > MCP Servers]. Under **Connected MCP Servers**, click **Add Server**.

   ::screenshot{id=xtm-one-build-mcp-servers-presets alt="The Add MCP Server page: the search of presets, Custom Server to configure any stdio, SSE or Streamable HTTP server by hand, and ready-made presets such as Atlassian Jira, Brave Search, CISA KEV, CrowdStrike Falcon and Datadog."}
2. Pick a preset, or **Custom Server** to configure any server by hand. A preset fills the transport, the command or URL and the variables it expects, and says where to get the credentials it needs.
3. Enter a **Server Name**. It is a label for people: the model sees each tool under the name the server gives it.
4. For a remote server, enter the **Server URL** (or **SSE URL**). XTM One calls it from its own network.
5. When the server is inside a network XTM One cannot reach, choose a relay group under **Route through** instead of **Direct (from the platform)**. Its requests then leave from that network, and fail rather than fall back to a direct call when no relay of the group is online. See [Reach on-premise tools through a relay](/xtm-one/build/relays/).
6. Choose the **Authentication**:
   - **Secret Headers**, for a static API key: one **Header Name** and **Header Value** per header, for example `Authorization` with `Bearer` and the key. Headers are stored encrypted and never shown again.
   - **OAuth 2.0**, for a provider that asks you to sign in: click **Detect settings**. XTM One reads the server's published OAuth metadata and fills **Authorize URL**, **Token URL** and **Registration URL (optional)**. Leave **Client ID (optional)** and **Client Secret (optional)** empty unless the provider gave you a client: XTM One then registers itself with the provider.
7. For a local server, enter the **Command** (for example `npx`), the **Arguments (space-separated)**, and the **Environment Variables (secrets)** it reads. Put credentials in the environment variables, never in the arguments.
8. Click **Create Server**. The server page opens.

   <!-- shot: xtm-one-build-mcp-servers-new -->
:::

A server that authenticates with OAuth needs one more step: open its **Configuration** tab and click **Authorize** in the **OAuth Connection** card, sign in at the provider, and come back to the server page, which shows **OAuth connected**.

:::warning
By default, the OAuth token belongs to the server, not to each user. Everyone who uses the server calls it with the account that authorized it, as they would share its API key. Authorize with a service account that has only the rights your agents need, or let each person use their own account (next section). On a company-managed server with one shared account, only an administrator can authorize.
:::

If the provider requires a client registered in advance, register `https://xtm-one.grand-meridian.example/api/oauth/callback` (your platform's address followed by `/api/oauth/callback`) as its redirect URI, then enter the **Client ID** and **Client Secret** it issued.

### Let each person use their own account

An OAuth server calls the provider with one shared account, or with the account of each person it acts for. With their own account, the provider applies each person's rights and records each action under their name: a ticket opened by an agent shows the analyst who asked for it, not a service account.

To choose, open the server's **Configuration** tab, click **Edit**, pick the **Connected account**, and click **Save changes**. Only the people who can edit the server choose it.

| **Connected account** | Whose account a call carries | Who connects it |
| --- | --- | --- |
| **One shared account** (default) | The one account authorized for the server, for every person who can use it. | The server's owner, or an administrator on a company-managed server, with **Authorize**. |
| **Each person's own account** | The account of the person the agent acts for: the person asking in the chat, the person an assignment's run acts as. Never another person's account. | Each person, once, with **Connect your account**. |

Changing it discards what the other choice stored. Moving to **Each person's own account** discards the shared connection. Moving back to **One shared account** disconnects every person's own account, and the server has to be authorized again, once for everyone.

On a server that uses each person's own account, the **Configuration** tab shows a **Your account** card instead of **OAuth Connection**, to everyone who can see the server, company-managed servers included. The OAuth badges of the server list and of the server page show your own connection too.

::screenshot{id=xtm-one-build-mcp-servers-your-account alt="The Your account card of the Grand Meridian service desk server, seen by Paul Garnier: Not connected, a note that each person uses their own account on this server and that its tools fail until he connects, and the Connect your account button."}

- **Connect your account** sends you to the provider to sign in, then back to the server page. Tokens are then refreshed automatically.
- Until you connect, the server's tools are called for you without any account, and fail. The error the agent receives tells it that the person the run acts as must connect their account on the server's page.
- **Error** means your connection failed to refresh: click **Connect again**.
- **Disconnect your account** removes your connection. Other people's connections stay.
- **Test connection** runs with your own account: connect it before you test.

:::example[Open service desk tickets under each analyst's name]
Grand Meridian Hotels' service desk publishes its tools over MCP and records who opens each ticket. The SOC wants a ticket opened from a triage chat to name the analyst, not a shared robot account.

Marc Dupuis, an administrator, adds a company-managed server:

- **Server Name**: `Grand Meridian service desk`
- **Transport**: **Streamable HTTP**
- **Server URL**: `https://servicedesk.grand-meridian.example/mcp`
- **Authentication**: **OAuth 2.0**, with **Detect settings**

He opens its **Configuration** tab, clicks **Edit**, chooses **Each person's own account** under **Connected account**, and clicks **Save changes**. Then he clicks **Connect your account** and signs in with his own service desk account, so that **Test connection** answers.

Paul Garnier opens the server from :menu[Tools & Integrations > MCP Servers], clicks **Connect your account** and signs in. The tickets an agent opens for him through this server now carry his name. For a colleague who has not connected, the call fails: it never uses Paul's account.
:::

### Import servers from a client configuration

Already use the server in Claude Desktop, Cursor or VS Code? Click **Import** on the list page, then **From a file** or **From a URL**. XTM One reads the `mcpServers` object of Claude Desktop and Cursor, the `servers` object of VS Code, or an MCP registry `server.json`. Select the servers to create in the review step, then open each one to fill in the values the import could not carry, such as a secret left as a placeholder.

## Test the connection

On the server page, click **Test connection** first, after any change of transport, URL or credentials. It tells you whether the server answers before you look at the agent:

- **Connection successful**, with the number of tools discovered: the server is ready.
- **Authorization required**: the server needs a credential. Set its headers, or switch **Authentication** to **OAuth 2.0** and authorize it.
- An error message, which names the cause: a command not found on the platform, a URL refused by the platform's address policy, a server that redirects elsewhere.

In the **Discovered Tools** card of the **Configuration** tab, read what the server offers, click **Refresh Tools** after a change, and click **Test** on a tool to run it once with arguments you give. Reading the tool list matters more than the server's name: it is what your agent will see.

<!-- shot: xtm-one-build-mcp-servers-tools -->

The URL of a remote server is checked against the MCP server URL policy an administrator sets in :menu[Administration > Settings > Access and security > Security options] (`MCP_SERVER_URL_POLICY`, see the [Security section of the configuration reference](/reference/xtm-one/configuration/#security)). A server the policy refuses shows "This MCP server URL is not allowed": an administrator can list its origin as trusted, or you route it through a relay group.

## Attach the server to an agent

An agent uses only the MCP servers attached to it.

:::steps
1. Open the agent, then its **Configuration** tab, and **Tooling**.
2. In **MCP Tool Servers**, click **Edit**, then click the server to attach it.
3. Click the tool scope button (**All tools** by default), keep only the tools the agent needs, and click **Apply**. Fewer tools give the model a clearer choice and fewer actions it could take.
4. Click **Save**.
:::

Only enabled servers you can see are offered. A company-managed agent can use only company-managed servers. The agent's assignments use its servers and its tool scope. In the chat, a server's **Hide from chat** switch keeps it out of the chat's tool picker while agents can still use it.

If two attached sources publish a tool with the same name, only one of them is offered: narrow the tool scope, or do not attach both.

To see what agents do with the server, open its **Activity** tab. It lists the runs of the agents the server is attached to, with their **Status**, **Type**, **Agent**, **Started** and **Duration**. Pick statuses in the toolbar's status filter (**Running**, **Awaiting input**, **Completed**, **Failed**, **Cancelled**) to find the runs that failed, and click a run to open it.

## Share a server

A server you own can be shared with groups or people, as **Read only** (they attach it to their agents) or **Can configure** (they also edit and test it). Sharing a server lends its credentials: whoever uses it calls the external system with your headers, variables or OAuth token, and XTM One asks you to confirm before it lends them. An OAuth server set to **Each person's own account** never lends yours: each person connects their own. See [Share agents and resources](/xtm-one/build/sharing/).

For a server the whole organization relies on, ask an administrator for a company-managed server, authorized with a dedicated service account or set to each person's own account. **Restrict access** limits it to named groups and people. The read tools of enabled company-managed servers are also the only MCP tools that run without asking on an agent that asks before running tools. See [Govern how AI is used](/xtm-one/administer/ai-governance/).

:::example[Give the triage agent the booking platform's lookup tools]
The hotel booking platform of Grand Meridian Hotels publishes internal tools over MCP, inside the Paris datacenter. Paul Garnier's night shift often needs to know whether a guest booking named in an alert is real. Sophie Laurent connects the server and gives its lookup tools to the **Night shift triage** agent.

In :menu[Tools & Integrations > MCP Servers], **Add Server**, then **Custom Server**:

- **Server Name**: `Hotel booking platform`
- **Transport**: **Streamable HTTP**
- **Server URL**: `https://booking-tools.grand-meridian.example/mcp`
- **Route through**: `Paris datacenter`
- **Authentication**: **Secret Headers**, **Header Name** `Authorization`, **Header Value** `Bearer` followed by the key the booking team issued for XTM One (kept in their vault as `BOOKING_TOOLS_API_KEY`)

**Test connection** answers **Connection successful** with the tools discovered. On **Night shift triage**, Sophie attaches **Hotel booking platform** in **MCP Tool Servers** and keeps only the tools that read bookings in its tool scope. The agent can now confirm a booking during triage, and cannot change one.
:::

## When something does not work

| What you see | What to do |
| --- | --- |
| A command not found on the platform | The command of a local server is not installed in the XTM One images. Ask the platform team to add it, or use a remote server. |
| **Authorization required** | Set the headers, or authorize the server. On a server you do not own, its owner (an administrator, for a company-managed server) authorizes it. On a server that uses each person's own account, click **Connect your account**. |
| The OAuth status shows **Error** | The token could not be refreshed. Click **Re-authorize**, or **Connect again** in the **Your account** card. |
| The server's tools fail for you, not for a colleague | The server uses each person's own account and yours is **Not connected**: click **Connect your account** in its **Configuration** tab. |
| The server is not offered on the agent | You cannot see it, it is disabled, or the agent is company-managed and the server is not. |
| The agent does not use a tool you expect | Check its tool scope, click **Refresh Tools** on the server, and look for another attached source with a tool of the same name. |

Automatic client registration can be turned off by an administrator (`MCP_OUTBOUND_DCR_ENABLED`, see the [MCP OAuth section of the configuration reference](/reference/xtm-one/configuration/#mcp-oauth)): every OAuth server then needs a client ID.

## What's next

- [Create an agent](/xtm-one/build/agents/)
- [Reach on-premise tools through a relay](/xtm-one/build/relays/)
- [Ask an agent in the chat](/xtm-one/use/chat/), then read the server's calls in its **Activity** tab.
