---
title: How you work with agents in XTM One
description: The order of everyday work with XTM One AI agents, from the first question in the chat to the work they do on their own and the results you review, and the page that covers each step.
doc_type: explanation
products: [xtm-one]
personas: [cti-analyst, soc-analyst, ai-builder]
owner: docs-xtm-one
last_reviewed: 2026-11-08
---

XTM One AI agents work for you in two ways. You talk to them: ask a question in the chat, attach a document, mention a second agent, get a summary, a file or a deck back. And they work on their own: at a time you chose, on a schedule, when another tool calls them, handing parts of the work to each other. Everything they do is recorded, so you can read a result before acting on it, answer an agent that waits for your decision, and find what they produced. This page gives the order of that work and the page that covers each step. How an agent is built, with its instructions, knowledge and tools, is in [What agents are made of](/xtm-one/concepts/agents/).

## The steps of the work

::diagram{id=xtm-one-use-workflow}

| Step | What you do | Where | Read |
| --- | --- | --- | --- |
| 1. Start | Sign in, find your way and any object you can see with the top bar's search field, set your time zone, your default agent and your API keys | The sign-in page, :menu[Home], the top bar, **My Profile** | [Start with XTM One](/xtm-one/use/getting-started/), [Set up your profile and API keys](/xtm-one/use/profile/) |
| 2. Get your agents | Use the agents shared with you, or add a ready-made one a colleague published | **AI Catalog** in the top bar | [Add ready-made agents from the AI Catalog](/xtm-one/use/ai-catalog/) |
| 3. Ask | Ask an agent in plain language, with files, prompts and several agents in one message | :menu[Chat] | [Get answers from an agent in the chat](/xtm-one/use/chat/) |
| 4. Keep a topic together | File the conversations of one investigation in a workspace, give it instructions and sources, share it with your team | :menu[Chat], :menu[Work > Operations] | [Share a workspace with your team](/xtm-one/use/workspaces/) |
| 5. Get a deliverable | Have the agent build a deck or a one-page brief, run code on your data | :menu[Chat], :menu[Work > Presentations] | [Turn an agent's work into a presentation](/xtm-one/use/presentations/), [Let an agent run code in a sandbox](/xtm-one/use/sandbox-workspace/) |
| 6. Let agents work on their own | Have an agent act later, at a set time; let another tool start an agent's assignment; see how agents hand work to each other and what runs when | :menu[Chat], the agent's assignments, :menu[Orchestration > Agentic Flow] | [Let an agent act later, at a time you choose](/xtm-one/use/work-items/), [Start an agent from another tool with a webhook](/xtm-one/use/webhook-triggers/), [Follow how agents hand work to each other](/xtm-one/use/agentic-flow/) |
| 7. Keep it in bounds | Cap what one run of an agent may spend, beside what the agent returns | The agent's **Value** tab | [Set what one agent run may spend](/xtm-one/use/run-budgets/) |
| 8. Review | Read the runs, answer the ones that wait for you, rate the results, find the files | :menu[Work > Operations], :menu[Work > Files] | [Review what agents did](/xtm-one/use/work-history/), [Find the files agents created and received](/xtm-one/use/files/) |
| 9. Follow the big picture | Read the exposure posture of the suite and the agents' activity; see the hours the agents saved and what their work is worth; ask what AI costs and who uses it | :menu[Home], :menu[Work > Value], :menu[Chat] | [Read the home dashboard](/xtm-one/use/dashboard/), [Get answers about the platform's own activity](/xtm-one/use/platform-insights/) |

Steps 1 and 2 are done once and kept up to date. Steps 3 to 5 are the daily work of asking. Step 6 turns a request you make again and again into work agents do without you, and steps 7 to 9 keep that work visible and in bounds.

## Asking, and letting agents work alone

The same agent serves both ways, with the same instructions and tools. What changes is who is there:

- **In a conversation**, you are there. The agent answers as you, with your rights, and asks you before an action that needs approval. Each answer counts as one agentic execution of your quota.
- **On its own**, nobody is watching as it works. A one-off action you scheduled runs as you, at its time. An assignment runs on a schedule, on a webhook or on an event, as the identity its owner chose. Each run is recorded in :menu[Work > Operations], and a run that needs a person waits under **Awaiting Input**.

A request you make every week is a good candidate for an assignment: ask the agent's builder, or build it yourself, as [Create an agent](/xtm-one/build/agents/) describes.

## What needs the Enterprise Edition

The chat, the agents and everything they do need the Enterprise Edition and AI switched on. On the Community Edition, these areas stay in the navigation with a lock, and the dashboard still shows the exposure posture of the connected products; :menu[Work > Operations] and :menu[Work > Files] stay readable. :menu[Work > Value] needs the Enterprise Edition too, and stays open when AI is switched off. See [What each edition of XTM One brings](/xtm-one/administer/editions/).

XTM One agents also work inside the other products of the suite: see [XTM One in OpenCTI](/xtm-one/use/in-opencti/), [XTM One in OpenAEV](/xtm-one/use/in-openaev/) and [XTM One in OpenCRQ](/xtm-one/use/in-opencrq/).
