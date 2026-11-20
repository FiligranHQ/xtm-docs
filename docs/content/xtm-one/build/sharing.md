---
title: Share agents and resources
description: Share an agent, a knowledge base or a connection with people and groups at the right level, know what sharing lends with it, and read who can use what.
doc_type: how-to
products: [xtm-one]
personas: [ai-builder, admin]
owner: docs-xtm-one
last_reviewed: 2026-11-08
migrated_from:
  - xtm-one:/usage/resources/sharing-and-access/
---

Everything you build in XTM One starts private: an agent, a knowledge base, a prompt, an integration. Sharing gives it to people and groups at a level you choose, and an administrator can open it to every user. Sharing also lends what the resource carries: the people who use a shared agent run it with its integrations, MCP servers and tools. This page shares a resource, explains what it lends, and shows where XTM One tells you who can use what.

::diagram{id=xtm-one-build-sharing-levels}

## The levels

| Level | Who uses it | Who changes it | Set by |
| --- | --- | --- | --- |
| Private | Its owner | Its owner | The default |
| Shared, **Read only** | The people and groups you name: they see it and use it | Its owner | Whoever can configure it |
| Shared, **Can configure** | The same | Also the people and groups at this level | Whoever can configure it |
| Shared, **Full access (incl. chats)** (agents only) | The same | The same, who also read the other users' chats with the agent and its traces | Whoever can configure it |
| Company-managed | Every user, or only the people and groups named in **Restrict access** | Administrators only | An administrator |

Agents, knowledge bases, prompts, skills, templates, MCP servers, integrations, custom tools and relay groups are shared this way. Variables have no sharing card: what a variable reaches is shown in its edit dialog. An administrator can also transfer the ownership of a resource to another user, from the same card. A company-managed agent accepts only company-managed integrations, knowledge bases, MCP servers and handover agents, so that what it does never depends on one person's private resources.

## Share a resource

:::steps
1. Open the resource. For an agent, open **Configuration** > **General**. For a knowledge base, a prompt, a skill, a template, an MCP server, an integration or a custom tool, open its **Configuration** and click **Edit**.
2. Turn on **Share with people and groups**.
3. Type at least two characters in **Search people or groups…**, and pick a group or a person with an account.
4. Choose the level next to each one.
5. For an agent, the change is saved at once. For the other resources, click **Save Changes**.
6. If **Before you save** opens, read it and confirm, or click **Keep things as they are**.
:::

::screenshot{id=xtm-one-build-sharing-agent-card alt="The sharing card of the Velvet Harbor watcher: Share with people and groups on, visible to the people and groups below, with the CTI team group listed."}

