---
title: Set up the platform in Settings
description: What each of the seven tabs of the XTM One Settings is for, in the order a first setup goes through them, and where each one is explained in detail.
doc_type: how-to
products:
  - xtm-one
personas: [admin, platform-engineer]
owner: docs-xtm-one
last_reviewed: 2026-11-08
migrated_from:
  - xtm-one:/administration/settings/
---

:menu[Administration > Settings] holds what applies to the whole platform: the profile of your organization that agents read, the fleets and agent runs, the integrations and channels, the AI models and the shared assistant, access and security, usage and quotas, and the platform's identity and license. Only administrators open it. This page says what each of its seven tabs is for, which ones a first setup goes through, and links the pages that explain each one in detail. Read [How access works in XTM One](/xtm-one/concepts/access/) first if you are new to the platform.

::diagram{id=xtm-one-administer-settings-tabs}

## Before you start

- Your account is an administrator. A user who is not sees no **Settings**, and every address under `/settings` takes them back home.
- While AI is switched off, **AI and models**, the **Agent runs** and **CTEM stage agents** sections of **Agents and automations**, and the **Observability** section of **Usage and quotas** are read-only. On Community Edition, prepare providers, users and groups now: nothing you configure is lost when the Enterprise Edition is activated. See [What each edition of XTM One brings](/xtm-one/administer/editions/).
- A value set by an environment variable shows an **Env** badge and cannot be changed here: change the variable and restart. The variables are in the [XTM One configuration reference](/reference/xtm-one/configuration/).

Each tab and each of its sections has its own address, such as `/settings/usage/observability`. The addresses of the former tabs (`/settings/parameters`, `/settings/models`, `/settings/channels`, `/settings/quotas`...) take you to their new place, so older bookmarks still work.

## Set up the platform

:::steps

1. In **Platform**, check the **Instance ID**, then install the license with **Install License**, or connect an Enterprise Edition OpenCTI, OpenAEV or OpenCRQ. Choose the **Default language** under **Platform Language**, then click **Edit** on **Platform Branding** to set the platform's name, logo and colors.
2. In **AI and models** > **Models**, click **Add Provider**, validate it, and choose the platform default model. Choose the embedding model before anyone fills a knowledge base: changing it later indexes every knowledge base again.
3. In **Access and security**, open **Authentication** and connect your identity provider. See [Configure single sign-on](/xtm-one/administer/sso/).
4. Still in **Access and security**, open **Security audit** and select **Re-run**. Fix each `Critical` and `Warning` finding, or dismiss the ones you accept on purpose.
5. In **Usage and quotas** > **Quotas**, set the agentic limit per user, then the exceptions per group or per user.
6. In **Integrations** > **Channels & Bots**, connect the messaging apps your teams use. See [Talk to agents from your messaging apps](/xtm-one/administer/channels/).
7. In **Usage and quotas** > **Observability**, choose how long traces and audit log entries are kept, and whether traces keep the prompts and answers.
8. In **Organization** > **Profile**, fill in the profile of your organization that agents read: sector, regions, crown jewels, tech stack, assets, risk appetite. See [Set up the organization profile agents read](/xtm-one/administer/organization-profile/).

:::

::screenshot{id=xtm-one-administer-settings-parameters alt="The Platform tab of Settings: the Instance ID a license is generated for, the XTM Hub card Not registered with its Register on the XTM Hub button, and the Enterprise License card. The license is read from the ENTERPRISE_LICENSE environment variable, which the card warns takes priority over a license stored in the database."}

Every change made in Settings is recorded in :menu[Administration > Audit logs].

## The tabs

### Organization

What agents know about your organization, and what the Value page counts their work with:

