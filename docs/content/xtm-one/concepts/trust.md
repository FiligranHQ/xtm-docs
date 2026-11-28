---
title: How XTM One marks what to trust and what to review
description: The signals XTM One gives to judge an agent's work - who owns the agent, the run's status, the tool calls a person must approve, and the records that show where an answer came from.
doc_type: explanation
products: [xtm-one]
personas: [ai-builder, soc-manager, admin]
owner: docs-xtm-one
last_reviewed: 2026-11-08
migrated_from:
  - xtm-one:/usage/foundations/trust-and-confidence/
---

An agent's output is proposed work, written by a language model that can be wrong. XTM One gives no single confidence score. Instead it gives you four signals to decide whether to act on it: **who owns** the agent and its resources, the **status** of the run, the **approvals** a person gave before the agent changed anything, and the **records** that show what the agent read and did. This page explains each signal, where you see it, and the habits that keep an agent's mistakes from reaching your systems.

::diagram{id=xtm-one-trust-review}

## Who owns it

The scope of an agent and of the resources it links is the first signal:

| Scope | What it tells you |
| --- | --- |
| Company-managed | Maintained by administrators, the same for everyone: the approved standard |
| Shared | A team's agent or resource, which the people allowed to configure it may change |
| Personal | Someone's own agent, for testing or personal use: not a team standard |

An answer from a company-managed agent that links a company-managed knowledge base rests on approved instructions and approved content. The same question asked to a personal copy may not. The scopes are explained in [Who sees what: containers, scopes and nesting](/xtm-one/concepts/scopes/).

## The status of the run

A run tells you whether the work is final:

- **Running**: not finished, nothing to use yet.
- **Awaiting input**: the agent waits for a person, to answer its question or approve a tool call. The output is not final.
- **Completed**: finished normally. Read the result before you act on it outside XTM One. A run whose question someone dismissed is **Completed** too, with a result saying no action was taken.
- **Failed**: not finished; do not use a partial result. The cause is on the run. A run the platform itself ended, during a restart for example, reads **Interrupted** instead: the agent did not fail, but the work is not finished either.

Every status is described in [What the dates and statuses of an object mean](/xtm-one/concepts/dates-and-status/).

## Approvals before an agent acts

An agent can be set to ask a person before it runs a tool, such as creating a ticket, posting a message or writing to OpenCTI. In the agent's **Configuration**, the tool approval setting is **Ask**, **Don't ask** or **Follow the platform default**; an administrator sets the platform default and the per-tool rules in :menu[Administration > Settings > Access and security > Tool approvals].

When an agent asks, the run or the chat turn pauses with **Approval Needed**, shows the tool and its **Arguments**, and runs the tool only once someone approves it. The approval prompt can also reach Slack or Microsoft Teams: see [Approve agent actions from Slack and Teams](/xtm-one/administer/chat-approvals/). A tool that changed since a person approved it asks again. Every tool call is recorded with whether it ran without asking, and why.

In OpenCTI, an agent can also write to a draft instead of the live knowledge, so an analyst reviews the changes before they apply: see [XTM One in OpenCTI](/xtm-one/use/in-opencti/).

## Where an answer came from

To check an answer, open the run's page, from :menu[Work > Operations] or the agent's **Activity** tab:

| Section | Shows |
| --- | --- |
| **Actions Taken** | Each tool call, with its arguments and result: which OpenCTI entities it read, which knowledge base it searched, which ticket it created |
| **Agent Reasoning** | What the agent wrote between its tool calls |
| **Full Rendered Prompt** | The exact prompt sent to the model for this run |
| **Generated Files** | The files the run produced |
| **Result** | The final answer |

The run's trace holds every prompt, tool call, token count and timing: administrators open it in :menu[Administration > Observability], tab **Traces**, and the people with **Full access (incl. chats)** on the agent from its **Observability** tab. The agent's **Feedback** tab gathers the ratings people gave its work, and its **Versioning** tab shows which configuration produced a given answer.

## Good habits

- Treat an answer as a draft until you have checked the facts it rests on in its sources.
- Prefer company-managed agents and resources for work that leaves XTM One.
- Make agents that change your systems ask before they act, at least until their runs have proven them.
- Answer **Awaiting input** runs promptly: the work waits for you.
- Open **Actions Taken** when an answer surprises you, before you correct the agent.

:::example[A Jira ticket the SOC manager approves]
At Grand Meridian Hotels, **Night shift triage** reads OpenCTI and OpenAEV, and can create Jira tickets. Sophie Laurent sets its tool approval to **Ask**.

At 03:12 on 13 April 2026, the agent triages an alert about the PowerShell loader of Velvet Harbor and wants to create a ticket. The run shows **Awaiting input**, with **Approval Needed**: the Jira tool and its arguments, a ticket titled "Velvet Harbor loader on fd-paris-02" for the SOC queue. Paul Garnier reads **Actions Taken**: the agent found the loader's hash in an OpenCTI report on Velvet Harbor. He approves, the ticket is created, and the run ends **Completed**. If the agent had matched the wrong report, he would have denied the call and nothing would have reached Jira.
:::

## What's next

- [Review what agents did](/xtm-one/use/work-history/)
- [Govern how AI is used](/xtm-one/administer/ai-governance/)
- [Stop every agent at once](/xtm-one/administer/emergency-stop/): when an agent misbehaves.
