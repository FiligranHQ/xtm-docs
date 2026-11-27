---
title: How XTM One keeps what agents know and do
description: The objects behind every XTM One screen - agents, assignments, runs, conversations, workspaces, work items and the resources agents use - and how they connect.
doc_type: explanation
products: [xtm-one]
personas: [ai-builder, admin]
owner: docs-xtm-one
last_reviewed: 2026-11-08
migrated_from:
  - xtm-one:/usage/foundations/
  - xtm-one:/usage/foundations/data-model/
---

XTM One keeps a small set of connected objects, and every screen is a view of them. An **agent** is the assistant that works. It is made of **resources** it links to, such as knowledge bases, prompts, skills, integrations and MCP servers, and it holds its own **assignments**, the rules that start its work. Each piece of work leaves a record: a **conversation** when someone chats with it, a **run** when an assignment or a delegation starts it. Knowing which object a screen shows tells you what a change there affects. This page describes each object, where it lives in the menu, and how they point to each other.

::diagram{id=xtm-one-data-model-objects}

## The agent and what it holds

An agent bundles everything it needs to do one job. Its page, under :menu[Orchestration > Agents], shows it in tabs: **Overview**, **Configuration**, **Value**, **Versioning**, **Memory**, **Observability**, **Feedback**, **Chats**, **Delegations** and **Activity**, each shown to the people allowed to see it.

::screenshot{id=xtm-one-data-model-agent-tabs alt="The agent Velvet Harbor watcher, owned by Sophie Laurent, open on its Configuration tab, between Overview and Value."}

| Part of the agent | What it is | Stored |
| --- | --- | --- |
| Instructions and persona | What the agent is for and how it answers | In the agent |
| Model | The AI provider and model it calls | In the agent; the providers are in :menu[Administration > Settings > AI and models > Models] |
| Linked resources | The knowledge bases, prompts, skills, integrations, MCP servers and tools it may use, chosen in **Configuration** | Each resource is its own object, linked |
| Assignments | The rules that start its work on its own: a trigger (**Scheduled**, **Webhook**, **Integration event**, **Manual** or **Work item**) and the task to do | In the agent: an assignment belongs to one agent |
| Memory | What it learned from past conversations and runs | In the agent, in the **Memory** tab |
| Budget | How much one run may spend | In the agent, in the **Value** tab, under what the agent's work returns ([how XTM One computes it](/xtm-one/concepts/value-methodology/)) |
| Versions | Every saved configuration, to compare or restore | In the agent, in the **Versioning** tab |

An assignment is not a second assistant: it is the agent's own automation, and it runs with the agent's instructions and tools. The **General Assistant**, which answers in :menu[Chat] when you pick no agent, is configured apart, in :menu[Administration > Settings > AI and models > General Assistant].

## Resources: what agents are made of

Resources live on their own and serve many agents. One prompt, skill, knowledge base or MCP server can be linked to ten agents; change it once and the ten change. The menu groups them by what they give an agent:

| Group | Resources | What they give an agent |
| --- | --- | --- |
| :menu[Agent Studio] | **Knowledge**, **Prompts**, **Templates**, **Skills** | What it knows and how it words and structures its work |
| :menu[Tools & Integrations] | **MCP Servers**, **Tools**, **Integrations**, and **Connections**, whose tabs hold the **Browser profiles**, **Variables** and **Relays** | What it can act on, with which credentials and by which network path |

How to build each one is in [What agents are made of](/xtm-one/concepts/agents/).

## Records: what work leaves behind

| Record | Created when | Where to find it |
| --- | --- | --- |
| Conversation | Someone chats with an agent, in XTM One, in a messaging app or from another module | :menu[Chat], and the agent's **Chats** tab |
| Workspace | Someone groups conversations on one topic, with the work items and shared state their agents build there | :menu[Work > Operations], tab **Workspaces** |
| Run | An assignment fires, a person starts one, or a run hands work to another agent | :menu[Work > Operations], tab **Executions**; each run has its own page |
| Background task | An agent delegates a task to another agent during a conversation or a run | :menu[Work > Operations], tab **Background Tasks** |
| Work item | An agent records a piece of work to track across runs, such as a ticket to follow up | :menu[Work > Operations], tab **Work Items** |
| Files and presentations | An agent generates or receives a file, or builds a deck | :menu[Work > Files] and :menu[Work > Presentations] |
| Trace | Any call to a model: prompts, tool calls, tokens, timing | :menu[Administration > Observability], tab **Traces**, for administrators |

A run is the answer to "did it work?": its page shows what triggered it, every tool call the agent made, the files it produced, and the question it asks when it waits for a person. Its statuses are explained in [What the dates and statuses of an object mean](/xtm-one/concepts/dates-and-status/).

## Linked or nested: what a change affects

Two kinds of relationships hold the model together, and they decide how far an edit reaches:

- **Linked**: the agent points to a resource that lives elsewhere. A picker or a card for a knowledge base, a prompt, a skill, an integration or an MCP server is a link. Changing the resource changes every agent that links it.
- **Nested**: the configuration is stored inside the object you edit. An assignment, an agent's instructions, its channel settings, your profile's preferences are nested. Changing them changes that object only.

Some values are inherited rather than set: an agent's temperature left empty uses the provider's default, an agent's limit of tool-calling rounds left empty uses the platform's. An empty field usually means "inherit", not "off". Who may see and change each object is the subject of [Who sees what: containers, scopes and nesting](/xtm-one/concepts/scopes/).

:::example[One agent, seen from four screens]
Sophie Laurent builds the agent `Velvet Harbor watcher` for the CTI team of Grand Meridian Hotels.

- In **Configuration**, she links the OpenCTI integration, the knowledge base `Grand Meridian security policies` and the prompt `Summarize a report for executives`. These three are resources: the agent `Crisis briefer` links the same knowledge base.
- In the agent's assignments, she adds `Weekly Velvet Harbor brief`, triggered **Scheduled** every Monday at 08:00, Europe/Paris. It is nested in the agent.
- On Monday 13 April 2026, the assignment fires. A run appears in :menu[Work > Operations], tab **Executions**, with the reports the agent read and the brief it wrote.
- Camille Roux asks the agent a follow-up question in :menu[Chat]. That conversation shows in her chat history and in the agent's **Chats** tab.

When Sophie rewords the prompt, both the weekly brief and every other agent that links the prompt change. When she changes the assignment's schedule, nothing else does.
:::

## What's next

- [Who sees what: containers, scopes and nesting](/xtm-one/concepts/scopes/)
- [How XTM One reuses what already exists](/xtm-one/concepts/deduplication/)
- [Create an agent](/xtm-one/build/agents/)
