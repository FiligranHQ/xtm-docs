---
title: XTM One in OpenCTI
description: What the AI of the XTM platform does in OpenCTI, how OpenCTI hands each request to an XTM One agent, which data an agent reaches with whose rights, and who governs it.
doc_type: explanation
products:
  - xtm-one
  - opencti
personas: [cti-analyst, soc-analyst, admin]
owner: docs-xtm-one
last_reviewed: 2026-11-08
migrated_from:
  - opencti:/usage/copilot/
  - opencti:/deployment/mcp-server/
  - opencti:/reference/artificial-intelligence/
  - xtm-one:/usage/xtm-suite/opencti/
  - xtm-one:/usage/xtm-suite/common-experience/
  - xtm-one:/usage/xtm-suite/indicator-decay/
---

XTM One is the AI of the XTM platform. In OpenCTI, it answers your questions about the knowledge in the **Ask Ariane** assistant, writes the AI insights of a threat, rewrites your texts, turns a question into search filters, extracts knowledge from documents and takes on playbook steps. Each time, an XTM One AI agent works with OpenCTI as you: it sees what you can see and changes only what you could change. This page explains how the pieces connect, what an agent can reach, who decides which agents answer, and what works without XTM One.

::diagram{id=xtm-one-in-opencti}

::::only{opencti="<7.260811"}
:::note
In this version, only **Ask Ariane** uses XTM One. The other AI features of OpenCTI call the language model set in the platform configuration, as [With or without XTM One](#with-or-without-xtm-one) describes. AI agents in screens, AI agent steps in playbooks and the MCP server come with a later release of OpenCTI.
:::
::::

## What the AI does for your work

| Your goal | Where | Details |
| --- | --- | --- |
| Get an answer from the knowledge, such as "what do we know about Velvet Harbor?" | **Ask Ariane**, in the top bar | [Ask Ariane, the assistant](#ask-ariane-the-assistant) below |
| Brief management on a threat, a sector or a report | **AI Insights**, on the entity page | [Read the home dashboard and AI summaries](/opencti/use/explore/insights/) |
| Rewrite a description for executives, fix it, shorten it or summarize it | **Ask AI**, next to text fields | [Refine content with AI](/opencti/use/enrich/refine-content/) |
| Draft a brief from the knowledge of a report or a case | **Generate an export**, format **Ask AI (multiple formats supported)** | [Refine content with AI](/opencti/use/enrich/refine-content/#generate-a-report-from-a-container) |
| Find entities with a question in plain language | **Ask AI**, in the search bar | [Search for knowledge](/opencti/use/explore/search/#ask-a-question-in-plain-language) |
| Turn a vendor PDF into entities, observables and relationships | An import connector, in **Data** > **Import** | [Import files](/opencti/use/ingest/files/) |
| Let an agent classify, enrich or summarize new intelligence | A playbook step | [Automate with playbooks](/opencti/use/automate/playbooks/) |

Every answer is written by a language model and can be wrong. Nothing an assistant proposes in a form changes the knowledge until you accept it; check names, dates and facts against the reports first.

::::only{opencti=">=7.260811"}
## Ask Ariane, the assistant

**Ask Ariane**, in the top bar, opens a chat with an XTM One agent about the knowledge of your platform. It shows when your role has the **Access knowledge** capability, the platform has an Enterprise Edition license, and an administrator has accepted the Filigran AI Terms.

What the assistant knows when you ask:

- **The page you are on.** OpenCTI sends the address of the page, without its filters, so "Analyze this indicator" on an indicator page is about that indicator.
- **The draft you work in.** In a draft, the border of the panel takes the warning color, and the assistant works on the draft, not on the live knowledge.
- **The files you attach.** They go to XTM One with your message.

::screenshot{id=opencti-xtm-one-ask-ariane-panel alt="The Ask Ariane panel open at the right of an OpenCTI page, with its suggested questions, such as What are the latest threats?, above the message field."}

Everything else, the assistant reads from OpenCTI as you, with the tools its agent has. It can open entities, list relationships, read reports and, when its agent allows it, create or change knowledge. Links in an answer open in OpenCTI.

The panel opens as a **Sidebar** you can widen, or **Floating**; your browser remembers the choice. **History** lists your past conversations and **New conversation** starts another.

:::only{opencti=">=7.261030"}
The panel also carries what the XTM One chat offers around a conversation:

- **Your prompts.** The prompts you can use in XTM One: your own, those shared with you or your groups, and those of the company, newest first. A prompt set to **Hide from chat** in XTM One is not offered.
- **Your quota.** The count and the limit of your XTM One agent runs for the current period, as the XTM One chat shows them.
- **Rating an answer.** A thumbs up or down, with a comment of up to 2,000 characters. It is the same rating as in XTM One: rating again replaces it, and the managers of the agent read it.
- **Reports and decks.** When an answer produces a report or a slide deck, its card opens it in XTM One and downloads it through OpenCTI: a deck as PDF or PowerPoint, a report as PDF.
:::

Next to **Ask Ariane**, **CTEM Command Center** opens XTM One in a new tab, on the exposure posture of the whole XTM platform.

:::example[Check an IP address from an alert in seconds]
The SIEM flags outbound traffic from a hotel front-desk workstation to 203.0.113.10. Open the observable in **Observations** > **Observables**, click **Ask Ariane** and ask: "Is this IP linked to a known intrusion set, and since when?"

The assistant reads the observable, its relationships and the reports that contain it, and answers that 203.0.113.10 is a command and control server of Velvet Harbor, citing the reports of Example CERT. Open the cited reports before you escalate: the answer is only as good as the knowledge and the agent.
:::

:::only{opencti=">=7.261030"}
### Approve what the assistant does

An XTM One administrator can make tools ask before they run, for example every tool that changes OpenCTI. When the assistant wants to run one, the answer stops and the panel shows **The agent needs your approval to run a tool:**, with the tool and the values it would use. Nothing runs until you answer:

| Answer | Effect |
| --- | --- |
| **Approve** | Runs once. The next call asks again |
| **Decline** | Does not run. Add a reason, and the assistant adapts to it |
| **Yes, always** | Runs, and saves a preference: this tool stops asking you |

"Yes, always" also covers runs nobody watches, such as scheduled ones, until you revoke it in XTM One, under your profile, **Tool approvals** > **Ask me again**. Use **Approve** for anything you mean to allow once.

Reloading the page keeps the question: the assistant waits for your answer. Features without anyone to ask, such as **AI Insights**, search in plain language and the AI agent steps of playbooks, cannot show it: they stop and report that an approval was needed.
:::

## The agents behind each feature

Each AI feature of OpenCTI is a task XTM One calls an intent. When OpenCTI registers with XTM One, it declares these intents and its business vertical, `cti`, and an XTM One administrator binds one or more agents to each. The agent with the highest priority answers by default. XTM One comes with a built-in agent for each:

| Intent | What it does in OpenCTI | Built-in agent |
| --- | --- | --- |
| `global.assistant` | **Ask Ariane** | **CTEM Assistant**, which hands the conversation to the **OpenCTI Assistant** for a question about OpenCTI |
| `global.fix_spelling`, `global.make_it_shorter`, `global.make_it_longer`, `global.change_tone`, `global.summarize`, `global.explain` | **Ask AI** next to text fields | **OpenCTI Assistant** |
| `cti.container_summary` | **AI Insights**: **Container summary** and **Containers digest** | **OpenCTI Entity Insights** |
| `cti.entity_activity`, `cti.entity_forecast`, `cti.entity_history` | **AI Insights**: **Activity**, **Forecast**, **Internal history** | **OpenCTI Entity Insights** |
| `cti.nlq_search` | **Ask AI** in the search bar | **OpenCTI NLQ** |
| `cti.stix_harvester` | Import connectors that hand a file to an agent | **CTI STIX Harvester (Filigran AI)**, then **CTI STIX Harvester** |
| `cti.stix_transformer` | The playbook step **Transform with AI agent** | **CTI STIX Transformer** |
| `cti.stix_consumer` | The playbook step **Send to AI agent** | **CTI STIX Consumer** |

:::only{opencti=">=7.261030"}
`cti.container_report` writes the text of **Generate an export** with **Ask AI (multiple formats supported)**. Its built-in agent is **OpenCTI Entity Insights**.
:::

The agent of a playbook step or of an import must answer in the shape OpenCTI expects. **Transform with AI agent** goes on with the STIX 2.1 bundle the agent returns; when the answer is no valid bundle, the step leaves by its unmodified output with the bundle it received. **Send to AI agent** ends its branch of the playbook: its agent acts on the bundle, such as posting a summary, and returns nothing to OpenCTI. An agent you build for `cti.stix_harvester` or `cti.stix_transformer` must therefore return a valid STIX 2.1 bundle.

Where several agents serve the same task, you choose: **Select agent** in **AI Insights** and in the step-by-step import, and the **AI agent** of a playbook step.

:::only{opencti=">=7.261030"}
In the search bar, **Choose AI agent** is the arrow next to **Ask AI**.
:::

When no agent is bound to a task, the feature says **No agent available for this action. Ask your administrator to configure XTM One.** An AI agent builder creates the agent in XTM One; an administrator binds it in :menu[Orchestration > Intents]. XTM One offers OpenCTI an agent only when all of this holds:

- The agent is enabled.
- The agent is company-managed or shared with at least one group. A private agent never reaches OpenCTI, even bound to an intent.
- Its binding applies to **All verticals** or to `cti`.

Within those agents, each person still works with their own OpenCTI rights.

## What an agent can reach, and with which rights

An agent never holds an OpenCTI token of its own. Each request carries your identity:

:::steps
1. When you use an AI feature, OpenCTI signs a short-lived token for you, valid 10 minutes at most, that names your email address. It sends your request to XTM One with that token.
2. XTM One runs the agent. When the agent needs knowledge, XTM One calls the OpenCTI API with a token it signs for the same email.
3. OpenCTI checks the signature against the keys XTM One publishes, finds the account with that email, and applies its capabilities, markings and organizations.
:::

The consequences:

- **An agent sees what you see.** Knowledge under a marking you lack, or shared with another organization, stays out of its answers.
- **An agent changes only what you could change**, and the history records the change under your name.
- **Your email must be the same in both modules.** An account with no match in OpenCTI gets no knowledge, and every call fails.
- **What leaves OpenCTI** is your question, the address of the page, the draft you are in, the files you attach, and what the agent reads through the API. XTM One sends them to the language models its administrator configured.

A playbook step has nobody in front of it, so it acts as the account in its **Run as** setting, a service account or the person who builds the playbook. Left empty, it acts as the platform's initial administrator: set it. See [Automate with playbooks](/opencti/use/automate/playbooks/).

:::warning
**AI Insights** keeps an agent's answer for 24 hours by default, for the entity and the agent, not for the person. Within that time, everyone who opens the tab reads the answer written for the first person, whose access may be wider than theirs. Administrators set the duration with `ai:agents_refresh_timeout` (`AI__AGENTS_REFRESH_TIMEOUT`, in minutes, `0` turns it off): see the [configuration reference](/reference/opencti/configuration/#ai-service).
:::

## Use OpenCTI from your own AI client

XTM One also exposes OpenCTI to any client of the Model Context Protocol (MCP), such as Cursor, Claude Desktop or an agent your team builds. The client searches, reads and creates knowledge in OpenCTI, as you. It works with the Community Edition of XTM One, and nothing is installed in OpenCTI: the server exists as soon as the platform is registered.

On your OpenCTI profile, the **XTM One MCP server** card gives the **MCP endpoint URL**, the address of XTM One followed by `/mcp/opencti`. The client authenticates with a personal API key, created in XTM One under your profile, **API Keys**. **Manage in XTM One** opens the **MCP Endpoint** tab of your XTM One profile: the connection status and a client configuration to copy.

::screenshot{id=opencti-xtm-one-mcp-card alt="The XTM One MCP server card of the OpenCTI profile, with the MCP endpoint URL ending in /mcp/opencti, its copy button and Manage in XTM One."}

The server offers knowledge tools only:

| Tools | Examples |
| --- | --- |
| Search and read | Keyword search, entities, relationships, observables, the content of a container, statistics, history |
| Create | Entities, observables, indicators, relationships, reports, cases, groupings, notes |
| Change | Fields, labels, markings, the content of containers, promote an observable to an indicator, delete |

It has no administration tool: no users, groups, roles, playbooks, dashboards, connectors or ingestion. Changes go straight to the live knowledge, not to a draft.

:::example[Query Velvet Harbor indicators from Cursor]
An AI agent builder at Example CERT creates a personal API key in XTM One, keeps it in the environment variable `XTM_ONE_API_KEY`, and adds OpenCTI to `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "opencti": {
      "url": "https://xtm-one.example.com/mcp/opencti",
      "headers": {
        "Authorization": "Bearer ${env:XTM_ONE_API_KEY}"
      }
    }
  }
}
```

Asked "List the indicators of Velvet Harbor created this month", Cursor calls the OpenCTI tools with the builder's own rights. A `401` means the key is wrong, a `403` that an administrator turned the platform MCP servers off, a `404` that no OpenCTI is registered with XTM One.
:::

## Work with OpenCTI from XTM One

The link works the other way too: once OpenCTI is registered, people start from XTM One and its agents read and write OpenCTI as them.

- **Chat with an agent that knows OpenCTI.** The **OpenCTI Assistant**, the **FIMI Analyst** and the **CTEM Assistant** query the knowledge while they answer, and turn it into a report or a slide deck on request, such as "a threat brief on Velvet Harbor".
- **Build agents on OpenCTI.** The OpenCTI integration of XTM One gives an agent tools to read and write entities, observables, relationships, indicators and containers. See [Connect agents to your tools with integrations](/xtm-one/build/integrations/).
- **Let agents work on a schedule.** An assignment runs an agent at set times, such as a weekly brief on an intrusion set for the CTI team, and its runs show in the [work history](/xtm-one/use/work-history/) of XTM One.

## Keep indicators that are still malicious

OpenCTI's decay lowers the score of an indicator over time, and revokes the indicator at the revoke score of its decay rule: see [Age indicators with decay rules](/opencti/configure/decay/). Some indicators reach it while their observable is still malicious. Two built-in agents of XTM One check every indicator about to be revoked:

- **Indicator Decay Guardian** lists the indicators that are not revoked and that decay revokes within 48 hours, enriches their observables with the threat intelligence integrations it is given, and applies the verdicts. It never decides on its own.
- **Indicator Decay Judge** decides, from the enrichment evidence alone, whether each observable is still malicious, with a confidence from 0 to 100. It runs on the routing model of XTM One and has no tools.

| Verdict of the judge | What changes in OpenCTI |
| --- | --- |
| Still malicious, with a confidence of 70 or more | The score rises to the higher of the current score and the one the judge proposes, between 50 and 100, and OpenCTI restarts the decay from it. The indicator gets a note with the evidence and the verdict, and the label `decay-reenriched` |
| Not confirmed | Nothing: the decay goes on |
| Not enough evidence, still malicious under 70, or an enrichment that failed | Nothing. The indicator is reported as degraded, never as kept, and tried again about 2 hours later |

An XTM One administrator sets it up:

::screenshot{id=xtm-one-in-opencti-decay-guardian alt="The Overview of the Indicator Decay Guardian with its two assignments: Re-check indicators about to decay, Active and run by hand, and Hourly indicator decay re-check, Paused until an administrator turns it on."}

:::steps
1. Set the **Routing Model** in **Administration** > **Settings** > **AI and models** > **Models**. Without one, the judge runs on the default model.
2. Open the agent **Indicator Decay Guardian**, its **Configuration** tab, and bind the threat intelligence integrations it queries, such as VirusTotal, GreyNoise, AbuseIPDB, Shodan or urlscan.io. It enriches with these integrations, not with the enrichment connectors of OpenCTI.
3. Run the assignment **Re-check indicators about to decay**, listed on the **Overview** tab of the agent. It runs as you, with your OpenCTI rights, and changes the score of shared indicators.
4. To run it every hour, open the assignment **Hourly indicator decay re-check**, choose the account it acts as in **Run As**, then turn it on. It is off on a new installation, and stays as you leave it.
:::

Each run reports the indicators it examined, kept with their new score, left to decay and degraded, with the reason. One run examines the 20 indicators closest to revocation at most, and says when more are waiting; the next run takes them. The Guardian keeps a record of each indicator, with its evidence and its verdict, in its workset, on the **Worksets** tab of :menu[Work > Operations], and does not judge an indicator again within 24 hours.
::::

:::only{opencti="<7.260811"}
## Ask Ariane, the assistant

**Ask Ariane**, in the top bar, opens a chat window about the knowledge of your platform. It shows when your role has the **Access knowledge** capability, the platform has an Enterprise Edition license, and an administrator has accepted the Filigran AI Terms. OpenCTI passes your question to the address set in `xtm:xtm_one_url`, with a token that lets the assistant read OpenCTI as you for one hour.

When the platform is registered with XTM One, OpenCTI creates for every user with an email address an API token named `XTM One`, with no expiry, and sends it to XTM One with the token of the initial administrator. The token shows in the **API access** part of each profile; revoking it cuts XTM One's access as that user.
:::

## Govern AI on the platform

This part is for administrators: who may use AI, under which terms, and with which agents.

**Enterprise Edition.** **Ask Ariane**, **AI Insights** and **Ask AI** need an Enterprise Edition license; without one, **Ask Ariane** shows an Enterprise Edition badge.

::::only{opencti=">=7.261030"}
The license can be OpenCTI's own or come from XTM One: the Enterprise Edition card says which in **License source**. See [XTM Hub and the Filigran Experience page](/opencti/configure/xtm-hub/#the-rest-of-the-filigran-experience-page).
::::

**The Filigran AI Terms.** **Ask Ariane** stays off until someone with the **Manage parameters** capability accepts them, in **Settings** > **Filigran Experience** or at their first click on **Ask Ariane**. Until then, other users read that **Ask Ariane** isn't activated yet. Declining, or turning the switch off later, hides **Ask Ariane**.

::::only{opencti=">=7.260811"}
When the platform is registered with XTM One, that switch reads **XTM One (Agentic AI)** and governs every AI feature of OpenCTI, not only the assistant: until the terms are accepted, **AI Insights** and **Ask AI** ask for them too. The **Generative AI (AI Insight, NLQ)** switch goes away.

::screenshot{id=opencti-experience-enterprise-edition-card alt="The Enterprise Edition card of the Filigran Experience page: the license details, then the XTM One AI row with its Validate the Filigran AI Terms button."}
::::

**Who sees the assistant.** **Ask Ariane** needs the **Access knowledge** capability. Every agent then works within the rights of the person who asks.

::::only{opencti=">=7.260811"}
**In XTM One**, its administrators decide what agents do in OpenCTI:

| In XTM One | What it governs |
| --- | --- |
| **Orchestration** > **Intents** | Which agents answer each task of OpenCTI, and in which order. A binding applies to **All verticals** or to one, such as `cti` |
| **Administration** > **Settings** > **Access and security** > **Tool approvals** | Whether tools ask before they run. Under **Approval for connected platforms**, a switch per platform makes every change to OpenCTI ask for approval, while its read tools run without asking |
| **Administration** > **Settings** > **Usage and quotas** > **Quotas** | How many agent runs each user, group and the platform may spend |
| **Administration** > **Settings** > **Integrations** > **MCP Endpoint** | **Platform MCP Servers**: off, the MCP endpoint of OpenCTI refuses every client |
| **Administration** > **Settings** > **Platform** | **Artificial Intelligence**: whether XTM One runs AI at all |
| **Administration** > **Observability**: **Model usage** and **Traces** | What agents did and what it cost |

XTM One runs agents only with an Enterprise Edition: a Filigran license installed in XTM One, or a registered platform that has one. OpenCTI sends its license at each registration.
::::

## With or without XTM One

OpenCTI has AI features of its own, which call a language model set in the platform configuration: Mistral AI, OpenAI or Azure OpenAI, hosted by Filigran or by your organization. Registering the platform with XTM One moves them to XTM One agents and adds what only agents do.

::::only{opencti=">=7.260811"}
| Feature | Without XTM One | With XTM One |
| --- | --- | --- |
| **Ask AI** on text fields, **AI Insights**, search in plain language | The platform's language model | The agent bound to each task; you can pick it |
| **Ask Ariane** | A chat window relayed to the address in `xtm:xtm_one_url`, by default the assistant Filigran hosts | The XTM One panel: agents, files, history, approvals |
| Extraction from documents | The Import Document AI web service | An agent, chosen in the import |
| AI agent steps in playbooks | Skipped | Available |
| MCP server, **CTEM Command Center** | Not available | Available |

:::only{opencti=">=7.261030"}
**Generate an export** with **Ask AI (multiple formats supported)** also moves to the bound agent, in HTML only.
:::

:::only{opencti="<7.261030"}
**Generate an export** with **Ask AI (multiple formats supported)** keeps the platform's language model.
:::
::::

:::only{opencti="<7.260811"}
In this version, **Ask AI**, **AI Insights**, search in plain language and **Generate an export** always use the platform's language model. **Ask Ariane** relays to the address in `xtm:xtm_one_url`: by default `https://common.xtm1.filigran.io`, the assistant Filigran hosts, or your own XTM One.
:::

Without XTM One, an administrator connects the platform's language model with the `ai` keys and turns on **Generative AI (AI Insight, NLQ)** in **Settings** > **Filigran Experience**. See [Connect OpenCTI to your own language model](/deploy/configure/ai-extraction/#connect-opencti-to-your-own-language-model).

To register OpenCTI with XTM One, a platform engineer sets `xtm:xtm_one_url` (`XTM__XTM_ONE_URL`) and `xtm:xtm_one_token` (`XTM__XTM_ONE_TOKEN`), the registration token XTM One shares with its platforms. OpenCTI then registers 30 seconds after it starts, and again every 5 minutes. See [Connect the modules to each other](/deploy/configure/connect-modules/#register-the-platforms-with-xtm-one) and the [XTM Suite settings](/reference/opencti/configuration/#xtm-suite).

## What's next

- [Read the home dashboard and AI summaries](/opencti/use/explore/insights/), to brief on a threat with **AI Insights**.
- [Refine content with AI](/opencti/use/enrich/refine-content/), to rewrite texts and draft reports.
- [Automate with playbooks](/opencti/use/automate/playbooks/), to put AI agents in your automations.
- [Connect the modules to each other](/deploy/configure/connect-modules/), to register OpenCTI with XTM One.
- [XTM One](/xtm-one/), to build and govern the agents.