- **Profile**: the **Organization profile** every agent and fleet reads before it ranks, scopes or estimates. It is shared context: every user and every agent of the platform sees it. It holds **Identity** (size, regions of operation), **Risk appetite and costs**, **Ticketing and intelligence** (the ticketing integration and project, the OpenCTI PIRs agents follow), **Crown jewels and business services**, **Tech stack**, **AI systems inventory**, **Assets and exposure**, **Asset groups**, the **Prioritization policy** of the CTEM funnel and **Enterprise knowledge**. Agents cite the fields they used and name the ones not filled, never guess them; each save is a new version. Outside edit mode, the lists read as tables you search, filter and export to CSV; each list card has its own **Edit**. **Draft** fills the profile **From the platforms**, **With AI** or **From a connected integration**, and the **Syncs** card runs the platform imports again on a schedule. **Organization context in agents** chooses whether every agent starts with a summary of the profile. See [Set up the organization profile agents read](/xtm-one/administer/organization-profile/). While fields are not filled, **Continue onboarding in chat** opens a new chat with your default assistant, which asks for the missing fields one at a time, proposes a value when it can infer one, and saves each answer you confirm as a new version. You can skip a field for now, and a chat started later resumes at the next field. The same interview starts after an AI draft saved outside edit mode. It requires the Enterprise Edition and AI switched on. **Tracked organizations**, at the bottom, lists your providers, customers, partners and subsidiaries: see [Track the organizations your company depends on](/xtm-one/administer/tracked-organizations/). The first administrator to sign in while the profile was never saved gets the **Set up your organization profile** dialog: **Set up now** opens this profile in edit mode, with the **Draft** menu to start **From the platforms** or **With AI**, and nothing is saved before **Save**. The dialog shows once for the whole platform, whatever the answer.
- **Departments**: the org units usage is rolled up by in **Model usage**, and who belongs to which, from group rules, the department your identity provider sends at sign-in, or an import.
- **Units of work** and **Rates by role**: the minutes each kind of work takes by hand and the hourly rate of each role, which the Value page computes hours saved and value with. See [How XTM One computes the value of agents' work](/xtm-one/concepts/value-methodology/).
- **Value visibility and privacy**, under **Rates by role**, chooses who sees the value figures:
  - **Organization totals for every user**, on by default: every user may read the organization's figures. Off, only administrators see them, and every user still sees their own.
  - **Smallest group shown apart**: 5 people by default, from 2 to 50. For users who are not administrators, a fleet, agent, unit of work or week backed by fewer people is folded into "Other". Exports and agent tools apply the same rule.
  - **By person breakdown for administrators**, off by default, adds a breakdown of the value by person. Every read of it is written to the audit log.

### Agents and automations

- **Fleets** holds the **Member presets** that **Apply a preset** offers in every fleet's **Members** view. A preset writes, on the fleet's defaults or on the members you select, any of **Conversation shared**, **Organizations** and **Collaboration instruction**. Three built-ins come with the platform, such as **Own organization, a summary**; **New preset** adds your company's own. A preset writes its values once: editing or deleting it later changes no fleet. The fleets themselves are listed on the Fleets page: see [Run agents as a team with fleets](/xtm-one/build/fleets/).
- **CTEM stage agents**, on XTM One only: the platform's default analyst for the dashboard briefing and for each of the five CTEM stages, per vertical. Each user can still pick their own in their profile.
- **Agent runs** bounds the `wait` tool for every run of chat, assignments and evaluations: **Longest single wait** and **Total wait per run**, in seconds. An agent or an assignment may lower them, never raise them. **Waits on named work are not counted** leaves out of the total the waits that name the tasks or runs they wait for.
- **Quality**: the feedback criteria people score when they rate an agent's work, beyond the thumbs up or down. A criterion is a **Scale** (1 to 5, for example), **Categorical** (options, each worth a score from 0 to 1) or **Numeric**, with a weight. Its kind cannot change once created. Deleting a criterion deletes the scores given on it; switch it off to keep them.

### Integrations

- **Catalog** chooses which integration types users are offered.
- **OAuth Defaults** holds the OAuth applications your company registered with a provider, so that users connect an integration without creating their own application.
- **Channels & Bots**: the bots of Slack, Microsoft Teams, WhatsApp, Telegram, Discord and Google Chat, which agent answers on which channel, who may write to them, and whether people without an account may. See [Talk to agents from your messaging apps](/xtm-one/administer/channels/).
- **MCP Endpoint**: how external MCP clients reach XTM One:
  - **The platform MCP endpoint**: its URL, the global MCP key (a call made with it has no account and counts against the platform-wide cap), and the agents exposed there.
  - **Platform MCP Servers**: the OpenCTI, OpenAEV and OpenCRQ MCP servers, on by default, active once a platform is registered. See [Use XTM One's own MCP servers from other clients](/xtm-one/build/platform-mcp-servers/).
  - **Platform Tools MCP Server**: XTM One's own tools at `/mcp/xtm` (traces, agents, assignments, knowledge bases...), **off by default** because it exposes the platform's configuration. Each caller only reaches what they can see and manage.

### AI and models

**Models** holds the model providers and the models each capability runs on: chat and reasoning, embeddings, and the decision model agents use to act on the page of an embedded chat. A provider's **Models** list holds the models that chat; the other capabilities are chosen in their own section, with **Configure**.

Three choices on a provider card matter for governance:

- **Access** restricts a provider to some groups and people, for example to bill each team's runs to its own API key. The card shows **Restricted**. A run only considers the providers its user may use; keep one provider unrestricted, since callers without an account and platform maintenance only use those.
- **Never use for people and groups**, in **Access**, keeps some people off a provider whatever else grants it, for example where a regulation forbids a vendor for one jurisdiction. The card shows **Exclusions**. Embeddings and image generation are platform-wide: an excluded person's knowledge base searches and image requests are refused rather than sent to that provider.
- **Delete** first shows what moves to another provider: the default models, the agents and users that picked a model on it. The embedding model is never switched automatically: it is cleared, because content indexed with one model cannot be found with another.

With an SMTP server (`SMTP_HOSTNAME`), changing the default model or the model list offers to email the users about it. Set `MODEL_ANNOUNCEMENT_EMAILS_DISABLED` to `true` to turn these emails off: XTM One then says that the users cannot be emailed. See the [XTM One configuration reference](/reference/xtm-one/configuration/).

See [Protect your data and approve what agents do](/xtm-one/administer/ai-governance/) for what is sent to a provider.

**Media** chooses the models of image generation and speech.

**General Assistant** is the assistant every user can talk to without picking an agent: its prompt, its model, the tools and resources it may use (**Built-in Tools**), the agents it may hand work to (**Agent Handover**), its **Skills** and its behavior on the channels (**Channels & Routing**).

- **Max Iterations** is the number of tool-calling rounds an agent takes before it is asked to wrap up: 300 by default, up to 1000. An agent's own value wins.
- **Link to Agent** makes an enabled company-managed agent answer in the assistant's place, with its own model, tools and persona. Only an agent every user can chat with can be linked. On XTM One, the assistant is linked to the **CTEM Assistant**.

**Workspace knowledge** sets how much of a workspace's sources agents receive whole (15 % of the context window by default), and after how many quiet minutes a conversation becomes searchable (30 by default). It shows with the Enterprise Edition only.

### Access and security

Eight sections, each with its own address under `/settings/security/`:

| Section | What it is for |
| --- | --- |
| **Security audit** | Checks the configuration the platform runs with and reports findings as `Critical`, `Warning` or `Info`, with how to fix them. **Findings** and **Controls** are its two views; **Dismiss finding** accepts a finding you chose on purpose. It runs every hour, and a new `Critical` or `Warning` finding notifies every administrator once. |
| **Security options** | The platform-wide security switches: session lifetime and idle timeout, password length, link lifetimes, webhook secrets, the prompt guard, the code interpreter and custom tools. Each is also an environment variable, which wins. See [Security switches](/reference/xtm-one/configuration/#security-switches). |
| **Tool approvals** | Whether agents ask a person before running tools, and which tools ask. See [Protect your data and approve what agents do](/xtm-one/administer/ai-governance/#approve-what-agents-do). |
| **Authentication** | Local accounts, OpenID Connect, SAML and allowed email domains. See [Configure single sign-on](/xtm-one/administer/sso/). |
| **Secret vaults** | The vaults secret fields read from. See [Keep secrets in your vault](/xtm-one/administer/secret-vaults/). |
| **API keys** | Every personal API key of the platform, whoever owns it, with **Revoke**. The key itself is never shown. |
| **Emergency stop** | The token that stops every agent from outside a session. See [Stop every agent at once](/xtm-one/administer/emergency-stop/). |
| **Memory purge** | Deletes every agent memory of the platform, for good. |

### Usage and quotas

**Quotas** holds the agentic limits: how many agentic executions (a chat turn, an assignment run, a background task) people may run per day, month and year. Requires the Enterprise Edition.

- **The platform-wide cap** comes from your license, or from the included allowance of a connected Enterprise Edition platform: 30 a day, 100 a month and 300 a year.
- **A limit per user, per group or on a user** is what one person may spend. `0` stops them. An empty field inherits: the user's own limit, then their groups', then the default per user. **Unlimited** lifts the limit.
- **A person in several groups** gets the most permissive group limit.
- **Requests with no account** (a public chat widget, an anonymous channel user, the platform MCP key) count against the platform-wide cap only.

**Agent Failure Pool** credits back each agent's first failed runs, and every run that never reached a model. **Enter a support code** takes a code from Filigran support that recharges the pool or resets the platform-wide counters. **Reset usage**, on a user's **Quotas** tab, resets that user's own counters.

**Observability** decides what is kept of the agents' activity:

- **Enable local trace storage**: whether the language model calls are traced at all.
- **Store full payloads**: whether a trace keeps the prompts, answers, tool arguments and tool results, or only timings, token counts, cost, model, tool names and status.
- **Trace retention (days)**: 90 by default, from 1 to 365.
- **Audit log retention (days)**: how long the audit log entries are kept, 90 by default, from 30 to 3650. Older entries are deleted every hour; each user's last sign-in is kept past the window.
- **Langfuse export**: an optional export of the traces, which the two switches above do not govern.

A change to the two switches applies within seconds, to the traces written from then on. See [Follow what agents do and what they cost](/xtm-one/administer/observability/).

**Model pricing** holds the price of each model, which **Model usage** computes costs with.

### Platform

The platform's identity and license: the **Instance ID** that a license is generated for, the registration on the XTM Hub, the license itself (**Install License**, or `ENTERPRISE_LICENSE`, which then wins), the **Default language** of the interface for the users who have not chosen one (**Platform Language**), the **Default Presentation Template** applied to the decks agents generate, the **Brand asset source**, and **Platform Branding**: **Identity** (platform name, login message), **In-App Colors**, **Browser & PWA** and **Media Assets** (logo and favicon).

**Brand asset source** reads your company's brand packs from a GitHub folder. See [Brand the decks and reports agents produce](/xtm-one/build/templates/).

- **GitHub folder**: a folder address, such as `https://github.com/owner/repository/tree/main/brand-assets`. Each sub-folder that holds a `manifest.json` is a pack. An address without `/tree/` reads the root of the default branch.
- **GitHub integration**: a connected GitHub integration that may read the repository's contents. Connect one in **Integrations** first.

The packs are read when you save, and at every restart. **Sync now** reads them again, and **Clear** removes the source. A pack the platform also ships keeps the platform's own files.

**Artificial Intelligence** is the switch that turns AI on and off for the whole platform. See [Run XTM One with AI switched off](/xtm-one/administer/running-without-ai/).

:::example[Prepare the platform of Grand Meridian Hotels]
Hélène Marchetti sets up `https://xtm-one.grand-meridian.example`, which Marc Dupuis deployed with an XTM One license in its environment.

- **Platform**: the license shows as set by `ENTERPRISE_LICENSE`. She names the platform "Grand Meridian AI" and uploads the company logo and favicon.
- **AI and models** > **Models**: she adds an Azure OpenAI provider for chat, and an Ollama provider in the Paris datacenter, which she chooses as the embedding model so that the documents of "Grand Meridian security policies" never leave the network.
- **Access and security** > **Authentication**: Microsoft Entra ID through OpenID Connect, with **On-the-fly user creation** on and `grand-meridian.example` as the only allowed domain.
- **Access and security** > **Security audit**: **Re-run** reports sessions that last 7 days; she sets **Absolute session lifetime (minutes)** to `1440` in **Security options**, so that everyone signs in again every day.
- **Usage and quotas** > **Quotas**: 50 agentic executions a day per user, and **Unlimited** on the group **SOC**, whose night shift runs "Night shift triage" all night.
- **Usage and quotas** > **Observability**: **Trace retention (days)** at `30`, **Store full payloads** off, **Audit log retention (days)** at `365`.
- **Organization** > **Profile**: hospitality, Europe, the payment platform and the booking engine as crown jewels. She imports the hotels' assets from OpenAEV with **Draft** > **From the platforms**, kept in sync every day.

The next morning, the audit shows no `Critical` finding, and Julien Bernard sees the SOC's usage in **Model usage**.
:::

## What's next

- [Configure single sign-on](/xtm-one/administer/sso/)
- [Protect your data and approve what agents do](/xtm-one/administer/ai-governance/)
- [Keep secrets in your vault](/xtm-one/administer/secret-vaults/)
- [XTM One configuration reference](/reference/xtm-one/configuration/)
