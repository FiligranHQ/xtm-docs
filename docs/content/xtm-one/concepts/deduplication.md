---
title: How XTM One reuses what already exists
description: When XTM One keeps one shared object for many agents, what a duplicate copies and leaves out, and how it handles a webhook, a document or an event that arrives twice.
doc_type: explanation
products: [xtm-one]
personas: [ai-builder, admin]
owner: docs-xtm-one
last_reviewed: 2026-11-08
migrated_from:
  - xtm-one:/usage/foundations/deduplication/
---

XTM One is built to keep one shared object rather than many copies. A prompt, a skill, a knowledge base or an MCP server linked to ten agents exists once: improve it, and the ten agents improve. A copy is made only when you ask for one, with **Duplicate**, and it is then yours and independent. On the automatic side, XTM One handles once what arrives twice: a webhook delivery the sender repeats, a document synchronized again with the same content, an email the integration already read. This page explains both, so you know when to edit a shared object, when to duplicate it, and why a repeated event did not start a second run.

::diagram{id=xtm-one-deduplication-reuse}

## Reuse first

Before you create a resource, look for one that already exists: a company-managed knowledge base, a prompt your team shares, an integration already connected to the tool. Linking it to your agent creates no copy. The agent points to the resource, so the resource's owner keeps maintaining it, and everyone who links it benefits from each fix.

The choice comes down to who should get the change:

| You want | Do this |
| --- | --- |
| The whole team to get the improvement | Edit the shared object, or ask the people who can configure it |
| A different behavior for you only | **Duplicate** it, then change the copy |
| To try a change without risk | **Duplicate** it, test the copy, then report the change to the owner |
| To change an object you may only read | **Duplicate** it: the copy is yours |

A copy drifts from its source: a fix made later to the original never reaches it. Duplicate when the behavior must differ, not because the shared version is the first one you found.

## What a duplicate copies

**Duplicate**, on an agent's page, creates a new personal agent named "Copy of" and the original name, owned by you, whatever the scope of the original:

- It copies the instructions, the model, the linked resources, the budget and the assignments.
- Its assignments start disabled, with no run history: turn on the ones you need, so the copy does not run twice the work of the original.
- Its **Expose on Web** switch is off: a copy is never public until you choose so.
- A resource of the original that you may not use yourself, such as an integration shared with someone else, is left out of the copy.
- Its **Versioning** starts again at a first version: the history of the original stays with the original.

Templates and presentations offer **Duplicate** too, with the same result: a new object, yours, independent from its source.

## What XTM One handles once

| What arrives twice | What XTM One does |
| --- | --- |
| A webhook delivery that the sender repeats | The second delivery starts nothing and receives the first delivery's answer, marked as deduplicated. Deliveries are recognized by an idempotency header such as `Idempotency-Key`, `X-GitHub-Delivery`, `Webhook-Id` or `Svix-Id`, else by an identifier in the payload, else by the body itself, within `WEBHOOK_DEDUP_TTL_SECONDS` (300 seconds by default) |
| A document synchronized again into a knowledge base | It is matched to the existing document by its source address, or its file name and type. Same content: nothing is indexed again. Changed content: the document is updated and indexed again, never added twice |
| An email or a message an integration event already delivered | It is recognized by its identifier and starts no second run, for seven days |
| A fact an agent already remembers | Memory consolidation merges it: the agent's **Memory** tab shows "deduplication active", with how long facts are kept and how many |

The webhook window is set in the [XTM One configuration](/reference/xtm-one/configuration/#webhook-delivery). A sender that sends a new delivery with a new identifier, such as a redelivery from GitHub's interface, starts a new run on purpose.

:::example[A personal variation of the triage agent]
Paul Garnier, SOC analyst on the night shift at Grand Meridian Hotels, wants **Night shift triage** to answer in French. The agent is shared with the SOC group **Read only**.

- He opens the agent and clicks **Duplicate**. **Copy of Night shift triage** appears in :menu[Orchestration > Agents], personal to him.
- He changes the copy's instructions. The original agent, used by the rest of the SOC, is unchanged.
- The copy's assignments are disabled: Paul leaves them off, since the original already triages the overnight alerts.

The next week, Sophie Laurent improves the knowledge base **Grand Meridian security policies**. Both agents link it, so both get the change. A change Sophie makes to the original agent's own instructions does not reach Paul's copy.
:::

## What's next

- [Who sees what: containers, scopes and nesting](/xtm-one/concepts/scopes/)
- [Share agents and resources](/xtm-one/build/sharing/)
- [Start an agent from another tool with a webhook](/xtm-one/use/webhook-triggers/)
