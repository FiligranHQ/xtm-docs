---
title: Write reusable prompts
description: Keep the instructions your teams repeat in a prompt library, insert them in chat, attach them to agents, and track every change of their wording.
doc_type: how-to
products: [xtm-one]
personas: [ai-builder, cti-analyst]
owner: docs-xtm-one
last_reviewed: 2026-11-08
edition: ee
migrated_from:
  - xtm-one:/usage/resources/prompts/
---

A prompt is a piece of instruction text you write once and reuse: a person inserts it in a chat message, and an agent that has it attached receives it in its system prompt on every conversation and run. Keep in the library the wording a team must not improvise, such as how to summarize a report for executives, so every analyst and every agent asks the same way. This page creates a prompt, uses it in both places, and changes it safely. How prompts fit with the other building blocks is in [What agents are made of](/xtm-one/concepts/agents/).

::diagram{id=xtm-one-build-prompts-reuse}

## Before you start

- Prompts are an AI area: the platform runs the Enterprise Edition and AI is switched on. Otherwise :menu[Agent Studio > Prompts] stays listed but marked unavailable, and **Add Prompt** opens the Enterprise Edition notice.
- Any user can create a private prompt. Only an administrator can make it **Company-managed prompt** (shared with all users, editable by administrators only).

## Create a prompt

:::steps
1. Open :menu[Agent Studio > Prompts] and click **Add Prompt**. The **Create Prompt** page opens.
2. Fill **Prompt Name** and, optionally, **Tags**. The name is what people pick in the chat and on an agent, so say what the prompt does.
3. In **Description**, say when to use the prompt. It shows under the name in the chat picker.
4. Write **Prompt Content**. To reuse a stored value, such as an email address, click **Variable** above the field and pick it: the editor inserts a reference that is replaced by its value when the prompt is used (see [Store values agents reuse in variables](/xtm-one/build/variables/)).
5. As an administrator, turn on **Company-managed prompt** to offer the prompt to everyone. A confirmation lists who gains access to what the prompt references, such as variables.
6. Click **Create Prompt**. The prompt's page opens on **Overview**.
:::

::screenshot{id=xtm-one-build-prompts-create alt="The Configuration of the prompt Summarize a report for executives: its name, its description, and its Content at version v1, which asks for a five-line summary for the executive committee of Grand Meridian Hotels."}

## Use it in a chat

In the chat, the prompt button of the message box lists the prompts you can use: yours, the ones shared with you and the company-managed ones, newest first, each with its description. Picking one puts its content in the message you are writing, so you can add the report or the question it applies to before sending.

A prompt with **Hide from chat** on is not listed there. Use it for prompts written only for agents, which would confuse someone chatting.

## Attach it to an agent

On an agent's page, open **Configuration**, then the **Knowledge** section. The **Prompts** card lists the prompts attached to the agent; click **Edit**, pick the prompts, and **Save**. Their content is added to the agent's system prompt at runtime, after its persona, in every chat and every run. When you create an agent with the wizard, the **Persona** step also offers the prompt library: picking a prompt there copies its text into the persona, which you then edit freely. See [Create an agent](/xtm-one/build/agents/).

An attached prompt stays linked: when someone edits it, every agent that carries it uses the new wording at its next message or run. A copied persona does not change.

A company-managed agent accepts only company-managed prompts, so that what it says does not depend on one person's private library. The card then says **Only company-managed prompts are shown.**

## Change a prompt safely

A prompt's page has four tabs:

| Tab | What it is for |
| --- | --- |
| **Overview** | The **Agents Using** count, the description, the tags and a **Content Preview**. Read the count before an edit: it is the number of agents whose behavior changes with it. |
| **Configuration** | Edit the name, description, tags and content, **Hide from chat**, and the sharing. **Commit message for content change (optional)** records why the wording changed. |
| **Versioning** | Every saved content as a numbered version. Select two to **Compare** them, **Side by side** or **Unified** in one column, **Rollback** to an earlier one (the current content is kept as a version), or **Lock version** to protect a version from deletion. |
| **Activity** | Who changed the prompt, and when. |

Sharing a prompt with a group or with people, transferring its ownership and the read-only states are the same for every resource: see [Share agents and resources](/xtm-one/build/sharing/).

:::example[One wording for every executive summary of Velvet Harbor]
Sophie Laurent, the AI engineer of Grand Meridian Hotels, wants the CTI team and the **Velvet Harbor watcher** agent to summarize reports for the CISO the same way. She creates this prompt:

| Field | Value |
| --- | --- |
| **Prompt Name** | `Summarize a report for executives` |
| **Tags** | `cti`, `executive` |
| **Description** | `Turns a threat report into a five-line summary for the CISO and the crisis cell.` |
| **Prompt Content** | `Summarize the report below for Hélène Marchetti, CISO of Grand Meridian Hotels. In five lines at most: who is behind it, what they did, whether our hotels in France and Spain are exposed, what we already detect, and the one decision you recommend. No jargon, no indicators.` |
| **Company-managed prompt** | Off: she shares it with the **CTI team** group from **Configuration** instead |

She attaches it to **Velvet Harbor watcher** in the **Prompts** card. Camille Roux, CTI analyst, inserts the same prompt in the chat with the prompt button when the CISO asks about a new report. When the CISO asks for one more line on cost, Sophie edits the content with the commit message `Add the cost line`, and the agent's next summary follows it.
:::

:::xtm-one[Let the General Assistant draft the wording]
Ask the General Assistant in the chat to draft or tighten a prompt, then paste the result in **Prompt Content**. A prompt is plain text: the assistant's wording is yours to review before anyone reuses it.
:::

## Check that it works

Open the prompt's **Overview**: **Agents Using** counts the agents it is attached to. In a chat with one of those agents, ask for the task the prompt describes: the answer follows its wording. In a new chat, the prompt button of the message box lists the prompt unless **Hide from chat** is on.

## What's next

- [Teach agents a procedure with skills](/xtm-one/build/skills/), for longer methods an agent loads only when a task calls for them.
- [Create an agent](/xtm-one/build/agents/).
- [Share agents and resources](/xtm-one/build/sharing/).
