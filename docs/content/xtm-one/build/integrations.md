---
title: Connect agents to your tools with integrations
description: Connect a service such as Jira, Slack or Microsoft 365 once, bind it to an agent with only the tools it needs, and share it without lending more than you intend.
doc_type: how-to
products: [xtm-one]
personas: [ai-builder, admin]
owner: docs-xtm-one
last_reviewed: 2026-11-19
edition: ee
migrated_from:
  - xtm-one:/usage/resources/integrations/
---

An integration connects XTM One to a service your team works in, such as Jira, Slack, Microsoft 365, GitHub or a threat intelligence feed, with its credentials. An AI agent bound to it gets the service's tools: it reads tickets, posts in a channel, searches mail. This page connects a service, shows what it gives an agent, binds it with only the tools the agent needs, and shares it safely. The OpenCTI, OpenAEV and OpenCRQ registered with XTM One are integrations too, created for you.

::diagram{id=xtm-one-build-integrations-binding}

## Before you start

- Creating an integration requires the Enterprise Edition. Integrations keep working when an administrator switches AI off, for the flows and syncs that use no model. See [What each edition of XTM One brings](/xtm-one/administer/editions/).
- Any user can create an integration for themselves. Only an administrator creates a company-managed one, which every user can use and only administrators edit.
- For an OAuth service, an administrator may already have registered the OAuth app of your organization in :menu[Administration > Settings > Integrations > OAuth Defaults]: you then only sign in. Otherwise, you need the client ID and secret of your own OAuth app, or an API key.
- Talking to an agent from Slack or Microsoft Teams is a different connection, a channel bot set up by an administrator: see [Talk to agents from your messaging apps](/xtm-one/administer/channels/). An integration gives an agent the tools of the service.

## The integrations of the registered modules

When an administrator registers OpenCTI, OpenAEV or OpenCRQ with XTM One, the registration creates a company-managed integration named after the platform, for example **Grand Meridian OpenCTI**. Its **Authentication Mode** is **Managed by the platform**: it stores no credential, and every call carries a short-lived token signed for the person the agent works for. An agent bound to it reads and writes in OpenCTI with that person's rights, never more.

Its configuration is kept up to date by the registration and cannot be edited, with a few exceptions. An administrator changes them with **Edit** on the **Configuration** card, then **Save changes**, and the next registration keeps them:

- On a multi-tenant OpenAEV, the **Tenant ID** makes every OpenAEV tool work in that tenant.
- On OpenCTI, the **Webhook secret**, and the Live writes field, which decides where the knowledge changes of agents go. Off, the default, agents stage their changes in a draft that a person reviews in OpenCTI. On, their changes go to the live graph with no review.

With Live writes on, agents also get the tools that validate a draft and that create, update or delete labels, and they can delete any draft, not only those of their run. The setting is the default of the bound agents: each agent can override it on its **Tooling** tab, as [Create an agent](/xtm-one/build/agents/) shows.

In a run no person is watching, such as an assignment or the background build of a CTEM cycle, XTM One does not send OpenCTI a request the person's role cannot pass. The tool answers that the request was not sent, and the run goes on, so the OpenCTI audit log records no refused access in that person's name. In a chat, the request goes to OpenCTI, since the person asked for it.

What XTM One does in each module is in [XTM One in OpenCTI](/xtm-one/use/in-opencti/), [XTM One in OpenAEV](/xtm-one/use/in-openaev/) and [XTM One in OpenCRQ](/xtm-one/use/in-opencrq/).

## The built-in web search

