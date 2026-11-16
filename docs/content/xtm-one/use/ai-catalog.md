---
title: Add ready-made agents from the AI Catalog
description: Browse the agents, skills, prompts, MCP servers and custom tools your colleagues published to the XTM One AI Catalog, review one, add it to your account and finish its setup.
doc_type: how-to
products: [xtm-one]
personas: [ai-builder, cti-analyst]
owner: docs-xtm-one
last_reviewed: 2026-11-10
migrated_from:
  - xtm-one:/usage/ai-catalog/
---

The **AI Catalog** is the library of AI resources published on your XTM One platform: agents, skills, prompts, MCP servers and custom tools that a colleague built and chose to offer. Instead of building an agent from scratch, you review one, add it to your account as your own copy, and finish what only you can set up, such as your integrations and secrets. This page browses the catalog, adds a resource, and lists what the copy carries and what it does not.

::diagram{id=xtm-one-ai-catalog-copy}

## Before you start

- An administrator has turned the catalog on with `AI_CATALOG_ENABLED=true`: see the **Advanced tuning** section of the [configuration reference](/reference/xtm-one/configuration/). Until then, the catalog is not offered.
- Adding a resource to your account needs AI switched on.

## Browse and review

:::steps
1. Click **AI Catalog** in the top bar. The catalog opens in a new tab.
2. Choose **All**, or one category: **Agents**, **Skills**, **Prompts**, **MCP Servers** or **Tools**.
3. Search by name or description. Each card shows the resource's name, a short description, its tags and who shared it.
4. Open a resource to review it before you add it:

   | Resource | What its page shows |
   | --- | --- |
   | Agent | Its persona, integrations, knowledge bases, MCP servers, custom tools, skills, prompts, built-in tools, variables, and its assignments |
   | Skill | Its content |
   | Prompt | Its full text |
   | MCP server | Its transport, its command or URL with its arguments, and the tools it exposes |
   | Custom tool | Its input parameters, its JavaScript code, its size and its timeout |

:::

::screenshot{id=xtm-one-ai-catalog-agent alt="The AI Catalog page of Velvet Harbor watcher, shared by Sophie Laurent: its persona, the OpenCTI integration, knowledge base, prompt and built-in tool a copy brings, its two assignments, and the Download JSON and Add to my account buttons."}

**Download JSON** downloads the resource's definition, to keep or import elsewhere.

## Add a resource to your account

:::steps
1. On the resource's page, click **Add to my account**. For an agent, the button opens **Install Agent**.
2. For an agent, give your copy a **Name**, then pick, for each integration it uses, one of your connected integrations. Company-managed integrations are connected for you. You can skip and connect one later.
3. Click **Install**. **Installed successfully** confirms it, and lists what your copy does not carry or carries only in part, with the reason.
4. Click **Open agent** to finish its setup, or **Set values in Variables** when it names variables to fill.
:::

What is copied depends on the kind of resource:

| Resource | What your account gets |
| --- | --- |
| Agent | A usable agent. Its assignments are installed disabled: turn on the ones you want |
| Skill, prompt | A copy of your own |
| MCP server | A copy without its secrets: configure its environment values and credentials, which the catalog shows masked |
| Custom tool | A copy, which may need setup |

Company-managed resources you can use are bound as they are, not copied. A resource restricted to a group you are not in is left out (an MCP server, a custom tool, a skill, a prompt), recreated empty (a knowledge base), or replaced by your own (an integration).

## What an agent's copy never carries

Nothing is left out silently: the end of the installation names each part, and the agent's page shows a notice until its variables have values.

- The publisher's own integrations, channel bots, sharing, run-as identity and handovers. Connect your own integrations; set handovers on your copy.
- Knowledge bases not shared with you, recreated empty: add their documents.
- The values of secret variables: the variables are created in your account, and you fill them.
- A run budget above what applies to you: it is lowered to your limits. See [Set what one agent run may spend](/xtm-one/use/run-budgets/).
- The code of the publisher's personal custom tools, unless they chose to include it.

To publish your own agent, switch on **Include in AI Catalog** on its configuration: see [Share agents and resources](/xtm-one/build/sharing/). Only an administrator publishes a company-managed agent.

:::example[Start the SOC's triage agent from the CTI team's]
On 12 April 2026, Julien Bernard, SOC manager at Grand Meridian Hotels, wants an agent like **Velvet Harbor watcher** for the Spanish hotels' SOC. Sophie Laurent has published it. Julien opens **AI Catalog** > **Agents**, opens **Velvet Harbor watcher**, reviews its persona and its assignment **Weekly Velvet Harbor brief**, and clicks **Add to my account**:

| Field | Value |
| --- | --- |
| **Name** | `Velvet Harbor watcher (Spain)` |
| Slack integration | His own Slack integration, connected to the Grand Meridian workspace |
| Microsoft Teams integration | Skipped, to connect later |

**Installed successfully** lists the handovers his copy does not carry, and the variable `SOC_ESCALATION_EMAIL`, which needs a value. He clicks **Set values in Variables**, fills it, opens the agent and turns on its assignment, which was installed disabled.
:::

## What's next

- [Create an agent](/xtm-one/build/agents/), to adapt the copy: its persona, tools and assignments.
- [Store values agents reuse in variables](/xtm-one/build/variables/), for the variables it needs.
- [Connect agents to your tools with integrations](/xtm-one/build/integrations/), for the integrations to connect.
