---
title: Protect your data and approve what agents do
description: What XTM One sends to an AI provider and keeps, how to scope what agents reach, and how to make agents ask a person before running a tool that changes something.
doc_type: how-to
products:
  - xtm-one
personas: [admin, ai-builder]
owner: docs-xtm-one
last_reviewed: 2026-11-08
edition: ee
migrated_from:
  - xtm-one:/administration/ai-governance/
---

An agent sends what it needs to answer to the AI provider you configured, and acts through the tools it is given. Governing AI in XTM One is deciding three things: which provider sees the prompts, what each agent may reach, and which of its actions wait for a person. This page explains what leaves the platform and what stays, then turns on tool approvals, the gate that stops an agent before a tool call until someone approves it.

::diagram{id=xtm-one-administer-ai-governance-approval}

## What an AI provider receives

A call to a model carries what the model needs to answer:

- the agent's persona, instructions and guardrails;
- the conversation or the assignment's prompt, with the recent messages;
- what the agent pulled in to answer: the passages of a knowledge base it retrieved, tool results, the payload of a trigger;
- the definitions of the tools it may call.

What never goes into a call: the stored credentials of integrations and channels, passwords and session tokens, and the full knowledge bases. Only the retrieved passages travel at question time. The whole text of a document goes once to the embedding model, when it is added to a knowledge base, and so do an agent's saved memories.

:::warning[Your provider sees your prompts]
XTM One does not anonymize or redact prompts before sending them, and stores them in your own deployment. Whether the provider keeps or trains on what it receives is decided by its terms, not by XTM One. XTM One itself never trains or fine-tunes a model on your data.
:::

Where prompts are processed is the provider's choice:

- **A hosted provider** (OpenAI, Anthropic, Gemini, Mistral...) processes them on its own infrastructure.
- **A local provider** (Ollama, or an OpenAI-compatible server on your network) keeps them inside your perimeter. The embedding model can be a local one even when chat runs on a hosted provider, so that documents never leave the network.