When the deployment sets a Brave Search or a Google Search API key, XTM One creates one company-managed integration that searches the web, named Brave Search or Google Search. It is read-only, and the built-in assistants, such as the OpenCTI, OpenAEV and OpenCRQ assistants, are bound to it. With both keys set, Brave Search wins. Google stops its Custom Search on 1 January 2027: see [Built-in web search on Brave Search](/deploy/upgrade/breaking-changes/#xtm-one-1-261116-0), and the keys in the [configuration reference](/reference/xtm-one/configuration/).

## Connect a service

:::steps
1. Open :menu[Tools & Integrations > Integrations] and click **New Integration**.
2. Search the catalog, or narrow it with the filters on its left, and pick the service. A filter whose name is too long for the column ends with an ellipsis: hover it to read it whole. **Certified** marks the integrations Filigran qualifies and supports: the Filigran products, and the services its own agent fleets run on every day in production. Among them are Jira, GitHub, Slack, Microsoft 365, Google, Notion, HubSpot, Microsoft Defender and HackerOne. **Preview** marks the others. A provider an administrator switched off shows **Disabled by your administrator**.
3. Enter a **Name** your colleagues will recognize, and a **Description** of what the connection reaches. Agents read the description as context for its tools, so say which workspace, site or projects it covers.
4. Fill in the credentials the **Authentication mode** asks for:
   - **OAuth**: keep **Use admin pre-configured credentials** when your administrator set up the app, or enter your own **Client ID** and **Client Secret**. Register your platform's address followed by `/api/oauth/callback` as the redirect URI of your own app.
   - **API key** or **Bot token**: paste the key the service issued for XTM One.

   The line under each credential field says where your account of the service shows it, and refers to the **Setup guide** when the provider has one.
5. For a service inside your network, choose its relay group in **Route through**. A service reached only through a relay, such as an LM Studio model server, offers no **Direct (from the platform)** choice: **Create integration** stays unavailable until you pick a group. See [Reach on-premise tools through a relay](/xtm-one/build/relays/).
6. Click **Create integration**, then, for OAuth, click **Authorize** and sign in at the service with the account the agents will act as. Accept every permission the consent screen asks for.
7. Click **Test connection**. A **Connection successful** answer means the credentials work.
:::

The status badge of each card on the list page tells you at a glance whether a connection is **Connected**, **Not tested**, **Pre-configured** but not authorized yet, **Not connected**, or in **Error**. An API key or a bot token entered at creation, or replaced later, reads **Not tested** until someone clicks **Test connection**. A successful test turns it **Connected**, a failed one **Error**. Testing your own account of an integration whose members each connect theirs leaves the badge as it is. A provider's card shows the worst status of its integrations. **Re-authorization required** means its permissions changed, the service refused its token, or the account that signed in declined a permission on the consent screen. In that last case, the **Overview** tab lists the missing ones under **Requested scopes not granted**: the calls that need them fail until someone clicks **Re-authorize** and accepts all of them. An OAuth integration that holds no token reads **Not connected**, whatever was stored before.

::screenshot{id=xtm-one-build-integrations-list alt="The Installed section of the Integrations page: Filigran AI marked Preview, and OpenAEV, OpenCRQ and OpenCTI marked Certified, all four Connected, OpenCTI with its Events badge."}

## See what it gives an agent

Open the integration's **Configuration** tab. The **Available Tools** card splits its tools into **Read Tools**, which retrieve information, and **Write Tools**, which change something in the service. That split is how you judge the risk of binding it. Hover a tool and click **Test** to try it with the real credentials before any agent does.

Some write tools reach far. The Microsoft Intune integration creates and updates configuration policies and remediation scripts, runs a remediation on a device, and assigns a policy, script or app to groups, to every device or to every user. The Microsoft integration posts in Teams chats as the signed-in account, and adds or removes an emoji reaction on any message of a chat that account belongs to. It also renames and moves OneDrive and SharePoint files and folders, sends them to the recycle bin, and converts a Word, PowerPoint or Excel file to PDF, kept in the conversation or also saved beside the original. The OpenAEV integration reads and changes OpenAEV's runtime settings, such as the inject execution threshold, except a setting its environment variable pins. The OVHcloud integration reboots, starts and stops bare metal servers, Public Cloud instances, VPS and Hosted Private Cloud virtual machines. The Grafana integration creates, updates and deletes Grafana-managed alert rules. Keep such tools for the agents whose task needs them.

The XTM Hub integration gives one tool per operation of the XTM Hub API, each acting with the personal API token the integration holds. The tools of its queries read and those of its mutations write, so a read-only binding keeps the reads.

The GIPHY integration, connected with an API key from the GIPHY Developers dashboard, gives four read-only tools: search GIFs and stickers, the best GIF for a phrase, a random GIF for a tag, and the trending ones. The agent puts the GIF in its answer: the web chat shows it, and Teams, Slack and the other messaging channels receive it as an image. Every call asks for a content rating, G by default and never above PG-13. Say in the agent's instructions when a GIF fits, such as a resolved incident.

::screenshot{id=xtm-one-build-integrations-tools alt="The Available Tools card of the OpenCTI integration: the Read Tools group first, each tool with its description and its Read badge."}

On the same tab:

- **Ask for approval before running its tools** makes an agent that asks before running tools stop for a person's decision on every tool of this integration. See [Govern how AI is used](/xtm-one/administer/ai-governance/).
- For a service that sends events, a card says which changes an agent can react to and how they arrive. New Slack messages, and new Microsoft 365 emails and Teams messages, are read every 5 minutes with no setup. For Slack messages in real time, set the URL the card shows as the Request URL of your Slack app, and store the app's Signing Secret in the integration's **Signing secret** field (or in the Slack OAuth default, when your administrator provides the app). Store the secret before you enter the URL.
- A Jira integration wakes the assignments that react to Jira: new issues, changes, transitions, reassignments and comments are read every minute. For a Jira integration connected with OAuth, XTM One also registers a Jira webhook, so that a change arrives the moment it is made, when the platform's address is a public `https` URL Jira can call. [Create an agent](/xtm-one/build/agents/) sets up these assignments. When an agent mentions a Jira user in a comment, a description or a worklog it writes, the mention is a real one, and Jira notifies that person.
- A Wordsmith integration wakes the assignments that react to Wordsmith events, such as a task assigned, a review completed or an input request from a Wordsmith agent. For each enabled assignment, XTM One creates a signed webhook endpoint in Wordsmith, checks it every 6 hours, and deletes it when the assignment stops reading Wordsmith. Every 15 minutes it also reads the Wordsmith event log, so a delivery Wordsmith gave up on still runs once. Connected with OAuth, XTM One registers itself with Wordsmith: there is no client ID to enter, but Wordsmith must first allow the platform's redirect URI. Until then, use an API key.

The **Overview** tab says why a connection works or does not (authentication, authorized and missing scopes). Its **Who can use it** card lists everyone who reaches the integration: look there before you change a shared integration. See [Share it](#share-it).

## Bind it to an agent

:::steps
1. Open the agent, its **Configuration** tab, then **Tooling**.
2. Under **Integrations**, click **Edit**, then click the integration to bind it.
3. Click the tool scope button next to it (**All tools** by default), keep only the tools the agent needs, then click **Apply** and **Save**.
:::

A company-managed agent can bind only company-managed integrations. To make each person who talks to a shared agent use their own account of a service, rather than one connection for everyone, use **Advanced: Dynamic integrations (per-user)** in the same card.

An agent bound to a HiBob integration also knows who it is talking to. In a conversation, XTM One looks the person's email up in HiBob, through that binding and with its permissions, and gives the agent their name, title, department, site and time zone, manager, direct reports and start date. No other field of the directory is read, such as pay or personal contacts. The profile is kept 24 hours per person, a person missing from HiBob is named as such, and when HiBob does not answer the conversation goes on without it. A turn that answers another agent gets no profile.

Two more settings narrow what the agent does with an integration:

- **Read-only**: an integration can be made read-only for one agent, which then keeps only the tools that read and is refused any write, whatever its prompt says. [Create an agent](/xtm-one/build/agents/) shows how.
- **Scope(s)**: select the repositories, channels or object types the agent works on. By default a selection is **Guidance only**: the agent is told to focus on them, but its tools can still reach other elements. For GitHub repositories, Slack channels, Microsoft Teams channels and HubSpot object types, turn on **Enforce as a boundary**: the card then reads **Enforced**, and the tools cannot reach anything else, in chats and in assignments. Other selections, such as Jira projects, cannot be enforced: restrict the integration's own credentials instead.

## Share it

A shared integration lends its credentials. Everyone it is shared with, and every agent, assignment, knowledge base source or skill that carries it, acts with the account that authorized it: in Jira or Slack, their actions appear as that account. Before a change lets new people act with your credentials, XTM One names the credential and who will use it, and asks you to confirm.

For an OAuth integration, the owner (or an administrator) can turn on **Don't share my connected account**. The integration stays shared, but everyone it is shared with connects their own account in the **Your account** card of its page (**Connect your account**), and each chat or run acts with the account of the person it works for. Someone who has not connected theirs gets no tools from it, and a note under the agent's answer says so.

The **Who can use it** card of the integration's **Overview** lists every way its credential reaches people, in one list. Its first line names whose account the integration acts with and who can use it. Then come:

- **Direct shares**: the integration shared with the company, a group or people, and who can use it that way.
- **Agents bound to it**: each agent once, with its owner, who it reaches, and what runs with the integration (chats, assignments, its public link, messaging channels). Click a row to open the agent.
- **Through other entities**: the assignments, knowledge base sources, skills and other resources that carry it.

::screenshot{id=xtm-one-build-integrations-who-can-use-it alt="The Who can use it card of the OpenCTI integration: the Direct shares section, then the Agents bound to it section with Crisis briefer, Night shift triage and Velvet Harbor watcher, each with its owner."}

An agent's binding works only while its owner can see the integration. When the owner loses access, because a share was revoked or they left a group, the agent stays bound, but every run refuses the integration's tools: chats, assignments, its public link and channels. The card marks it **Owner no longer has access**, and someone who manages the agent clicks **Unbind** to remove the binding. A company-managed agent is never cut off this way. Before you save a change of sharing that would cut an owner off, the confirmation lists their agents under **These agents would lose this integration**: check **Unbind them too** to remove those bindings in the same step.

How sharing works for every resource is in [Share agents and resources](/xtm-one/build/sharing/).

## Let a partner's agents work with yours over A2A

The agent-to-agent (A2A) protocol lets an agent outside the platform, at a partner or on another XTM One, send work to one of your AI agents, or lets yours hand work to it. An A2A integration holds that trust: **OAuth A2A** for any partner that can run an OAuth exchange, **Foundry A2A** for Azure AI Foundry partners and **Google Agent Platform A2A** for Google partners. A2A is available only where chatting with agents is, and never while AI is switched off.

The **Direction**, chosen at creation, decides what binding the integration to an agent does. It cannot change while an agent binds the integration.

- **Inbound: a partner tasks our agents**: binding it exposes the agent to the partner. The partner's work runs as the agent's owner, or as the user an administrator chose for a company-managed agent, and sees what that user sees. It uses the agent's whole tool set, and that of the agents it hands work over to: bind only agents whose tools you accept to lend.
- **Outbound: our agents task a partner**: you enter the partner's agent card URLs, its token endpoint, and the client ID and secret it issued you. Each partner agent XTM One verifies becomes one tool the bound agent uses to delegate work. In a chat, the agent can continue the partner's task in a later message of the same conversation: answer the partner's question, or collect its result.

### Give a partner its credential

An inbound **OAuth A2A** integration carries one credential, issued by XTM One: a client ID and a secret the partner exchanges for a short-lived token. It works on every agent the integration is bound to. Generating it requires the right to manage the integration.

:::steps
1. Open :menu[Tools & Integrations > Integrations], click **New Integration**, and pick **OAuth A2A**.
2. Enter a **Name**, choose **Inbound: a partner tasks our agents**, and click **Create integration**.
3. On its **Configuration** tab, click **Generate credential** in the **Partner credential** card. Copy the client ID and the secret at once, or both with **Copy both**: the secret is never shown again.
4. Bind the integration to each agent the partner should reach, as in [Bind it to an agent](#bind-it-to-an-agent).
5. Send the partner the client ID, the secret, the agent's address from the **Agent card URLs** card, and the **Token URL** of the **Partner credential** card. The token URL shows once a bound agent publishes its card. A partner on XTM One pastes it into its **Partner token endpoint** field.
:::

The **Partner reachability** card then reads **Ready. This partner has not connected yet.**, or names what still blocks the partner: no credential, no agent bound, or a bound agent with no published card.

::screenshot{id=xtm-one-build-integrations-a2a-credential alt="The Partner credential card of an inbound OAuth A2A integration: the client ID with its copy button, the Regenerate secret button at the top, and Rotate client ID and secret below."}

When the secret is lost or leaked, click **Regenerate secret**. The client ID stays the same: the partner pastes the new secret only. The old secret stops working at once on every bound agent, and so do the tokens the partner obtained with it. To change the client ID too, click **Rotate client ID and secret**: both change, and the partner pastes the new pair.

When the partner's token request fails, the card shows the **Last refused token request**, with its date, the client ID presented and the reason:

- **Wrong secret for the current client ID**: the partner must paste the latest secret.
- **Client ID rotated away**: the partner presents a client ID you rotated, and must paste the current client ID and secret.
- **No agent exposed**: the credential is valid, but no agent bound to the integration is exposed.

Only the people who manage the integration see this panel: the partner receives the same refusal whatever the reason.

:::example[Let Example CERT's triage agent task the Velvet Harbor watcher]
Example CERT runs its own XTM One and wants its triage agent to ask the **Velvet Harbor watcher** of Grand Meridian Hotels for the latest sightings. Hélène Marchetti opens :menu[Tools & Integrations > Integrations], **New Integration**, **OAuth A2A**:

- **Name**: `Example CERT triage partner`
- **Direction**: **Inbound: a partner tasks our agents**

She clicks **Create integration**, then **Generate credential**, and copies the pair with **Copy both**. She binds **Example CERT triage partner** to **Velvet Harbor watcher**, and sends Example CERT the client ID, the secret, the watcher's agent card URL and the token URL, `https://xtm-one.grand-meridian.example/mcp/oauth/token`.

A month later the secret leaks in a support ticket. Hélène clicks **Regenerate secret** and sends only the new secret: Example CERT keeps its client ID, and the leaked secret no longer gets a token.
:::

## Connect an API from its OpenAPI document

Any REST API that publishes a Swagger 2.0 or OpenAPI 3 document can become an integration, without code. Choose **OpenAPI / Swagger** in the catalog, paste the URL of the raw document or upload it, and click **Load spec**. Then check the base URL, choose the authentication among those the document declares, and select the operations to expose: each one becomes one tool, and **Read only** keeps only the operations that read. The integration's **API Spec** tab changes that selection later, and **Refresh from URL** reloads the document.

An OpenAPI integration is bound as that specific integration, never per user: for the members of a shared agent to use it, share it with their groups.

## Keep agents under the row level security of a Supabase project

A Supabase integration connected with a Management API personal access token reaches the project's data with roles that bypass row level security, and gives agents free SQL tools. To hold agents to what your row level security allows, give them a Postgres login role of their own. The **Database access** tab of the integration shows to the people who manage it.

:::steps
1. Open the Supabase integration, then its **Database access** tab.
2. Under **Database role**, choose the **Project** and the **Connection host**. The **Session pooler** works on every network; **Direct connection (IPv6)** works only over IPv6.
3. Choose the **Role**, type its **Password** and the **Database**, and click **Probe**. XTM One lists what it finds in the project, and the functions the role may execute. A role that bypasses row level security cannot be chosen.
4. Under **Functions agents may call**, allow the functions the agents need, then click **Save**.
:::

Agents then reach the data as that role, under its grants and row level security. The free SQL tools and the log query tool are withheld: agents list and call only the allowed functions. **Tables the role can still read directly** lists the tables the role can still select from: revoke that grant in Supabase so that the data goes through the functions only. To change the role, the host or the project, type the password again.

The database connection never goes through a relay: a Supabase integration routed through a relay refuses database calls. **Remove the role** brings back the Management API roles and the free SQL tools.

:::example[Let the triage agent read the SOC's Jira tickets]
The SOC of Grand Meridian Hotels tracks its incidents in Jira. Paul Garnier wants **Night shift triage** to find the ticket already open for an alert before it suggests a new one. Hélène Marchetti registered the organization's Atlassian app in the OAuth defaults, and Julien Bernard owns the SOC's service account in Jira.

Julien opens :menu[Tools & Integrations > Integrations], **New Integration**, **Atlassian Jira**:

- **Name**: `Jira SOC`
- **Description**: `Grand Meridian Jira site: the SOC incident project and its service desk queue.`
- **Authentication Mode**: **OAuth**, with **Use admin pre-configured credentials**

He clicks **Create Integration**, **Authorize** with the SOC service account, then **Test Connection**: **Connection successful**. He shares **Jira SOC** with the group **SOC**, and confirms that its members will act with the service account.

Sophie Laurent binds **Jira SOC** to **Night shift triage** in **Tooling**, keeps only its search and read tools in the tool scope, and selects the incident project under **Scope(s)**, as guidance. The agent now quotes the open ticket in its triage notes, and cannot create, comment on or close one.
:::

## What's next

- [Create an agent](/xtm-one/build/agents/)
- [Reach on-premise tools through a relay](/xtm-one/build/relays/), for a service inside your network.
- [Give agents recurring work](/xtm-one/use/work-items/), with assignments that react to an integration's events.
