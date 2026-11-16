---
title: Store values agents reuse in variables
description: Keep an address, a URL or a credential in one variable, reference it from personas, prompts, skills and custom tools, and know who it resolves for.
doc_type: how-to
products: [xtm-one]
personas: [ai-builder, admin]
owner: docs-xtm-one
last_reviewed: 2026-11-08
edition: ee
migrated_from:
  - xtm-one:/usage/resources/variables/
---

A variable is a named value you store once in XTM One and reference wherever an agent reads text: its persona and guardrails, an assignment prompt, a prompt of the library, a skill and the code of a custom tool. When the work runs, XTM One replaces the reference with the value, so changing an escalation address or rotating a key happens in one place. A variable can be **Secret**: its value is encrypted, hidden on screen and kept out of what XTM One stores of a run. This page enables variables, creates one, references it, and explains who a reference resolves for.

::diagram{id=xtm-one-build-variables-resolution}

## Before you start

- Creating a variable requires the Enterprise Edition. Variables are not an AI feature: they keep working with AI switched off.
- Variables are off by default on XTM One. An administrator turns off **Disable custom variables** in :menu[Settings > Access and security], or sets `DISABLE_CUSTOM_VARIABLES` to `false` (see the [configuration reference](/reference/xtm-one/configuration/)). The variable, when set, wins and locks the switch. While variables are off, the **Variables** tab of :menu[Tools & Integrations > Connections] and the **Variable** button of the editors are hidden.
- To read a secret from a vault instead of storing it, an administrator must have granted you that vault: see [Secret vaults](/xtm-one/administer/secret-vaults/).

## Create a variable

:::steps
1. Open :menu[Tools & Integrations > Connections], select the **Variables** tab and click **New Variable**. The tab sits beside **Browser profiles** and, when relays are on, **Relays**.
2. Fill **Name**, the name people see in the variable picker. References do not use it, so you can rename a variable without breaking anything.
3. Fill **Value**.
4. Turn on **Secret** for a credential or any value that must not be shown (**Encrypted at rest, hidden in UI**). The value field then hides what you type, and editing it later never shows the stored value: type a new one only to replace it. With a vault granted, **From a vault** picks the secret instead, and the field shows the vault, the path and the key; XTM One reads the value from the vault each time a run needs it.
5. Add a **Description** that says where the variable is used, and **Tags** if you keep many.
6. Click **Save**. The variable appears as a card, its value masked behind a **Secret** badge when it is secret.
:::

::screenshot{id=xtm-one-build-variables-board alt="The Variables tab of Connections for Sophie Laurent: BOOKING_API_TOKEN with its Secret badge and its value masked, and SOC_ESCALATION_EMAIL showing its address, each with its Copy reference button and its tags."}

Clicking a card opens the same dialog to edit or **Delete** the variable. There is no detail page.

The view switch shows the variables as cards or as a table with their **Value**, **Type**, **Tags** and **Updated** date, where clicking a row opens the same dialog. Search them by name or description and filter them by tag; **Clear all filters** resets both. **Columns** hides the columns you do not need, and your browser keeps that choice and the page size for your next visit.

## Reference it

- In a persona, a guardrail, an assignment prompt, a prompt, a skill or a custom tool, click **Variable** next to the field and pick the variable. The editor inserts its reference at the cursor.
- Or click **Copy reference** on the variable's card and paste it. A reference is `{{var_id.` followed by the variable's identifier and `}}`, never its name: two variables with the same name never shadow each other.
- At the bottom of the **Variable** list, **Create a secret variable from a vault secret** creates a secret variable that holds a vault reference, visible only to you until you share it, and inserts it. A prompt or a persona never holds a vault reference itself.
- In the **API key** or the **Client secret** of an [integration](/xtm-one/build/integrations/), paste the reference of a **secret** variable, alone or inside the text the service expects (`Bearer` followed by the reference, for example). XTM One refuses to save a reference to a variable that is not secret, or that you or the integration's owner cannot see. The integration's page then shows the field under **Read from a variable:**, by the variable's name, and XTM One reads the value each time a call needs it, as the integration's owner sees it.

An agent's page lists the variables it references that have no value yet, and the references to variables that no longer exist.

## Who a reference resolves for

A reference is replaced only for the person the work runs as, and only if that person can see the variable: their own variables, the ones shared with them or one of their groups, and company-managed ones not restricted to an audience they are outside of. Administrators get no exception.

| The work | Runs as |
| --- | --- |
| A chat | The person chatting |
| An assignment run | The person who triggered it, else its run-as user, else its owner |
| A flow | Its owner, or the person who ran it |
| An integration's **API key** or **Client secret** | The integration's owner; an integration with no owner reads company-managed variables only |

When no value resolves, the reference reaches the model as written. An integration credential fails instead: when its variable is deleted, no longer secret, empty or no longer visible to the owner, the calls that need it fail and the integration's page shows **A secret could not be read** until the credential references a secret variable its owner can read. So when an agent's automations run as someone else, make sure that person can see the variable: [Share agents and resources](/xtm-one/build/sharing/) explains who sees what.

## Where a secret value goes

A secret value reaches the model and the tools it calls, and nothing else keeps it in clear:

- What a run or a chat stores (messages, tool calls and their results, an approval waiting for a reviewer), traces, server logs and the agent's memory hold the marker `[secret]` instead of the value. When the work resumes, the model and the tools get the value back.
- When the model repeats the value in its answer, people see `[secret]`, in the chat, in channels, in notifications and in the conversation's title.
- A command an agent runs on XTM One Desktop gets the value on your computer; its history and what it prints back show `[secret]`. See [Let agents act on a computer with XTM One Desktop](/xtm-one/build/runners/).
- A value shorter than 8 characters is not replaced, because it would mask unrelated words and numbers. Real credentials are longer.
- Each chat message resolves the variables you may read at that moment: once a variable is no longer shared with you, the next message no longer gets its value.

:::example[One escalation address for every SOC agent]
The SOC of Grand Meridian Hotels escalates to an on-call mailbox that changes with the rota. Sophie Laurent, the AI engineer, stores it once:

| Field | Value |
| --- | --- |
| **Name** | `SOC_ESCALATION_EMAIL` |
| **Value** | `soc-oncall@grand-meridian.example` |
| **Secret** | Off: an address is not a credential |
| **Description** | `Where the Night shift triage agent sends escalations. Updated by Julien Bernard with the rota.` |
| **Tags** | `soc` |

In the persona of **Night shift triage**, she writes `Escalate a confirmed intrusion to`, clicks **Variable**, picks `SOC_ESCALATION_EMAIL`, and ends the sentence with `and say so in your answer.` The editor writes the variable's reference with its identifier. When Julien Bernard, the SOC manager, changes the on-call mailbox, the next triage of Paul Garnier's night shift uses the new address, with no agent edited.
:::

## Check that it works

In a chat with the agent, ask the question that uses the value: the answer uses the current value, or `[secret]` for a secret one. If the reference comes back as written, the person the work ran as cannot see the variable.

## What's next

- [Write reusable prompts](/xtm-one/build/prompts/) and [Write custom tools](/xtm-one/build/tools/), which reference variables.
- [Secret vaults](/xtm-one/administer/secret-vaults/), to read credentials from 1Password, HashiCorp Vault or a cloud secret manager.
- The [configuration reference](/reference/xtm-one/configuration/).
