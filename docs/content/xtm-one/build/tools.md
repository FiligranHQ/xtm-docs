---
title: Give agents built-in and custom tools
description: Know the built-in tools an agent can use, then write a JavaScript or Python tool of your own, test it, and give it to an agent.
doc_type: how-to
products: [xtm-one]
personas: [ai-builder, developer, admin]
owner: docs-xtm-one
last_reviewed: 2026-11-08
edition: ee
migrated_from:
  - xtm-one:/usage/resources/tools/
---

A tool is an action an agent can take beyond writing text: fetch a web page, run Python, generate a file, open a signed-in browser. XTM One ships built-in tools, and lets you write custom tools: a short JavaScript or Python function with declared inputs, which an agent calls like any other tool and which runs in a sandbox. This page shows what the built-in tools give an agent, then writes, tests and attaches a custom tool. Integrations and MCP servers bring the tools of whole external services: see [Connect agents to your tools with integrations](/xtm-one/build/integrations/) and [Connect an MCP server](/xtm-one/build/mcp-servers/).

::diagram{id=xtm-one-build-tools-call}

## The built-in tools

:menu[Tools & Integrations > Tools] opens on **Built-in Tools**: one card per tool the platform offers, with its category and the number of functions it grants on this platform. A built-in tool is a set of related functions: **Web**, for example, reads pages, saves images and drives a browser. Opening a card shows the **Agents Using This Tool** and its runs. Search the list by a tool's name or description; the view switch shows it as cards or as a table, with its **Category** column. You turn a built-in tool on for an agent in its **Configuration**, **Tooling** section, **Built-in Tools** card, or in the **Tools** step of the creation wizard.

| Tool | What it gives the agent |
| --- | --- |
| **Platform** | Manage the platform from a conversation: create and configure agents, assignments, knowledge bases, skills, variables, integrations, MCP servers, custom tools and fleets. It acts as the person the agent works for, and offers only what that person may do. |
| **Web** | Read public web pages and documents, such as a link a user shares, save a public image to the conversation, and drive a headless browser to navigate, click, fill forms and take screenshots; with [browser profiles](/xtm-one/build/browser-profiles/), signed in to your applications. |
| **Files** | Generate downloadable files, read and edit the conversation's files, render HTML to an image, and generate images from a text prompt. The image model is a platform-wide setting an administrator changes in **Image Generation Model**, on the tool's **Configuration** tab. |
| **Report Generation**, **Presentation Generation** | Executive reports and slide decks, branded by [templates](/xtm-one/build/templates/). |
| **Design** | Find a [template](/xtm-one/build/templates/) and read its layouts and assets, write one from a conversation, and read the exact palette and typefaces of a PDF, a PowerPoint file or an HTML page. |
| **Code Interpreter** | Run Python in a sandbox for calculations and charts. Off by default on XTM One: an administrator turns off **Disable the code interpreter** in :menu[Administration > Settings > Access and security > Security options]. |
| **Threat Intelligence** | Check the structure and references of a STIX 2.1 bundle before it is imported, compare two bundles, analyse a timeline, and run statistical tests and FIMI metrics, each result with its formula. |
| **Coordination** | Coordinate agents across runs: a shared work queue, a key-value store, a large working corpus. |
| **Traces** | Read the agent's own traces, to review how its runs went. It also reads other agents' traces: for an administrator, those of any agent they can see; for anyone else, those of the agents listed in its **Trace supervision allow-list**, which shows under the card while you edit it. |
| **Platform Insights** | Answer questions about the platform's AI usage and cost, its objects and its audit trail, with charts: see [Get answers about the platform's own activity](/xtm-one/use/platform-insights/). |
| **Computer Use (Runners)** | Drive a computer enrolled with XTM One Desktop: see [Let agents act on a computer with XTM One Desktop](/xtm-one/build/runners/). |

Give an agent only the tools its job needs: every tool is something it can act through. When the wizard's **Tools** step is left empty, the agent starts with the platform's default tool set, which you can change on its page afterwards.

### Keep only some functions of a tool

An agent can carry a built-in tool without all of its functions: an agent that only reads web pages has no use for the browser.

:::steps
1. Open the agent, its **Configuration** tab, then **Tooling**.
2. In the **Built-in Tools** card, click **Edit**. Each enabled tool with more than one function shows **All functions** under its name.
3. Click it. The **Function Scope** dialog lists the tool's functions: tick only the ones the agent needs (the search, **All functions** and **None** help), then click **Apply**.
4. Click **Save**. The button now reads how many functions the agent keeps, for example "2 of 3 functions selected", and outside edit mode the tool shows the same count next to its name.
:::

::screenshot{id=xtm-one-build-tools-function-scope alt="The Function Scope dialog of Traces for the Crisis briefer agent: Inspect my traces and Get trace payload checked, Inspect agent traces unchecked, 2 of 3 functions selected, with the search, the All functions and None buttons, and Apply."}

An agent set up with one of the earlier, narrower tools, such as **Browser** or **Inspect My Traces**, keeps exactly what it had: it shows the tool that includes it now (**Web**, **Traces**), limited to those functions. Open the scope to give it the rest.

An assignment can narrow the functions further for its own runs, never beyond the agent's: on its **Prompt & Bindings** tab, the **Built-in Tool Scope** card lists the agent's tools that keep more than one function. Click **Edit**, then a tool's button, choose in the same dialog, and click **Save**.

## Before you write a custom tool

