---
title: Set what one agent run may spend
description: Cap the cost, tokens, tool calls and duration of one run of an XTM One agent on its Value tab, finish on a cheaper model instead of stopping, and understand how the platform budget applies.
doc_type: how-to
products: [xtm-one]
personas: [ai-builder, admin]
owner: docs-xtm-one
last_reviewed: 2026-11-10
edition: ee
migrated_from:
  - xtm-one:/usage/agent-budgets/
---

A run budget caps what one run of an AI agent may spend: one chat answer, one assignment run, or one step of a flow. It bounds the run on four dimensions at once (money, tokens, tool calls and time) and ends the run when any of them is reached, with an answer written from the work done so far. Budgets exist because the other limits have the wrong shape: one round of an agent can fan out into a dozen tool calls, and a change of model can multiply the cost of the same work without any setting changing. This page sets an agent's budget and explains how it combines with the platform's.

::diagram{id=xtm-one-run-budgets-layers}

## Before you start

- You can edit the agent: you own it, or it is shared with you for editing.
- To raise a limit above the platform budget, you are an administrator. Anyone who can edit an agent may lower any limit.
- The platform budget is set by an administrator, with the `AGENT_LOOP_*` variables of the **Agent run budget** section of the [configuration reference](/reference/xtm-one/configuration/). By default every dimension is `0`, which means no limit.

## Where a budget comes from

There are up to three layers, and the nearest wins, dimension by dimension:

1. The **Run budget** of the agent's **Value** tab. It applies wherever the agent runs, chat included.
2. For an agent that is a member of a fleet, the **Run defaults** of that fleet's **Fleet settings**. They fill only the limits the agent leaves empty and, like the agent's run budget, only an administrator sets them above the platform budget.
3. The platform budget, the default every run inherits and the ceiling nobody but an administrator can exceed.

A run budget that pins only the cost does not lift the token limit: the fleet's or the platform's still applies. The fleet layer counts only for the fleet the agent is a member of: an agent that another fleet calls on runs on its own fleet's defaults. A ready-made fleet applies its run defaults only once one of its automations is switched on, and the built-in agents it gathered from earlier versions keep their own budgets. See [Run agents as a team with fleets](/xtm-one/build/fleets/).

| | Cost | Tokens |
| --- | --- | --- |
| Agent's **Run budget** | $2.00 | (empty) |
| Platform | $5.00 | 1,000,000 |
| **Effective** | **$2.00** | **1,000,000** |

## Set the agent's budget

:::steps
1. Open :menu[Orchestration > Agents], the agent, then its **Value** tab. The **Run budget** card follows the agent's return on investment (ROI); each of its fields shows what it inherits when it is empty.
2. Fill only the limits you want to pin for this agent:

   | Field | Bounds | Note |
   | --- | --- | --- |
   | **Max cost (USD)** | The priced model spend of one run | Needs a price for the model: with none, cost is not enforced and the other limits still apply |
   | **Max tokens** | The tokens of one run, prompt and completion together | |
   | **Max tool calls** | The tool calls of one run | Counts the calls of a round that fans out, which the maximum number of rounds does not |
   | **Max duration (seconds)** | The wall-clock time of one run | The tighter of this and the run's own timeout applies |

3. Leave the other fields empty. An empty field, or `0`, means inherit, not zero. Use **Inherit** next to a field to hand it back to the platform.
4. Click **Save budget**.

::screenshot{id=xtm-one-run-budgets-tab alt="The Run budget card of an agent before any limit is set: Max cost (USD), Max tokens, Max tool calls and Max duration (seconds) all on No limit, the Final-answer reserve at 10%, and Finish on a cheaper model below."}
:::

Leave a field empty rather than typing the platform's value to be explicit: a typed value freezes a copy that no longer follows the platform when an administrator changes it.

A field above the platform budget is flagged before you save, with the platform's value: **Above the platform budget of {value}. Only an administrator can set this.** Ask an administrator to set it. Once they have, the value stays: you can keep editing the rest of the agent, without being able to raise it further.

A budget you did not type yourself follows the same rule. Duplicating an agent, importing one, adding one from the AI Catalog or restoring an older version brings any limit above the platform budget down to it. An administrator's copy keeps its values.

## Keep enough for the answer

A run that spends its whole budget on work would have nothing left for the call that writes the answer, and would return a raw digest of what its tools returned. The **Final-answer reserve** holds back a share of the cost and token ceilings for that closing call: work stops at the rest, and the answer may spend the full budget. The platform default is 10 %. Leave it empty to inherit.

## Finish on a cheaper model instead of stopping

A cost ceiling usually means "do not spend more than this at this quality", not "abandon the task". With **Finish on a cheaper model** on (**Enable the failsafe**), a run that reaches its cost ceiling switches to a cheaper model, once, and carries on. The rest of the run, its answer included, is charged at that model's prices, so the ceiling still holds. The agent is told of the switch, and the run's trace records it.

- **Fallback model** chooses the model. **Platform fallback model** uses the **Fallback Model** an administrator sets in :menu[Administration > Settings > AI and models > Models]: see [Set up the platform in Settings](/xtm-one/administer/settings/#ai-and-models).
- It acts on cost only: a cheaper model buys no extra tokens, tool calls or time.
- The switch is refused, and the run stops at its ceiling, when the fallback has no known price, when its context window cannot hold the agent's instructions, or when it would drop a structured output the run depends on.

The reserve, the failsafe and the fallback model need no administrator: none of them can raise what a run spends. The failsafe is off on the platform by default; the **Run budget** card says what the agent inherits.

## What happens when a budget runs out

The run stops calling the model and writes its answer from the work it completed, using the reserve. The answer says which limit was reached, so you get a shorter result rather than a failure.

:::warning
A limit lower than the cost or tokens of a single model call makes every run stop at once and return a summary of tool output, and the platform logs a warning at the start of each run. Raise the budget.
:::

:::example[Keep the night triage agent cheap, and let the weekly brief run long]
Sophie Laurent, the AI engineer of Grand Meridian Hotels, wants **Night shift triage** to answer each alert for a few cents, and the **Weekly Velvet Harbor brief** assignment of **Velvet Harbor watcher** to read every new report without stopping half-way. The platform budget, set by Marc Dupuis, is $5.00 and 1,000,000 tokens per run.

On **Night shift triage**, **Value** tab, **Run budget**:

| Field | Value |
| --- | --- |
| **Max cost (USD)** | `0.25` |
| **Max tool calls** | `40` |
| **Max tokens**, **Max duration (seconds)** | Empty: inherit 1,000,000 tokens and the platform duration |
| **Finish on a cheaper model** | **Enable the failsafe** on, **Fallback model** **Platform fallback model** |

On **Velvet Harbor watcher**, Sophie wants $8.00: the field shows **Above the platform budget of $5. Only an administrator can set this.** Hélène Marchetti, administrator, sets it. A long night alert now ends on the cheaper model instead of stopping at $0.25, and the Monday brief can spend up to $8.00.
:::

## What's next

- [Review what agents did](/xtm-one/use/work-history/), to read what a run spent and why it stopped.
- [Get answers about the platform's own activity](/xtm-one/use/platform-insights/), to see what each agent costs over a month.
- [Govern how AI is used](/xtm-one/administer/ai-governance/), for the platform's limits and quotas.
