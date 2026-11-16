---
title: XTM One in OpenAEV
description: What XTM One does in OpenAEV, from Ask Ariane to generated TTPs, detection rules, phishing pages and autonomous attacks, with whose rights an agent works, and what each edition brings.
doc_type: explanation
products:
  - xtm-one
  - openaev
personas: [validation-engineer, admin, ai-builder]
owner: docs-xtm-one
last_reviewed: 2026-11-08
migrated_from:
  - openaev:/integrations/xtm-suite/xtm-one/
  - xtm-one:/usage/xtm-suite/openaev/
---

XTM One is the AI of the XTM suite. In OpenAEV, it answers your questions about scenarios and results in the **Ask Ariane** assistant, finds the techniques of a threat report for the scenario assistant, writes detection rules, emails and landing pages, plans autonomous attacks, and lets your own AI client work with OpenAEV through its MCP server. Each time, an XTM One AI agent works with OpenAEV as you: it sees what you can see and changes only what you could change. This page explains what each feature does, how the pieces connect, who decides which agents answer, and what works without XTM One.

::diagram{id=xtm-one-in-openaev}

## What the AI does for your work

| Your goal | Where | Edition |
| --- | --- | --- |
| Ask which techniques a simulation did not prevent, or draft a readout for the CISO | **Ask Ariane**, in the top bar | Enterprise |
| Find the techniques of a threat report and turn them into injects | **Suggest TTPs with XTM One**, in the scenario assistant. See [Build a scenario that replays an attack](/openaev/use/scenarios/build/#generate-injects-from-techniques) | Enterprise |
| Write or rewrite the name, description or message of a scenario, a simulation, an inject or a media article | **Ask AI**, next to text fields | Enterprise |
| Get a detection rule for an action your tools missed | **Generate with AI**, on the **Remediations** tab of a payload action | Enterprise |
| Write the email or the landing page of a phishing exercise | **Generate with AI**, in the email and landing page editors. See [Run a phishing exercise](/openaev/use/phishing/) | Enterprise |
| Let an agent design and drive a chained attack on your scope | **AI builder** and the **Autonomous** launch of a chained scenario. See [Let AI run a penetration test](/openaev/use/attack-chaining/autonomous/) | Enterprise |
| See the exposure of the whole suite | **CTEM Command Center**, in the top bar | Community |
| Work with OpenAEV from Cursor, Claude Desktop or your own agent | The **XTM One MCP server** card of your profile | Community |

Every answer is written by a language model and can be wrong. Check the techniques, rules and texts an agent proposes before you run them on your environment.

## Ask Ariane, the assistant

**Ask Ariane**, in the top bar, opens a chat with an XTM One agent about your OpenAEV data. It shows when XTM One is configured and the AI terms are not turned off, and it works with the Enterprise Edition once someone with **Manage platform settings** has accepted the Filigran AI Terms. Until then, other users read **Ask Ariane isn't activated yet. Please reach out to your administrator to enable this feature.**

What the assistant knows when you ask:

- **The page you are on.** OpenAEV sends its address, so a question asked on a simulation is about that simulation.
- **The files you attach.** They go to XTM One with your message.

Everything else, the assistant reads from OpenAEV as you, with the tools of its agent. The answer shows in the panel as the agent writes it. An answer can produce a report or a presentation, such as a validation readout for the CISO. Its card opens it in XTM One, and downloads it through OpenAEV: a presentation as PDF or PowerPoint, a report as PDF. Your browser never talks to XTM One: OpenAEV relays each message.

The panel also carries what the XTM One chat offers around a conversation:

- **Your prompts.** The prompts you can use in XTM One: your own, those shared with you or your groups, and those of the company, newest first. A prompt set to **Hide from chat** in XTM One is not offered.
- **Your quota.** The count and the limit of your XTM One agent runs for the current period, as the XTM One chat shows them.
- **Rating an answer.** A thumbs up or down, with a comment of up to 2,000 characters. It is the same rating as in XTM One: rating again replaces it, and the managers of the agent read it.

Next to **Ask Ariane**, **CTEM Command Center** opens XTM One in a new tab, on the exposure posture across the products of the suite.

### Approve what the assistant does

An XTM One administrator can make tools ask before they run, such as every tool that changes OpenAEV: a scenario it would create, a simulation it would launch. When the agent wants to run one, the answer stops and, as soon as the agent asks, the panel shows **The agent needs your approval to run a tool:**, with the tool and its values. Nothing runs until you answer:

| Answer | Effect |
| --- | --- |
| **Yes** | Runs once. The next call asks again |
| **Decline** | Does not run. Add a reason, and the agent adapts to it |
| **Yes, always** | Runs, and stops asking you, including in scheduled runs nobody watches, until you revoke it in XTM One |

## The agents behind each feature

Each AI feature of OpenAEV is a task XTM One calls an intent. When OpenAEV registers with XTM One, it declares them with its business vertical, `aev`, and an XTM One administrator binds one or more agents to each, in :menu[Orchestration > Intents]. The agent with the highest priority answers by default; where several serve a task, **Ask AI** and **Suggest TTPs with XTM One** let you choose. XTM One comes with a built-in agent for each:

| Intent | What it does in OpenAEV | Built-in agent |
| --- | --- | --- |
| `global.assistant` | **Ask Ariane** | **CTEM Assistant**, which hands the conversation to the **OpenAEV Assistant** for a question about OpenAEV |
| `global.fix_spelling`, `global.make_it_shorter`, `global.make_it_longer`, `global.change_tone`, `global.summarize`, `global.explain` | **Ask AI** next to text fields | **OpenAEV Assistant** |
| `aev.message_generator` | **Ask AI** > **Generate a message**, in the content of an inject | **OpenAEV Assistant** |
| `aev.media_article_generator` | **Ask AI** > **Generate an article**, in a media article | **OpenAEV Assistant** |
| `cti.ttp_harvester` | **Suggest TTPs with XTM One**, in the scenario assistant | **CTI TTP Harvester (Filigran AI)** |
| `aev.detection_rules_generator` | **Generate with AI**, detection rules | **AEV Detection Rules Generator** |
| `aev.phishing_email_html_generator`, `aev.phishing_landing_page_html_generator` | **Generate with AI**, phishing emails and landing pages | **AEV Phishing Email Generator**, **AEV Phishing Landing Page Generator** |

The autonomous attack of a chained scenario uses an orchestrator agent, and the specialist agents it may consult, chosen in **Settings** > **Customization** > **Autonomous attack**. See [Configure autonomous attack chaining](/openaev/configure/autonomous-attack/).

When no agent is bound to a task, the feature says **No agent available for this action. Ask your administrator to configure XTM One.** XTM One offers OpenAEV an agent only when all of this holds:

- The agent is enabled.
- The agent is company-managed or shared with at least one group. A private agent never reaches OpenAEV, even bound to an intent.
- Its binding applies to **All verticals** or to `aev`.

## Work with OpenAEV from XTM One

The link works the other way too: once OpenAEV is registered, people start from XTM One, and its agents read and write OpenAEV as them.

- **Design and review exercises.** The **OpenAEV Scenario Creator** builds a complete scenario in OpenAEV: it plans the injects, and has the **OpenAEV Payload Creator** add the actions they need to the threat arsenal. The **OpenAEV Assistant** reviews the results of a simulation with you.
- **Turn results into a deliverable.** The **OpenAEV Assistant** writes a validation readout as a report, or a deck for a review meeting, in your templates.
- **Let agents work on a schedule.** Longer work runs in the background, and its runs show in the [work history](/xtm-one/use/work-history/) of XTM One.

## What an agent can reach, and with which rights

An agent never holds an OpenAEV token of its own. Each request carries your identity:

:::steps
1. When you use an AI feature, OpenAEV signs a token for you, valid 10 minutes, that names your email address, and sends your request to XTM One with it.
2. XTM One runs the agent. When the agent needs data, XTM One calls the OpenAEV API with a token it signs for the same email.
3. OpenAEV checks the signature against the keys XTM One publishes, finds the account with that email, and applies its capabilities and grants.
:::

The consequences:

- **An agent sees what you see**, in your tenant, and changes only what your capabilities allow, such as **Manage assessment** to create a scenario or **Launch assessment** to launch it.
- **Your email must be the same in both products.** An account with no match in OpenAEV gets no data.
- **What leaves OpenAEV** is your request, the address of the page, the files you attach, and what the agent reads through the API. XTM One sends them to the language models its administrator configured.

## Use OpenAEV from your own AI client

XTM One exposes OpenAEV to any client of the Model Context Protocol (MCP), such as Cursor, Claude Desktop or an agent your team builds. The client works with scenarios, simulations, payloads, atomic tests and findings, with your permissions. It works with the Community Edition of XTM One, and nothing is installed in OpenAEV: the server exists as soon as the platform is registered.

On your OpenAEV profile, the **XTM One MCP server** card gives the **MCP endpoint URL**, the address of XTM One followed by `/mcp/openaev`. The client authenticates with a personal API key, created in XTM One under your profile. **Manage in XTM One** opens the MCP page of your XTM One profile, with the connection status and a client configuration to copy.

:::example[Ask Cursor for the techniques Meridian EDR missed]
The validation engineer of Grand Meridian Hotels keeps a personal XTM One API key in the environment variable `XTM_ONE_API_KEY`, and adds OpenAEV to `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "openaev": {
      "url": "https://xtm-one.example.com/mcp/openaev",
      "headers": {
        "Authorization": "Bearer ${env:XTM_ONE_API_KEY}"
      }
    }
  }
}
```

Asked "Which injects of the last simulation of Velvet Harbor - front-desk intrusion were not prevented?", Cursor reads the simulation with the engineer's own rights and answers with the Web Protocols inject. A `401` means the key is wrong, a `403` that an administrator turned the platform MCP servers off in XTM One, a `404` that no OpenAEV is registered with XTM One.
:::

## Govern AI on the platform

This part is for administrators: who may use AI, under which terms, and with which agents.

**Enterprise Edition.** Every AI feature but **CTEM Command Center** and the MCP server needs it. It can come from a license of OpenAEV's own, or from the Filigran license of the connected XTM One: when XTM One returns a license signed by Filigran that covers this platform, OpenAEV turns the Enterprise Edition on. The **Enterprise Edition** card of **Settings** > **Filigran Experience** then shows **Filigran license (through XTM One)** as its **License source**. The license of OpenAEV's own wins when both are valid, and its **License source** reads **OpenAEV license**, or **Filigran license** when the Filigran license of XTM One was entered in OpenAEV itself. See [Editions and licensing](/start/editions/#what-the-enterprise-edition-adds-to-openaev).

**The Filigran AI Terms.** On that card, the **XTM One (Agentic IA)** row has **Validate the Filigran AI Terms** until someone with **Manage platform settings** accepts them; the same dialog opens at their first click on an AI feature. Afterwards, the row is a switch: turned off, every AI entry point of OpenAEV disappears.

**In XTM One**, its administrators bind the agents to each intent, decide which tools ask for approval, set the quotas, and can turn the MCP servers off. See [XTM One](/xtm-one/).

## Connect OpenAEV to XTM One

A platform engineer sets `openaev.xtm.one.url` (`OPENAEV_XTM_ONE_URL`), the address of XTM One as OpenAEV reaches it, and `openaev.xtm.one.token` (`OPENAEV_XTM_ONE_TOKEN`), the registration token XTM One shares with its platforms. Both are needed. OpenAEV registers 30 seconds after it starts, then every 5 minutes, and sends its license, its address and its intents each time. See [Connect the modules to each other](/deploy/configure/connect-modules/#register-the-platforms-with-xtm-one) and the [configuration reference](/reference/openaev/configuration/#openaev-xtm).

## With or without XTM One

| Feature | Without XTM One | With XTM One |
| --- | --- | --- |
| **Ask AI** on text fields | A language model set with the `ai` keys of the configuration, with the Enterprise Edition | The agent bound to each task; you can pick it |
| Detection rules | **Use Ariane**, only when an administrator has configured a detection rule service, with the Enterprise Edition | **Generate with AI**, the bound agent |
| **Ask Ariane**, **Suggest TTPs with XTM One**, phishing generation, autonomous attack, **CTEM Command Center**, MCP server | Not available | Available |

Without XTM One, an administrator connects a language model for **Ask AI** with `ai.enabled`, `ai.type` (`mistralai` or `openai`), `ai.endpoint`, `ai.token` and `ai.model`: see the [configuration reference](/reference/openaev/configuration/#ai).

## What's next

- [Build a scenario that replays an attack](/openaev/use/scenarios/build/), with the techniques XTM One suggests.
- [Let AI run a penetration test](/openaev/use/attack-chaining/autonomous/), the autonomous attack.
- [Connect the modules to each other](/deploy/configure/connect-modules/), to register OpenAEV with XTM One.
- [XTM One in OpenCTI](/xtm-one/use/in-opencti/) and [XTM One in OpenCRQ](/xtm-one/use/in-opencrq/), the same assistant in the other products.