- Creating a custom tool requires the Enterprise Edition. Custom tools are not an AI feature: authoring them keeps working with AI switched off.
- Custom tools are off by default on XTM One. An administrator turns off **Disable custom tools** in :menu[Administration > Settings > Access and security > Security options], or sets `DISABLE_CUSTOM_TOOLS` to `false`. While they are off, the **Custom Tools** tab and **Add Tool** are hidden, and an agent never calls one.
- Tools run in a hardened Docker container when the platform can start one, otherwise in a restricted subprocess. A platform engineer chooses with `CUSTOM_TOOL_SANDBOX` (`auto` by default). See [Security switches](/reference/xtm-one/configuration/#security-switches) in the configuration reference.

## Write and test a custom tool

:::steps
1. Open :menu[Tools & Integrations > Tools], select **Custom Tools**, and click **Add Tool**. The **Create Custom Tool** page opens.
2. Fill **Tool Name** (required), the name the model calls the tool by (for example `booking_lookup`). Optionally add **Icon URL (optional)** and **Tags**.
3. Write **Description** for the model: what the tool does and when to call it. The model decides to call a tool from this text alone.
4. Under **Input Parameters**, click **Add Parameter** for each input: **Name**, **Type** (`string`, `number`, `boolean` or `integer`), **Description** and **Required**. The model fills them when it calls the tool.
5. Choose the **Language** and write the function:
   - **JavaScript Function**: read a parameter as `$param_name`, call web services with `fetch`, and write the result to the standard output (`console.log`).
   - **Python Function**: read a parameter as a plain variable (`param_name`) and `print()` the result. Network libraries are blocked: keep Python tools for processing and formatting data.

   To reuse a stored value such as an API key, click **Variable** and pick it rather than typing it in the code: see [Store values agents reuse in variables](/xtm-one/build/variables/). **Generate with AI** writes the code from the name, the description and the parameters you declared; on existing code it reads **Improve with AI**. Nothing is saved until you save the tool.
6. Set **Execution Timeout (seconds)**, from 1 to 3600: the platform stops a run still going after that time. The default is 300 seconds.
7. As an administrator, turn on **Company-managed tool** to offer it to every user (only administrators edit it then).
8. Click **Create Tool**. The tool's page opens.
9. Click **Test Tool**, write the **Arguments (JSON)** and click **Run**. The dialog shows **Success** or **Failed** with the output, exactly what an agent would read.
:::

::screenshot{id=xtm-one-build-tools-create alt="The booking_lookup tool in edit mode, with the same fields as the creation page: its description for the model, the required reference parameter, the JavaScript function that reads $reference and sends the BOOKING_API_TOKEN variable by its reference, and a 30-second timeout."}

The tool's page has three tabs: **Overview** (its description, language, timeout, parameters and **Agents Using**), **Configuration** (everything above, plus **Hide from chat**, which keeps the tool out of the chat's tool picker while agents can still use it, and the sharing) and **Activity**, the runs of the agents that use it, with their total next to the search and one pager under the list.

The **Custom Tools** tab lists the tools you can see, as cards or as a table with their **Language**, **Parameters**, **Timeout**, **Calls**, **Tags** and **Updated** date. Search them, narrow them with **Mine**, **Company**, **Shared** or **All**, and filter them by tag; **Clear all filters** resets the search and the tags. **Columns** hides the columns you do not need, and your browser keeps that choice and the page size for your next visit.

## Give it to an agent

On the agent's page, open **Configuration**, then the **Tooling** section. In the **Custom Tools** card, click **Edit**, select the tool and **Save**. The agent can call it from its next message or run. To change who may reuse the tool, see [Share agents and resources](/xtm-one/build/sharing/).

:::example[Look up a guest booking for the night shift]
Paul Garnier, SOC analyst on the night shift at Grand Meridian Hotels, often needs to know whether a booking named in a phishing email exists. Sophie Laurent, the AI engineer, writes a tool for the **Night shift triage** agent against the booking platform's internal API:

| Field | Value |
| --- | --- |
| **Tool Name** | `booking_lookup` |
| **Description** | `Checks whether a booking reference exists in the Grand Meridian booking platform and returns the hotel, the arrival date and the booking channel. Call it when an email or an alert names a booking reference.` |
| **Input Parameters** | `reference`, `string`, `The booking reference, for example GM-2026-118734`, **Required** on |
| **Language** | JavaScript |
| **Execution Timeout (seconds)** | `30` |

```javascript
const response = await fetch(`https://booking.grand-meridian.example/api/bookings/${encodeURIComponent($reference)}`, {
  headers: { Authorization: `Bearer {{var_id.7f3c2a10-5b8e-4d21-9a64-0c1e8b2f4d75}}` }
});
if (response.status === 404) {
  console.log(JSON.stringify({ found: false }));
} else {
  const booking = await response.json();
  console.log(JSON.stringify({ found: true, hotel: booking.hotel, arrival: booking.arrival, channel: booking.channel }));
}
```

The reference in the header is the secret variable `BOOKING_API_TOKEN`, inserted with **Variable**: the token never appears in the code, and traces show `[secret]`. She tests it with `{"reference": "GM-2026-118734"}`, then adds it to **Night shift triage**. When an alert quotes a fake summit booking like the one Velvet Harbor sent on 9 January 2026, the agent answers that no such booking exists.
:::

:::xtm-one[Packaged skills bring their own tools]
A skill package can carry scripts: importing it creates one custom tool per script, which follows the skill onto every agent it is attached to. See [Teach agents a procedure with skills](/xtm-one/build/skills/).
:::

## Check that it works

The tool's **Overview** counts the agent under **Agents Using**. In a chat with the agent, ask the question the tool answers: the answer shows the tool call and its result. A tool that times out or fails returns its error to the agent, which says so.

## What's next

- [Connect agents to your tools with integrations](/xtm-one/build/integrations/), for services XTM One already knows.
- [Connect an MCP server](/xtm-one/build/mcp-servers/), for a whole set of tools served by another system.
- [Create an agent](/xtm-one/build/agents/).