Choose the providers, and who may use each one, in :menu[Administration > Settings > AI and models > Models]: see [Set up the platform in Settings](/xtm-one/administer/settings/#ai-and-models).

## Scope what each agent reaches

An agent only sends what it can reach. Keep each one to what its task needs:

- the knowledge bases it reads, and the integrations, MCP servers and tools it calls;
- per assignment, a narrower set of tools and knowledge bases than the agent's;
- its **Guardrails**, the rules it always follows ("never send external email"), kept apart from its persona so that an edit to its tone cannot erase them.

XTM One adds its own protections to every run: it records attempts to make an agent ignore its rules (prompt injection, jailbreak) as security events without blocking them, wraps what agents fetch from outside (web pages, webhook payloads) so that it reads as data and not as instructions, adds fixed security instructions to every agent, and corrects the links to modules an answer gives. To block such attempts, connect Nova: see [Protect agents from prompt injection with Nova](/xtm-one/administer/prompt-safety/).

## Approve what agents do

On an agent that asks before running tools, every tool call stops until a person approves it, unless a rule lets that tool run without asking. The platform checks each call before it runs: the model cannot talk its way past it. Tool approvals are opt-in: until you turn them on, no agent asks.

### Before you start

- Your account is an administrator, for the platform settings. Whoever manages an agent can set the agent's own choice.
- The agents run with AI switched on.

::screenshot{id=xtm-one-administer-tool-approvals alt="The Tool approvals section of Settings > Access and security marked Not in use: no agent asks before running tools, and Every agent asks by default is off, with Tool rules below."}

### Turn approvals on

:::steps

1. Open :menu[Administration > Settings > Access and security > Tool approvals]. The top says whether approvals are **In use**, **Not in use** or **Suspended**, and lists the agents that ask.
2. To make every agent ask, turn on **Every agent asks by default** and confirm. Agents left on **Follow the platform default** follow it, the General Assistant included where chat is available.
3. To make one agent ask, open the agent, then **Configuration** > **Tooling**, and set **Ask before running tools** to **Ask**. **Don't ask** exempts one agent from the default. This works on the built-in agents too: it is the one setting an administrator may change on them, and it survives restarts.
4. Review **Tool rules** (below), then save.

:::

Every tool call is recorded with why it ran: allowed by a rule, covered by a saved approval, approved by a person, or run while approvals were off. Every change of these settings is recorded in :menu[Administration > Audit logs].

### Which tools ask first

On an agent that asks, a tool asks first unless a rule lets it run. XTM One ships rules that let the tools that only read (search, list, fetch) run without asking, including the read tools of your integrations, of the company-managed MCP servers and of the connected modules. A tool nobody considered yet, a new integration or a custom tool, asks.

**Tool rules** lists every tool and whether it asks first:

- **Let it run without asking** trusts a tool for everyone, even one that deletes: XTM One warns but does not refuse. **Add a tool by name** does it for a tool not connected yet.
- **Make it ask first** gates a read tool XTM One ships as trusted. The choice survives upgrades. **Back to the default** undoes it.
- **Make all its reads ask first** gates every read tool of a connected platform at once.

Some tools ask on purpose even though they only read: those that download a file into the agent's context (an attachment, a shared file), and the `manage_…` tools of the modules, which can list or delete depending on the call. `code_interpreter` runs without asking by default: make it ask first if a person must see generated code before it runs.

The tools of a Slack, Teams or Discord bot are never trusted automatically, since reading a channel reads people's conversations. A bot cannot show an approval prompt: a gated tool asked for there stops, and the bot says so. Each bot has its own **Ask before running tools** switch in :menu[Administration > Settings > Integrations > Channels & Bots] to let it run its tools. See [Collect tool approvals in your own chat client](/xtm-one/administer/chat-approvals/).

**Approval for connected platforms** decides, per connected OpenCTI, OpenAEV or OpenCRQ, whether its tools that change data ask while the platform cannot show an approval prompt (**Require approval**) or run.

### Answer an approval

The reviewer sees the tool, what it does and each argument with its description, then answers **Yes**, **No** (the agent is told and can propose something else) or **Yes, always**. They answer on the run itself, or in :menu[Work > Operations], where **Awaiting Input** filters the paused runs and several can be answered at once. **Reject** over several runs asks for one reason, applied to each. The notification bell says a run is waiting and links to it.

- **Yes, always** saves a preference for that person only, which also covers their scheduled runs. Another person is still asked. It is dropped when the tool's parameters change. It covers the tool of one MCP server or one custom tool: a tool of the same name on another server asks again. An approval of an MCP tool given before 1.261110.2 names no server, so the person is asked once more.
- **Allow every tool for me**, in each person's **Tool approvals** of their profile, stops asking them about anything, tools that delete included. It is recorded in **Audit logs**.
- A scheduled run that reaches a gated tool pauses and waits, without time limit, and its owner is notified. To let an agent's scheduled work run unattended, let its tools run without asking, or set the agent to **Don't ask**.

### Suspend all approvals

**Suspend all approvals**, at the bottom of **Tool approvals**, lets every tool call run without asking, for everyone, until **Resume approvals**. The rules stay saved. Runs already paused stay paused: answer or dismiss them in :menu[Work > Operations]. Use it for an emergency, such as an outage of the people who approve.

## How long XTM One keeps what agents did

Everything lives in your own deployment.

| Data | Kept | How to control it |
| --- | --- | --- |
| Conversations and messages | Until deleted | Delete the conversation, or its agent |
| Assignment runs and results | Until deleted | Delete the run, or the assignment |
| Language model traces | 90 days by default | **Trace retention (days)**, from 1 to 365, in :menu[Administration > Settings > Usage and quotas > Observability]. **Store full payloads** off keeps no prompt, answer or tool content; **Enable local trace storage** off keeps no trace |
| Daily usage aggregates | Without limit | Counts, tokens, cost and latency per day, agent, model and person, so that **Model usage**, in :menu[Administration > Observability], covers long periods. No prompt nor answer |
| Audit logs | 90 days by default | **Audit log retention (days)**, from 30 to 3650, in the same **Observability** section. Older entries are deleted every hour; each person's last sign-in is kept past the window. Read by administrators only |

Application logs and metrics are kept by your own log and monitoring systems. See [Monitor and troubleshoot XTM One](/deploy/operate/xtm-one/).

:::example[Make the SOC triage agent ask before it changes anything]
"Night shift triage" helps Paul Garnier triage the night's alerts. It reads OpenCTI and OpenAEV, and can create tickets in Jira and post in the Slack channel #soc-night. Hélène Marchetti wants a person to approve anything it changes, without slowing its reads.

1. Sophie Laurent opens "Night shift triage", **Configuration** > **Tooling**, and sets **Ask before running tools** to **Ask**.
2. In :menu[Administration > Settings > Access and security > Tool approvals], Hélène checks **Tool rules**: the OpenCTI and OpenAEV reads run without asking, the Jira ticket creation asks first. She clicks **Make it ask first** on `code_interpreter`.
3. **Every agent asks by default** stays off: "Crisis briefer" and "Velvet Harbor watcher" only draft text.

At 2:40, the agent proposes to create a Jira ticket for an alert on the PowerShell loader of Velvet Harbor. The run pauses; Paul reads the summary and priority it proposes in :menu[Work > Operations], **Awaiting Input**, and answers **Yes**. Its OpenCTI searches never stopped.
:::

## What's next

- [Protect agents from prompt injection with Nova](/xtm-one/administer/prompt-safety/)
- [Collect tool approvals in your own chat client](/xtm-one/administer/chat-approvals/)
- [Stop every agent at once](/xtm-one/administer/emergency-stop/)
- [Review what agents did](/xtm-one/use/work-history/)