To stop sharing with someone, remove them from the list, or turn **Share with people and groups** off: the people concerned lose access at once. As an administrator, **Company-managed agent** (or the switch of the resource's kind) opens it to every user; turning on **Restrict access** then narrows it to the people and groups you list.

## What sharing lends

A resource that stores something others act with lends it to everyone it reaches:

| Resource | What its users act with |
| --- | --- |
| Integration | Its credential: a connected account, a token, an API key |
| MCP server | Its headers, its environment variables, or its own OAuth token |
| Custom tool | Its code, run on their behalf |
| Variable | Its value, secret or not, when something they use references it |
| Channel bot | Its bot token (bots are managed by administrators) |

It reaches them directly, when it is shared with them, or through something that carries it: an agent bound to it, an assignment, a knowledge base import source, a prompt or skill that references a variable. An agent's audience is also wider than its shares: the people who message it on a messaging channel, its agent-to-agent (A2A) partners, and anyone holding the address of one of its webhook assignments.

The integrations that platform-managed authentication backs, such as the registered OpenCTI, lend nothing: every call carries a token of the person the agent works for, so each person reads only what their own rights allow.

## A shared agent and its integrations

In a chat, a person who cannot see an integration bound to the agent, or whose owner did not share the connected account, does not use it, unless its owner uses it through the agent (below). Their own integration of the same service takes its place, or a company-wide one they may use. When there is none, a note under the answer says the integration was not used and why, and the agent is told too, so it can say what it cannot do. The agent's scheduled runs, its channels and its A2A partners keep acting with the bound integration.

When you share an agent with groups and it is bound to personal integrations those groups do not get otherwise, its sharing settings show **Personal integrations on a shared agent**. Choose what the members get when they chat with the agent:

- **Use my integrations through this agent**: the agent runs with your connected accounts in their chats too, under its persona, guardrails and tool scoping. The members never get these integrations in their own chats or agents, and never see the credential. Only the agent's owner can make this choice, for their own integrations. The warning then folds into one line, **Personal integrations used through this agent**, with who set it and when; **Review** reopens it, with **Stop using my integrations through this agent**.
- **Switch to per-user**: in a chat, each member uses their own connected integration of the same service, and gets none of its tools without one. Your integration stays for the agent's assignments and flows.

Binding another personal integration brings the full warning back until you choose again. Integrations connected from an OpenAPI document have no per-user form: use them through the agent, or the members get none of their tools in their chats. You can still share an integration with the same groups from its own page, so that the members use it everywhere.

## When XTM One asks

Before a change that lends something to new people, or takes it from people who use it, **Before you save** lists each resource concerned: whose it is, who newly gets it or loses it, through what, and how to avoid it (a company-managed resource instead of a personal one, per-user integrations). Nothing is refused: one click goes ahead. It opens only when someone other than you and the resource's owner is concerned, when you:

- share a resource, or something that carries one: an agent, a flow, a knowledge base, a transfer of ownership;
- bind a resource to something shared: an integration, an MCP server, a custom tool, a knowledge base, a prompt or a skill on a shared agent, a variable in its persona;
- open a public entry point: a webhook assignment, a channel bot that answers people without an account;
- change the members of a group, as an administrator: you are told what the group's shares lend them;
- unshare, unbind or delete something others use: the dialog says who loses what, with **Unshare** and **Keep sharing**, and replaces the usual confirmation.

## See who can use what

The people who can manage a resource see:

- a **Shared - N ways** badge on its card, with the breakdown in its tooltip;
- a **Who can use it** card on the overview of an integration: who holds its credential and who can use it, then its **Direct shares** (a group with its number of members, people, company-managed), the **Agents bound to it**, each with whom it reaches and what runs with it (chats, assignments, its public link, channels), and the paths **Through other entities** (an assignment, a flow, a knowledge base, a prompt, a skill);
- a **Sharing and access** card on the overview of an MCP server or a custom tool (in the edit dialog of a variable), listing the same paths.

Something you cannot open is named by its owner, without its name. Both follow every change as soon as it is saved. When an administrator deletes an account or a group, it leaves the sharing of everything it was given.

When the owner of an agent bound to an integration can no longer see that integration, the agent's row shows **Owner no longer has access**: its runs refuse the integration's tools until its owner gets access back or the agent is unbound. If you can configure that agent, **Unbind** removes the integration from it in one click.

::screenshot{id=xtm-one-build-integrations-who-can-use-it alt="The Who can use it card of the OpenCTI integration: the Agents bound to it section lists Crisis briefer, Night shift triage and Velvet Harbor watcher."}

:::example[One agent per team at Grand Meridian Hotels]
Sophie Laurent shares the agents she built:

| Agent | Shared with | Level | Why |
| --- | --- | --- | --- |
| **Velvet Harbor watcher** | **CTI team** | **Read only** | Camille Roux and the analysts use it; only Sophie changes it. Its OpenCTI integration is platform-managed, so each analyst reads only what their OpenCTI rights allow. |
| **Night shift triage** | **SOC** | **Read only** | Paul Garnier uses it at night. Julien Bernard, the SOC manager, gets **Can configure** as a person, to adjust its persona. |
| **Crisis briefer** | **Crisis cell** | **Read only** | Hélène Marchetti then makes it **Company-managed agent**, with **Restrict access** on the **Crisis cell** group. |

**Night shift triage** is bound to **Jira SOC**, the Jira integration Julien Bernard connected with his account and shares with the **SOC** group and with Sophie, so sharing the agent with **SOC** lends nothing new and no warning shows. Its booking tools come from the **Hotel booking platform** MCP server, whose secret headers the **Before you save** dialog names when Sophie binds it: she confirms, since the SOC is the audience that server was set up for.
:::

:::xtm-one[Share from the chat]
The General Assistant shares an agent for you: "share the Night shift triage agent with the SOC group, read-only". It names only people who already have an account, says who the agent is shared with afterwards, each group with its number of members, and takes access away at once when you ask.
:::

## What's next

- [Create an agent](/xtm-one/build/agents/).
- [Connect agents to your tools with integrations](/xtm-one/build/integrations/).
- [How access works in XTM One](/xtm-one/concepts/access/).
