---
title: What agents are made of
description: The building blocks of an XTM One AI agent, what each one adds, where you find it, and the order in which to put them together.
doc_type: explanation
products: [xtm-one]
personas: [ai-builder, admin]
owner: docs-xtm-one
last_reviewed: 2026-11-16
migrated_from:
  - xtm-one:/usage/resources/index/
---

An XTM One AI agent is a persona and a model, given what it needs to know and what it may act through. The agents themselves are listed in :menu[Orchestration > Agents], and you build both halves in two places of the left navigation: **Agent Studio** holds what agents are made of (knowledge, prompts, templates, skills), and **Tools & Integrations** holds what they act through (MCP servers, tools, integrations, and the **Connections** row, whose tabs are browser profiles, variables and relays). This page explains each block, how the blocks fit together, and where to start.

::diagram{id=xtm-one-build-overview-blocks}

::screenshot{id=xtm-one-build-overview-studio alt="The Agent Studio group of the left navigation, with Knowledge, Prompts, Templates and Skills."}

## The blocks

| Block | What it adds to an agent | Where |
| --- | --- | --- |
| [Agent](/xtm-one/build/agents/) | The persona (its system prompt), guardrails, the model, memory, and the blocks below bound to it. | **New Agent** in :menu[Orchestration > Agents], or the **Create an agent** card of :menu[Agent Studio] |
| [Knowledge base](/xtm-one/build/knowledge-bases/) | Your documents, searched while the agent answers, so it cites your policies instead of guessing. | :menu[Agent Studio > Knowledge] |
| [Prompt](/xtm-one/build/prompts/) | A wording written once, inserted in a chat or attached to an agent's system prompt. | :menu[Agent Studio > Prompts] |
| [Template](/xtm-one/build/templates/) | The brand of the decks, reports and dashboards agents produce. | :menu[Agent Studio > Templates] |
| [Skill](/xtm-one/build/skills/) | A procedure the agent loads only when a task calls for it, such as writing a Sigma rule. | :menu[Agent Studio > Skills] |
| [Integration](/xtm-one/build/integrations/) | A connection to a service with its credentials: the agent gets the service's tools. The OpenCTI, OpenAEV and OpenCRQ registered with XTM One are integrations created for you. | :menu[Tools & Integrations > Integrations] |
| [MCP server](/xtm-one/build/mcp-servers/) | A whole server of tools speaking the Model Context Protocol (MCP), such as your internal booking tools. | :menu[Tools & Integrations > MCP Servers] |
| [Tool](/xtm-one/build/tools/) | A built-in capability (web fetch, browser, code interpreter) or a function of your own run in a sandbox, written in XTM One or synced from a GitHub or GitLab repository. | :menu[Tools & Integrations > Tools], tab **Repositories** for the synced ones |
| [Browser profile](/xtm-one/build/browser-profiles/) | A saved sign-in the agent's browser uses, without the agent seeing the credentials. | :menu[Tools & Integrations > Connections], tab **Browser profiles** |
| [Variable](/xtm-one/build/variables/) | A named value, secret or not, resolved at runtime in prompts, personas, skills and tools, and, a secret one, in an integration's credential. | :menu[Tools & Integrations > Connections], tab **Variables** |
| [Relay](/xtm-one/build/relays/) | A path to the services inside your network, opened from inside, with no inbound flow. | :menu[Tools & Integrations > Connections], tab **Relays** |
| [Runner](/xtm-one/build/runners/) | A computer on which XTM One Desktop lets an agent run commands and open files. | The user menu, **Get XTM One Desktop** |

The other way round, XTM One also serves MCP: other clients reach OpenCTI, OpenAEV, OpenCRQ and XTM One itself through it. See [Use XTM One's own MCP servers from other clients](/xtm-one/build/platform-mcp-servers/).

## How the blocks fit together

- **The persona decides, the blocks enable.** A knowledge base the agent is not told to use is rarely searched; a tool its persona does not mention is called by chance. Say in the persona when to use each block.
- **What the agent reaches depends on the integration.** A registered OpenCTI is bound through a platform-managed integration: every call carries a token of the person the agent works for, so the agent reads only what that person may read in OpenCTI. A service connected with an API key acts as the key's owner for everyone who uses the agent.
- **Give the fewest tools that do the job.** Every tool is something the agent can act through. Narrow an integration to the tools the agent needs, or mark it read-only for that agent: [Create an agent](/xtm-one/build/agents/) shows how.
- **Sharing an agent lends what it carries.** People you share an agent with run it with the integrations, MCP servers and tools bound to it. XTM One asks before every change that lends something to new people: see [Share agents and resources](/xtm-one/build/sharing/).

## What needs the Enterprise Edition

Agents and the **Agent Studio** blocks are AI areas: they work when AI is switched on and the platform runs the Enterprise Edition, from an XTM One license or a registered OpenCTI or OpenAEV in Enterprise Edition. Otherwise they stay listed and readable, marked unavailable: on the **Agent Studio** page their cards carry an **AI off** or **Unlicensed** badge and no **Create**, and creating one from its list shows why. **Tools & Integrations** stays usable with AI off, so you can still connect services, answer a sign-in prompt or manage relays. See [What each edition of XTM One brings](/xtm-one/administer/editions/) and [Run XTM One with AI switched off](/xtm-one/administer/running-without-ai/).

## Grid and list views

The **Agents** list and the lists of knowledge bases, prompts, skills, MCP servers, tools and variables show cards by default. The **Grid view** and **List view** toggle, at the right end of the toolbar, switches to a sortable table, which compares a whole fleet at once: which model each agent runs, how many documents each knowledge base holds. Click a column header to sort, again to reverse; items without a value stay at the bottom. The cards sort too: pick a column in **Sort by**, and the arrow beside it switches between **Ascending** and **Descending**. Both views share one order, one search and one pager, so switching keeps your place. The table adds **Columns**, to hide or bring back a column. XTM One remembers the view, the page size and the shown columns per page. On **Integrations**, the grid view is the catalog of services to connect. On **Agents**, the star of a card pins it above the others, for you only.

## Where to start

1. Put the documents the agent needs in a [knowledge base](/xtm-one/build/knowledge-bases/).
2. Check that the services it reads are [integrations](/xtm-one/build/integrations/): the registered products already are.
3. [Create the agent](/xtm-one/build/agents/), test it in the chat, and [share it](/xtm-one/build/sharing/) with the team that uses it.
4. Give it recurring work with an [assignment](/xtm-one/build/agents/#give-it-recurring-work-with-an-assignment): a prompt it runs on a schedule, on a webhook call or on an event.
