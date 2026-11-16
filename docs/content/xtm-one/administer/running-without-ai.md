---
title: Run XTM One with AI switched off
description: Switch AI off on an XTM One platform so that no model is contacted and no AI work starts, know what keeps working and what waits, and switch it back on.
doc_type: how-to
products: [xtm-one]
personas: [admin, platform-engineer]
owner: docs-xtm-one
last_reviewed: 2026-11-08
migrated_from:
  - xtm-one:/administration/running-without-ai/
---

You can run XTM One with AI switched off. No language or embedding model is contacted, no AI work starts, and nothing leaves the deployment for an AI provider, while everything that needs no model keeps working: sign-in, users, settings, the connections to the suite, and every agent, run and trace already there, readable. Use it for a platform installed before AI is approved, for a deployment whose policy forbids sending data to a model, or for an air-gapped install with no provider to reach. This page switches AI off and on from the interface, sets the default at deployment time, and explains what happens to the work in progress. AI off needs no license; the editions are explained in [What each edition of XTM One brings](/xtm-one/administer/editions/).

::diagram{id=xtm-one-running-without-ai-switch}

## Before you start

- You are an XTM One administrator: :menu[Administration > Settings] is reserved to administrators.
- To stop the runs already in progress as well, use the emergency stop instead: switching AI off stops new work only. See [Stop every agent at once](/xtm-one/administer/emergency-stop/).

## Switch AI off or on

:::steps
1. Open :menu[Administration > Settings > Platform]. The card **Artificial Intelligence** shows the current choice in **AI on this platform**.

   ::screenshot{id=xtm-one-running-without-ai-parameters alt="The Artificial Intelligence card of Settings > Platform, its AI on this platform list open on Enabled, Disabled and Deployment default (enabled), the value this platform follows until someone makes an explicit choice."}
2. Choose a value in **AI on this platform**:

   | Value | Effect |
   | --- | --- |
   | **Enabled** | AI runs, whatever the deployment default |
   | **Disabled** | AI does not run, whatever the deployment default |
   | **Deployment default (enabled)** or **Deployment default (disabled)** | Follows `PLATFORM_MODE`, the value the platform was started with, shown in brackets |

   An explicit choice shows "This is an explicit choice made here, overriding the deployment configuration."; the deployment default shows "Follows the deployment configuration until a choice is made here." **Deployment default** is not the same as **Disabled**: it hands the decision back to the deployment, so a later change of `PLATFORM_MODE` takes effect.
3. When the choice switches AI off, the dialog **Turn off AI on this platform?** explains what follows. Click **Turn off AI**.
4. When the choice switches AI on, wait for the confirmation: the first time, saving creates the built-in agents, skills and templates a normal start would have created, which takes a moment. If the page is reloaded before it ends, choose the value again to complete it.
:::

A notification reads **AI disabled** or **AI enabled**. No restart is needed: every server of the deployment follows within a few seconds.

After switching AI on for the first time, add a model provider in :menu[Administration > Settings > AI and models > Models] if none exists: agents need one to answer.

## What keeps working

- Sign-in and single sign-on, users, groups, invitations and API keys.
- :menu[Administration > Settings], branding, :menu[Administration > Audit logs], the health and metrics endpoints.
- The connections to OpenCTI, OpenAEV and OpenCRQ, their exposure figures, and their MCP servers for AI clients, which call the products without a model.
- Integrations, custom tools, variables, relays and browser profiles: you can still configure them, within what your edition allows.
- Every agent, knowledge base, conversation, run and trace that exists, readable as before, including :menu[Administration > Observability], which carries no lock.

## What waits for AI

- Chat, in XTM One, in messaging apps and in the assistants of OpenCTI, OpenAEV and OpenCRQ.
- Creating agents, knowledge bases, prompts, skills, templates and MCP servers. :menu[Administration > Settings > AI and models > Models] and **Media** are read only.
- Indexing documents in knowledge bases: they stay pending, and nothing is sent to an embedding provider.
- The MCP endpoints that expose agents, and Agent-to-Agent interoperability.

These areas stay in the menu with a lock, so you still see what is configured. Their actions are disabled and say "AI is disabled on this platform. An administrator can enable it in Settings → Platform." instead of failing, and an **AI off** badge tells every user why. On the exposure briefing and the CTEM stage pages, the cards that need AI also give an administrator **Turn AI on**, which opens the **Artificial Intelligence** card.

To switch AI off for some people only, switch it off for their group instead: see [Switch AI off for a group](/xtm-one/concepts/access/#switch-ai-off-for-a-group). The platform switch wins: while it is off, AI is off for everyone.

## What happens to work when AI goes off

- **New AI work** is refused at once: a chat message, a creation, an assignment triggered by a person.
- **Runs already in progress** finish under their own time limit, so you may see AI activity for a few more minutes. They are not cancelled, which would leave half-written runs and traces.
- **Work that was due but had not started** is **Held**: a run waiting for its delay, a run resumed after a person answered it (the answer is kept), a background task not yet picked up. Held work is never charged and starts by itself within about five minutes of AI being on again. Work held longer than `AI_UNAVAILABLE_HOLD_MAX_SECONDS` (one week by default) fails with the cause **AI disabled on the platform**; set it to `0` to fail such work at once instead of holding it.
- **Schedules, integration events and webhooks** start no AI work, and resume on their own. A schedule that missed several occurrences runs once, not once per occurrence.

Nothing is deleted, disabled or paused: switching AI back on needs no repair.

## Set the deployment default

To start a platform with AI off, set `PLATFORM_MODE=ai_disabled` on the platform and the worker. It suits an install that must never contact a provider unless an administrator deliberately decides so. A platform started this way creates no built-in agents, assignments, templates, skills or knowledge bases, and its startup log says what was skipped: they are created the first time an administrator switches AI on. See the [XTM One configuration](/reference/xtm-one/configuration/#platform-mode).

:::example[An air-gapped install until the AI policy is signed]
Grand Meridian Hotels installs XTM One in its Paris datacenter before the AI usage policy is approved. Marc Dupuis, IT operations, starts it with AI off, and keeps the default hold of one week for work that comes due:

```bash
# .env of the XTM One deployment, read by the platform and the worker
PLATFORM_MODE=ai_disabled
AI_UNAVAILABLE_HOLD_MAX_SECONDS=604800
```

Hélène Marchetti, the CISO, configures single sign-on, the groups CTI team, SOC and Crisis cell, and registers OpenCTI. :menu[Administration > Settings > Platform] shows **Deployment default (disabled)**. When the policy is signed, she chooses **Enabled**, waits for **AI enabled**, adds the company's model provider in :menu[Administration > Settings > AI and models > Models], and Sophie Laurent starts building **Velvet Harbor watcher**. `PLATFORM_MODE` stays as it is: the explicit choice in the interface wins.
:::

## Check that it works

With AI off, the home page reads **AI is switched off on this platform**, and :menu[Chat] carries a lock in the menu. :menu[Administration > Settings > Platform] shows the value you chose. In :menu[Work > Operations], a run that came due meanwhile shows **Held**.

## What's next

- [Stop every agent at once](/xtm-one/administer/emergency-stop/)
- [Govern how AI is used](/xtm-one/administer/ai-governance/)
- [What each edition of XTM One brings](/xtm-one/administer/editions/)
