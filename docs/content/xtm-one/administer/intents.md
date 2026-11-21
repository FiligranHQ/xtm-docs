---
title: Choose which agent answers each AI feature
description: Bind your agents to the intents OpenCTI, OpenAEV and OpenCRQ declare, per vertical and by priority, so that each AI feature of the XTM platform reaches the agent you chose.
doc_type: how-to
products:
  - xtm-one
  - opencti
  - openaev
personas: [admin, ai-builder]
owner: docs-xtm-one
last_reviewed: 2026-11-08
edition: ee
migrated_from:
  - xtm-one:/administration/intents/
---

Every AI feature of OpenCTI, OpenAEV and OpenCRQ (summarize a report, turn a question into search filters, extract knowledge from a document in a playbook) asks XTM One for an intent, a named capability such as `cti.container_summary`. An intent says what has to be done, never who does it: a binding decides which agent answers. This page binds your own agents to intents, orders them when several answer the same one, and finds why a feature does not appear in a connected product.

::diagram{id=xtm-one-administer-intents-routing}

## How a request reaches an agent

1. OpenCTI, OpenAEV or OpenCRQ registers with XTM One and declares the intents it uses, with its vertical: `cti`, `aev` or `crq`.
2. XTM One adds the intents it does not know to the list, tagged with the platform that declared them.
3. You bind one or more agents to each intent, for one vertical or for **All verticals**, in an order of priority.
4. When a person uses the feature, the product asks which agents are bound to the intent for its vertical, and uses the one of highest priority.
5. That agent runs in XTM One, with the context the product sent, and the answer goes back into the product.

An intent a platform declared and no agent is bound to shows **Unbound** in the **Usage** column: the product treats the feature as unavailable, and usually hides it. Binding an agent is what turns the feature on.

## Before you start

- XTM One runs the Enterprise Edition with AI switched on: **Intents** is an AI area.
- The platform runs in its XTM One mode, `PLATFORM_MODE=xtm_one`, the default. Started in another mode, such as `copilot`, it routes no request through an intent: **Intents** is neither in the menu nor in the command palette (**Browse intents**), and `/intents` opens the home page.
- Your account is an administrator, to set the bindings everyone follows.
- The agent to bind is enabled, and company-managed or shared with at least one group. XTM One refuses a binding for everyone to a private agent, since it would reach the people of the connected product. See [Share agents and resources](/xtm-one/build/sharing/).
- The connected product is registered: see [Configure XTM One](/deploy/configure/xtm-one/).

::screenshot{id=xtm-one-administer-intents alt="The Intents page: the Verticals card, then the table of intents with the Intent, Platform, Usage and Bindings columns, the search, the Platform and Usage filters and Export CSV; Create Intent at the top right."}

## Bind an agent to an intent

:::steps

1. Open :menu[Orchestration > Intents]. The table lists each intent with the platforms that declared it, its **Usage** and its number of **Bindings**; hover an intent's name to read its description. **Search intents…** finds one by name, description or platform, and the **Platform** and **Usage** filters narrow the list.
2. Click the intent's row: a dialog shows its description and its bindings, grouped by vertical. Click **Add Binding**: the **Add Agent Binding** dialog opens. The **+** at the end of the row opens it too, without the intent's dialog.
3. Search for the agent, then select one or more verticals, or **All verticals**. One binding is created per vertical.
4. Click **Add 1 Binding**, or **Add 2 Bindings** for two verticals.
5. When several agents answer the intent, open its row again and use **Increase priority** and **Decrease priority** to put yours first in its vertical.

:::

::screenshot{id=xtm-one-administer-intents-bindings alt="The dialog of the intent cti.container_summary: its description, then the cti group where Velvet Harbor watcher, priority 1, is above the built-in OpenCTI Entity Insights, with the priority arrows; Edit Intent and Add Binding at the bottom."}

The agent now answers the feature in every product of that vertical. The same bindings are on the agent's side: open the agent, then **Configuration**, and the **Intent Bindings** card lists the intents it answers, to add or remove one without leaving the agent.

Some lists of the connected products, which let the person choose an agent, only show the agents tagged with the product's name, `opencti` or `openaev`. Tag your agent to appear there.

:::xtm-one[Bind an agent from the chat]
The General Assistant, and any agent with the agent builder built-in tool, manages bindings too. Ask it "which intents is Velvet Harbor watcher bound to?" or "bind Velvet Harbor watcher to `cti.container_summary` for `cti`": it lists the agent's bindings with their vertical and priority, binds it, or removes a binding. It also says when no connected product can be offered the agent, because the agent is disabled or private. It acts as you, with the checks of this page: an administrator's binding applies to everyone, anyone else's is their own personal binding; a private agent is never bound for everyone, a locked binding is not overridden, and a **builtin** binding is neither changed nor removed. Each change is recorded in the audit logs.
:::

## Verticals and priorities

A vertical is the business context of a connected platform: `cti` for OpenCTI, `aev` for OpenAEV, `crq` for OpenCRQ, and others a platform may declare, such as `fimi`. It lets the same intent reach a different agent depending on where the request comes from.

