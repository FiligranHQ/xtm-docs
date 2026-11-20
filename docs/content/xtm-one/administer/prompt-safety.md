---
title: Protect agents from prompt injection with Nova
description: Connect the Nova prompt guard, choose which detection rules log and which block, extend it to tool results and model answers, and review what it matched.
doc_type: how-to
products:
  - xtm-one
personas: [admin]
owner: docs-xtm-one
last_reviewed: 2026-11-08
edition: ee
no_screenshot: XTM One screens are not captured yet; the screenshots of this page are proposed in capture/proposals/xone-administer.yml.
migrated_from:
  - xtm-one:/administration/prompt-safety/
---

The **Nova Prompt Safety** integration screens what reaches your agents against detection rules, and logs or blocks what they match: jailbreaks, prompt injections, attempts to extract secrets and other attacks on a language model. Once an administrator connects it, it applies to every agent of the platform. This page connects it, sets the rules, extends it to more surfaces, and reviews its matches.

::diagram{id=xtm-one-administer-prompt-safety-guard}

Nova is built on two open-source projects by Thomas Roccia, under the MIT license: [NOVA](https://github.com/Nova-Hunting/nova-framework), the prompt pattern matching framework whose `.nov` rule format you write, and [NOVA rules](https://github.com/Nova-Hunting/nova-rules), the community rules XTM One syncs by default. A rule that would help others belongs upstream.

## Before you start

- Your account is an administrator: the rules and the guard are platform-wide.
- The agents run with AI switched on: Nova screens what is sent to them.

## Connect Nova

:::steps

1. Open :menu[Tools & Integrations > Integrations] and click **New Integration**.
2. Pick **Nova Prompt Safety**. It needs no credentials.
3. Make it **Company-managed**, then save. A personal connection syncs nothing and screens nothing.
4. Click **Test Connection**. It says how many rules are active, when the rule source last synced, and which tiers can fire. It reports a problem when no rule is active (the guard then lets everything through), when the last sync failed, or when syncing is off.

:::

Connecting adds the community rules as the rule source and syncs them once a day. Disconnecting stops the guard at once and removes the synced rules, including those you edited; the rules you wrote are kept.

<!-- shot: xtm-one-administer-nova-rules -->

## Choose what the rules do

The **Rules** tab of the integration, which only administrators see, holds:

- **Rule source**: the repository of the community rules, when it last synced and how many rules it brought. Sync **Manual**, **Every 6h**, **Every 12h**, **Daily** or **Weekly**, or click **Sync now**.
- **Models**: the language model the `llm` tier asks and the embedding model the `semantics` tier scores with, the platform defaults unless you pick others.
- The rules, with their tiers and their action. **New rule** writes one in **Rule (.nov)**; open a rule to edit, disable or delete it.

A rule matches a message in three ways: `keywords` (words and regular expressions), `semantics` (similarity of meaning, with the embedding model) and `llm` (a yes or no question to a language model). The `semantics` and `llm` tiers need the full NOVA engine installed with the platform; without it, the tab says **semantics/llm tiers need the optional engine** and only keywords match.

Each rule has an **Action on match**:

- **Log**: records each match and lets the message through.
- **Block**: refuses the message. The agent never sees it, and the person gets a short refusal instead of an answer.

Synced community rules arrive as **Log**. Watch what they match before switching one to **Block**: your choice of action, and whether a rule is enabled, is kept across syncs. A rule whose `semantics` are scored by a model XTM One has no calibration for shows **Uncalibrated semantics**: keep it on **Log**.

## What the guard screens

While Nova is connected, the guard evaluates:

- **Every message sent to an agent**, before the model is called. A **Block** rule refuses it.
- **Messages to the General Assistant and to a public chat widget**, and the text of the documents a widget visitor uploads (not images). By default a match is only recorded.
- **What an agent saves to its memory**, after it is saved. A match is recorded; the memory is kept.

If the guard fails or does not answer in time, the content goes through, and the failure is written to the server log.

## Extend the guard

In :menu[Administration > Settings > Access and security > Security options], each option also has its environment variable, which wins:

| Option | Variable | Default | Effect |
| --- | --- | --- | --- |
| **Prompt guards block General Assistant messages** | `PROMPT_GUARD_BLOCK_GENERAL_ASSISTANT` | Off | A General Assistant message a **Block** rule matches is refused. A false positive refuses a colleague's question. |
| **Prompt guards block widget messages** | `PROMPT_GUARD_BLOCK_PUBLIC_WIDGET` | Off | A public widget message a **Block** rule matches is refused. |
| **Tools screened by the prompt guard** | `PROMPT_GUARD_SCANNED_TOOLS` | None | The tools whose results are evaluated, with the text of their files. Pick tools that return outside content: web fetch, email, tickets. |
| **Withhold tool results a guard rule blocks** | `PROMPT_GUARD_BLOCK_TOOL_RESULTS` | Off | A screened result a **Block** rule matches is withheld: the agent reads a short refusal instead. The agent then waits for each screened result's verdict. |
| **Prompt guards on model answers** | `PROMPT_GUARD_MODEL_OUTPUT` | `off` | `log` or `block` evaluates every model answer. `block` withholds one a **Block** rule matches; an answer already shown as it was written, in the web chat, is only recorded. |

`NOVA_LLM_MAX_CONDITIONS` caps how many `llm` conditions are asked per message, to bound the cost of the `llm` tier. See [Nova prompt-safety engine](/reference/xtm-one/configuration/#nova-prompt-safety-engine).

## Review the matches

Every match is in :menu[Administration > Audit logs], with the action `security_alert`: the rule, its action, whether the content was refused, and where it came from (a chat message, a tool result, a memory, a widget upload, a model answer). The **Security audit**, in :menu[Administration > Settings > Access and security], reports which surfaces the guard covers and the rules scored by an uncalibrated model.

:::example[Guard the agents that read outside content]
"Velvet Harbor watcher" reads OpenCTI reports, some of which quote the fake summit booking request that Velvet Harbor sent to front-desk staff on 9 January 2026: a text written to manipulate its reader. Hélène Marchetti wants the agents protected from instructions hidden in such content.

1. She connects **Nova Prompt Safety**, company-managed. **Test Connection** reports the community rules active and the three tiers available.
2. On the **Rules** tab, she sets **Rule source** to **Daily**. After a week on **Log**, she switches to **Block** the community rules whose matches were all real attacks.
3. In **Security options**, **Tools screened by the prompt guard** gets the web fetch tool, and **Withhold tool results a guard rule blocks** is turned on.

Marc Dupuis could set the same choices in the deployment instead, where they win over the screen:

```bash
PROMPT_GUARD_SCANNED_TOOLS=web_fetch
PROMPT_GUARD_BLOCK_TOOL_RESULTS=true
PROMPT_GUARD_BLOCK_GENERAL_ASSISTANT=false
PROMPT_GUARD_MODEL_OUTPUT=log
```

When the agent fetches a web page that tells it to ignore its instructions, it reads a refusal naming the tool instead, and a `security_alert` entry appears in **Audit logs**.
:::

## What's next

- [Protect your data and approve what agents do](/xtm-one/administer/ai-governance/)
- [Set up the platform in Settings](/xtm-one/administer/settings/#access-and-security)
- [Security switches](/reference/xtm-one/configuration/#security-switches)