- The **Verticals** panel, at the top of the page, unfolds into the table of verticals, with how many bindings use each. A vertical a platform declares is created by itself; **Add Vertical** adds your own: type its **Vertical name**, a lowercase id such as `fimi`, and click **Add**.
- The pencil of a vertical's row renames it, and its bindings follow. A vertical still used by a binding cannot be deleted, and a built-in vertical can be neither renamed nor deleted.
- A binding for **All verticals** applies to every platform, and is listed in each vertical group too, since it competes with the bindings of that vertical.
- Priority is decided per vertical: the highest wins.

## The built-in bindings

XTM One binds its built-in agents to the intents the modules declare. These bindings are marked **builtin**, cannot be changed or deleted, and are restored at every start. To take over a feature, bind your own agent to the same intent and vertical, with a higher priority. The main ones:

| Intent | What the product asks | Built-in agent | Vertical |
| --- | --- | --- | --- |
| `global.assistant` | The assistant behind the in-product chat | **CTEM Assistant** and **XTM-One Assistant**, then the assistant of each product | all |
| `global.summarize`, `global.explain`, `global.fix_spelling`, `global.change_tone`... | Rewrite a text | **OpenCTI Assistant**, **OpenAEV Assistant**, **OpenCRQ Assistant** | `cti`, `aev`, `crq` |
| `cti.nlq_search` | Turn a question into OpenCTI search filters | **OpenCTI NLQ** | `cti` |
| `cti.container_summary`, `cti.container_report`, `cti.entity_activity`... | Summarize a report, a grouping or a case, or an entity's activity | **OpenCTI Entity Insights** | `cti` |
| `cti.stix_harvester` | Extract a STIX 2.1 bundle from a document | **CTI STIX Harvester (Filigran AI)**, then **CTI STIX Harvester** | `cti` |
| `cti.stix_transformer`, `cti.stix_consumer` | Change a bundle, or act on it at the end of a playbook | **CTI STIX Transformer**, **CTI STIX Consumer** | `cti` |
| `cti.ttp_harvester` | Extract techniques from documents | **CTI TTP Harvester (Filigran AI)** | `aev` |
| `aev.attack_path_orchestrator` | Drive an autonomous attack path in OpenAEV | **OpenAEV Attack Path Orchestrator** | `aev` |
| `aev.phishing_email_html_generator`, `aev.detection_rules_generator`... | Write a phishing lure or detection rules for an exercise | The AEV generators | `aev` |
| `crq.risk_scenario_generator`, `crq.control_gap_analysis`, `crq.assessment_summary` | Draft risk scenarios, analyze control gaps, summarize an assessment | **OpenCRQ Assistant** | `crq` |

The **Intents** page of your platform is the reference: it shows what the connected products declared and what is bound today, and **Export CSV** downloads it as filtered and sorted, with the description of each intent. The **CTEM Assistant** has the highest priority on `global.assistant`, and hands the conversation to the assistant of the product.

## Personal bindings and custom intents

The bindings an administrator creates apply to everyone. A user's own binding applies to their own requests only, unless an administrator locked the binding for everyone through the API.

**Create Intent** adds an intent in dotted notation, `product.feature.action`, with a description. It only matters when something asks for it by name: a platform that declares it, or your own integration. Until then, with no binding, it shows **Unused**. **Edit Intent**, in the intent's dialog or as the pencil of its row, renames it or changes its description; it is unavailable on a built-in intent. An intent with no binding left can be deleted through the API.

## When a feature does not appear in a connected product

- The intent shows **Unbound**: bind an agent. The **Usage** filter set to **Unbound** lists every intent in this case.
- The bound agent is disabled, or private: enable it, or share it with a group.
- The binding is for another vertical than the platform's: bind it to that vertical, or to **All verticals**.
- The intent is not listed: the platform never declared it. Check that it is registered and runs a version with the feature.
- The person's own rights in the connected product do not allow what the feature does.

:::example[Let the Velvet Harbor watcher summarize OpenCTI reports]
Sophie Laurent built "Velvet Harbor watcher", which drafts summaries of the reports on Velvet Harbor for the CTI team, in the style the team's briefings use. She wants the **Summarize** action of OpenCTI's reports to use it instead of the built-in agent.

1. She shares the agent with the group **CTI team**, **Read only**, and tags it `opencti`.
2. Hélène Marchetti opens :menu[Orchestration > Intents], searches `cti.container_summary` and clicks its row: **OpenCTI Entity Insights** is bound, **builtin**, for `cti`.
3. She clicks **Add Binding**, picks "Velvet Harbor watcher", selects the vertical `cti`, and clicks **Add 1 Binding**.
4. In the `cti` group of the intent's dialog, she clicks **Increase priority** until "Velvet Harbor watcher" is above **OpenCTI Entity Insights**.

When Camille Roux summarizes the report "Velvet Harbor - fake summit booking requests to front desks" in OpenCTI, the summary comes from "Velvet Harbor watcher". OpenAEV and OpenCRQ, which do not use the `cti` vertical, are unchanged.
:::

## What's next

- [XTM One in OpenCTI](/xtm-one/use/in-opencti/)
- [XTM One in OpenAEV](/xtm-one/use/in-openaev/)
- [Create an agent](/xtm-one/build/agents/)
