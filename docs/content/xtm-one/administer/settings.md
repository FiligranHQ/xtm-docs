---
title: Set up the platform in Settings
description: What each of the seven tabs of the XTM One Settings is for, in the order a first setup goes through them, and where each one is explained in detail.
doc_type: how-to
products:
  - xtm-one
personas: [admin, platform-engineer]
owner: docs-xtm-one
last_reviewed: 2026-11-19
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

1. In **Platform**, check the **Instance ID**, then install the license with **Install License**, or connect an Enterprise Edition OpenCTI or OpenAEV. Choose the **Default language** under **Platform Language**, then click **Edit** on **Platform Branding** to set the platform's name, logo and colors.
2. In **AI and models** > **Models**, click **Add Provider**, validate it, and choose the platform default model. Choose the embedding model before anyone fills a knowledge base: changing the model or its dimensions later indexes every knowledge base again.
3. In **Access and security**, open **Authentication** and connect your identity provider. See [Configure single sign-on](/xtm-one/administer/sso/). To let it create and deactivate the accounts too, see [Provision accounts from Microsoft Entra](#provision-accounts-from-microsoft-entra).
4. Still in **Access and security**, open **Security audit** and select **Re-run**. Fix each `Critical` and `Warning` finding, or dismiss the ones you accept on purpose.
5. In **Usage and quotas** > **Quotas**, set the agentic limit per user, then the exceptions per group or per user.
6. In **Integrations** > **Channels & Bots**, connect the messaging apps your teams use. See [Talk to agents from your messaging apps](/xtm-one/administer/channels/).
7. In **Usage and quotas** > **Observability**, choose how long traces and audit log entries are kept, and whether traces keep the prompts and answers. In **Storage & retention**, choose how long the files people upload and agents create are kept.
8. In **Organization** > **Profile**, fill in the profile of your organization that agents read: sector, regions, crown jewels, tech stack, assets, risk appetite. See [Set up the organization profile agents read](/xtm-one/administer/organization-profile/).

:::

::screenshot{id=xtm-one-administer-settings-parameters alt="The Platform tab of Settings: the Instance ID a license is generated for, the XTM Hub card Not registered with its Register on the XTM Hub button, and the Enterprise License card. The license is read from the ENTERPRISE_LICENSE environment variable, which the card warns takes priority over a license stored in the database."}

Every change made in Settings is recorded in :menu[Administration > Audit logs].

## Provision accounts from Microsoft Entra

With SCIM provisioning, Microsoft Entra ID manages the XTM One accounts for you: it creates the account of each person you assign to the application, keeps their name, title and department current, adds them to groups, and deactivates the account when they leave. People still sign in through single sign-on: an account SCIM creates has no password, and it is a user account, never an administrator.

Before you start:

- Your account is an administrator. Provisioning needs no Enterprise Edition.
- [Single sign-on](/xtm-one/administer/sso/) works with Microsoft Entra ID, through OpenID Connect or SAML.
- In Microsoft Entra, you may create an enterprise application and configure its provisioning (Application Administrator or Cloud Application Administrator).
- Microsoft Entra reaches XTM One from the internet: its provisioning service calls the platform from Microsoft's cloud.

:::steps

1. Open :menu[Administration > Settings > Access and security > Provisioning] (`/settings/security/provisioning`). On the **SCIM provisioning** card, copy the **Endpoint URL** with **Copy**: the address you open XTM One with, followed by `/api/v1/scim/v2`.
2. In **Token name**, name the identity provider, such as `Microsoft Entra production`, and select **Generate token**. Use one token per identity provider, so that you can revoke one without stopping the others.
3. Copy the token at once: it shows only this time, and XTM One keeps only a hash of it.
4. In the Microsoft Entra admin center, open **Enterprise applications** > **New application** > **Create your own application**, name it `XTM One`, and choose the application you don't find in the gallery.
5. In the application, open **Provisioning** > **Get started** and set **Provisioning Mode** to **Automatic**.
6. Under **Admin Credentials**, paste the endpoint URL in **Tenant URL** and the token in **Secret Token**. Select **Test Connection**, then **Save**.
7. Under **Users and groups**, assign the people or the groups who should have an XTM One account.
8. Under **Provisioning**, set **Provisioning Status** to **On** and save.

:::

::screenshot{id=xtm-one-administer-settings-provisioning alt="The SCIM provisioning card of Settings > Access and security > Provisioning: the Endpoint URL ending with /api/v1/scim/v2 and its Copy button, the token Microsoft Entra production with its prefix, Never used and Revoke, and the Token name field with Generate token."}

Microsoft Entra runs a first cycle, then one about every 40 minutes; **Provision on demand**, in the application, tests one person first. The token's line then shows when it was last used, and **Latest SCIM changes** lists the last 50 accounts and groups the identity provider created, changed or deactivated. Each line reads in the language of the interface, such as "Account Camille Roux provisioned" or "Group SOC changed", with its date. A call XTM One refused is not listed there: Entra's provisioning logs show it (`401` for a wrong or revoked token, `409` for an email already linked to another Entra user).

What Microsoft Entra sends:

| Entra attribute | XTM One field |
| --- | --- |
| `userPrincipalName` (or `mail`) | The email, which the person signs in with |
| `objectId` | The link to the Entra user |
| `active` (the default mapping) | Whether the account is active or deactivated |
| `displayName`, `jobTitle`, `department` | The full name, the title and the department |

The other attributes (phone numbers, addresses, manager) are accepted and ignored. When Entra creates a person whose email already has an account, created by single sign-on or by an administrator, XTM One links that account instead of creating a second one, and the **Users** list then shows the account comes from SCIM. To provision groups, turn on the group mappings in Entra: SCIM then only removes the members it added itself, never one an administrator or the SAML group sync added.

When Entra disables, unassigns or deletes a person, XTM One deactivates the account at once, as **Disable** does on the person's page: sign-in is refused, the personal API keys are revoked, the runs stop, the private agents are disabled, and what the person shared moves to an administrator. Nothing is deleted: a deletion in Entra also shows **Removed by the identity provider**. Reactivating the person in Entra brings their agents and assignments back. See [How access works in XTM One](/xtm-one/concepts/access/).

**Deactivation review**, on the same page, lists each account deactivated this way or any other way, with what the deactivation did to each of its objects: **Disabled**, **Moved**, **Paused**, **Disconnected**, **Kept, no administrator** (no other active administrator could take it) or **Used by another agent**. Check each item, then record what you decided with **Mark as kept**, **Mark as reassigned** or **Mark as deleted**: these buttons only record your review, and the transfer or the deletion itself is yours to make. The account's group memberships are kept for audit.

**Revoke**, beside a token, cuts the identity provider off: it can no longer create, update or deactivate accounts until you generate a new token and paste it in **Secret Token**. Each token makes at most 600 calls a minute; beyond, Entra waits and retries on its own. Every change it makes is recorded in :menu[Administration > Audit logs] under the token's name.

:::example[Let Microsoft Entra ID manage the Grand Meridian accounts]
Grand Meridian Hotels signs its staff in with Microsoft Entra ID through OpenID Connect. Hélène Marchetti opens **Provisioning** on `https://xtm-one.grand-meridian.example`, types `Microsoft Entra production` in **Token name** and selects **Generate token**. In the Entra admin center, she creates the enterprise application "XTM One", pastes `https://xtm-one.grand-meridian.example/api/v1/scim/v2` in **Tenant URL** and the token in **Secret Token**, and **Test Connection** succeeds. She assigns the Entra groups of the SOC and of the CTI team, runs **Provision on demand** for Camille Roux, whose existing account is linked rather than duplicated, and sets **Provisioning Status** to **On**.

When Paul Garnier leaves the SOC, Entra deactivates his account in its next cycle. **Deactivation review** lists his private agent as **Disabled** and the agent he shared with the SOC as **Moved** to the administrator of the platform. Hélène Marchetti gives that agent to Julien Bernard, then selects **Mark as reassigned**.
:::

## The tabs

### Organization

What agents know about your organization, and what the Value page counts their work with.

**Profile** is the **Organization profile** every agent and fleet reads before it ranks, scopes or estimates. It is shared context: every user and every agent of the platform sees it, as its info bubble **About the organization profile** recalls. Agents cite the fields they used and name the ones not filled, never guess them; each save is a new version. It has five tabs, each with its own address:

| Tab | What it holds | Address |
| --- | --- | --- |
| **Identity and risk** | **Identity** (size, regions of operation), **Risk appetite and costs**, **Ticketing and intelligence** (the ticketing integration and project, the OpenCTI PIRs agents follow) | `/settings/organization` |
| **Business** | **Crown jewels and business services**, and **Tracked organizations**: your providers, customers, partners and subsidiaries. See [Track the organizations your company depends on](/xtm-one/administer/tracked-organizations/). | `/settings/organization/profile/business` |
| **Technology** | **Tech stack**, **AI systems inventory** | `/settings/organization/profile/technology` |
| **Exposure** | **Assets and exposure**, **Asset groups**, the **Prioritization policy** of the CTEM funnel | `/settings/organization/profile/exposure` |
| **Knowledge and sources** | **Enterprise knowledge**; **Syncs**, which runs the platform imports again on a schedule; **Organization context in agents**, which chooses whether every agent starts with a summary of the profile | `/settings/organization/profile/knowledge` |

Each tab shows how many of its fields are filled, such as `3/5`, and a mark when a draft filled some of them, when it holds changes not saved yet, or when one of its fields is refused. The fields not filled are listed at the top of their tab, under "Fields not filled, reported by every agent that needs them": each one opens the form on that field. An address followed by `?card=` and a card's key opens the tab on that card: the CTEM pages link there when a stage lacks its data, such as **Add tech stack** or **Map crown jewels to assets**.

**Edit** opens the whole profile: switching tabs keeps what you typed, one **Save** saves every tab, and closing or reloading the browser page before you save asks first. Outside edit mode, the lists read as tables you search, filter and export to CSV; each list card has its own **Edit**. **Draft** fills the profile **From the platforms**, **With AI** or **From a connected integration**; a draft from the platforms lists the fields it filled tab by tab, with their number. See [Set up the organization profile agents read](/xtm-one/administer/organization-profile/).

While fields are not filled, a line above the tabs counts them, with **Continue onboarding in chat**. It opens a new chat with your default assistant, which asks for the missing fields one at a time, proposes a value when it can infer one, and saves each answer you confirm as a new version. You can skip a field for now, and a chat started later resumes at the next field. The same interview starts after an AI draft saved outside edit mode. It requires the Enterprise Edition and AI switched on.

The first administrator to sign in while the profile was never saved gets the **Set up your organization profile** dialog: **Set up now** opens this profile in edit mode, with the **Draft** menu to start **From the platforms** or **With AI**, and nothing is saved before **Save**. The dialog shows once for the whole platform, whatever the answer.

The other sections of the tab:

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
- **Platform Health** (`/settings/automations/health`) sets up the platform's own maintenance agents. They consolidate memories, diagnose agents and flows, try changes on sandbox copies and propose improvements that the managers approve. **Open Platform Health** leads to what it found and proposes, in :menu[Administration > Platform Health]: see [Review what Platform Health finds and proposes for your agents](/xtm-one/administer/platform-health/). **Edit** opens the settings below, and one **Save** keeps them together:
  - **Enable Platform Health** is off by default. While it is off, Platform Health reads nothing and starts no work, and its settings are kept. **Name** is what people call it on its page, in the inbox and on its agent: up to 40 characters, "Platform Health" when empty.
  - **Supervision scope**: **All agents and flows**, the default, or **Selected agents and flows**, picked in **Supervised agents** and **Supervised flows**. Either way, it reads an agent or a flow only once its managers opted it in, and never the memories and conversations of a user who opted out.
  - **Budget**: Platform Health's own caps, on top of the license and the quotas. A cycle that reaches **Tokens per cycle**, 500,000 by default, or **Cost per cycle (USD)**, 5, starts no more work. Past **Tokens per month**, 10,000,000, or **Cost per month (USD)**, 100, it starts nothing until the next month. An empty cap reads **No cap**, and the line below the caps gives what it spent this month.
  - **Schedule**: when each fleet-wide cycle starts, as a **Cron expression** read in the **Timezone** (UTC by default). **Memory consolidation** runs every day at 02:00 (`0 2 * * *`); **Quality review**, **Flow review** and **Weekly digest** run every Monday at 03:00, 04:00 and 07:00. Each one can be switched off.
  - **Signals**: thresholds read without any model call. A crossing starts one investigation on its agent or flow, never a second while one is open there. **Feedback drop**: a score down 10 points over 3 days, with at least 10 ratings. **Failure spike**: above 5 % over the last 20 runs. **Cost drift**: up by 20 % over 7 days, with at least 10 runs. **Model change**: the model of a supervised agent changes.
  - **Durations**: an applied improvement is watched for **Watch an applied proposal**, 7 days, and a regression in that time proposes the rollback. A flow that replaces an assignment first runs beside it without acting, for **Flow shadow** (3 days) and **Shadow pairs needed** (5 runs side by side). Once it replaced the assignment, it is watched for **Watch a switched-over flow**, 7 days.
  - **Dry-run**: **Log every write instead of executing it**, off by default, logs every write of Platform Health's agents (memories, proposals, sandbox copies) without making it. Use it to see what Platform Health would do before letting it act.
  - **Companion pets**, outside **Edit**: the pets users may pick for their companion. **Add a pet** takes a package of up to 8 MB in the Codex pet format: `pet.json` and an 8-column spritesheet of 192x208 cells, in PNG or WebP. Upload it as one `.zip` or as the two files. The pets that come with the platform show **Built-in** and stay. Removing a pet you added gives the users who picked it the default pet.

  Platform Health works only with AI switched on and the Enterprise Edition.

::screenshot{id=xtm-one-administer-settings-health alt="The Platform Health section of Settings > Agents and automations at rest, with Open Platform Health and Edit: Enable Platform Health reads Off and the name Platform Health, Supervise reads All agents and flows, and the Budget block shows its four caps, 500,000 and 10,000,000 tokens, 5 and 100 USD."}

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

When every chat provider is configured but offers no model, the section says so: "The configured LLM providers offer no model". Agents and the features that call a model wait until one does. For a provider set by environment variables, it names the `LLM_PROVIDER_<NAME>_MODELS` variable to set before a restart, or `LLM_PROVIDER_MODELS_EDITABLE=true` to choose its models here; otherwise, select models on a provider of the section. See [AI providers](/reference/xtm-one/configuration/#ai-providers).

Changing the embedding model or its dimensions indexes every knowledge base again, and their documents cannot be searched until they are. Moving the same model, with the same dimensions, to another provider keeps what is indexed: use it when a provider fails, for example from OpenAI to Azure OpenAI with the same `text-embedding-3-small`. The documents that failed to index on the former provider are then tried again at once. Without an embedding model chosen, each provider uses its own default, so changing the provider indexes everything again.

With an SMTP server (`SMTP_HOSTNAME`), changing the default model or the model list offers to email the users about it. Set `MODEL_ANNOUNCEMENT_EMAILS_DISABLED` to `true` to turn these emails off: XTM One then says that the users cannot be emailed. See the [XTM One configuration reference](/reference/xtm-one/configuration/).

See [Protect your data and approve what agents do](/xtm-one/administer/ai-governance/) for what is sent to a provider.

**Media** chooses the models of image generation and speech. With the Filigran AI Gateway as provider, an image model the gateway declares for editing also takes up to three reference pictures and a mask. Agents can then edit a picture, as with OpenAI's own image models, rather than only draw a new one.

**General Assistant** is the assistant every user can talk to without picking an agent: its prompt, its model, the tools and resources it may use (**Built-in Tools**), the agents it may hand work to (**Agent Handover**), its **Skills** and its behavior on the channels (**Channels & Routing**).

- **Max Iterations** is the number of tool-calling rounds an agent takes before it is asked to wrap up: 300 by default, up to 1000. An agent's own value wins.
- **Link to Agent** makes an enabled company-managed agent answer in the assistant's place, with its own model, tools and persona. Only an agent every user can chat with can be linked. On XTM One, the assistant is linked to the **CTEM Assistant**.

**Workspace knowledge** sets how much of a workspace's sources agents receive whole (15 % of the context window by default), and after how many quiet minutes a conversation becomes searchable (30 by default). It shows with the Enterprise Edition only.

### Access and security

Nine sections, each with its own address under `/settings/security/`:

| Section | What it is for |
| --- | --- |
| **Security audit** | Checks the configuration the platform runs with and reports findings as `Critical`, `Warning` or `Info`, with how to fix them. **Findings** and **Controls** are its two views; **Dismiss finding** accepts a finding you chose on purpose. It runs every hour, and a new `Critical` or `Warning` finding notifies every administrator once. |
| **Security options** | The platform-wide security switches: session lifetime and idle timeout, password length, link lifetimes, webhook secrets, the prompt guard, the code interpreter and custom tools. **Trusted certificates for web fetches** takes the PEM certificate of a private certificate authority, or a host's self-signed one, so that `web_fetch`, the knowledge base URL crawl and Deep Investigation reach an internal site. Verification stays on and the host name must still match; paste the certificate only, never its private key. Empty, only the system certificate store is trusted. Each option is also an environment variable, which wins, such as `WEB_FETCH_TRUSTED_CA_CERTIFICATES`. See [Security switches](/reference/xtm-one/configuration/#security-switches). |
| **Tool approvals** | Whether agents ask a person before running tools, and which tools ask. See [Protect your data and approve what agents do](/xtm-one/administer/ai-governance/#approve-what-agents-do). |
| **Authentication** | Local accounts, OpenID Connect, SAML and allowed email domains. See [Configure single sign-on](/xtm-one/administer/sso/). |
| **Provisioning** | **SCIM provisioning**, the endpoint and the tokens an identity provider creates and deactivates accounts with; **Deactivation review**, what each deactivation moved or disabled; **Latest SCIM changes**. See [Provision accounts from Microsoft Entra](#provision-accounts-from-microsoft-entra). |
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

**Storage & retention** (`/settings/usage/storage`) decides how long each category of stored file is kept. Its table shows, per category, how many files and how much space it holds today, and in **Last sweep** how many files the last sweep deleted:

| Category | What it holds | Kept by default | Counted from |
| --- | --- | --- | --- |
| **Agent working files** | Scratch files agents write while they work | 60 days | The last use |
| **Agent deliverables** | Files agents hand to people to download | 60 days | The last use |
| **Other agent files** | Files tools and MCP servers return | **Never** deleted | The file's creation |
| **Chat uploads** | Files people attach in a conversation | **Never** deleted | The upload |
| **Channel files** | Files received from Teams, Outlook and other channels | **Never** deleted | The file's arrival |
| **Check pictures** | Frames and captures a tool makes for the agent to look at | 7 days | The capture |

- **The last use** of a file is a download, a preview or an agent reading it.
- **Edit** takes a number of days per category, from 0 to 3650. `0` keeps the files for good.
- **A category set by its environment variable** shows "Set by" and the variable, such as `FILE_RETENTION_CHAT_DAYS`, and cannot be changed here. See [Performance](/reference/xtm-one/configuration/#performance) in the configuration reference.
- **A sweep runs every hour**, and the line below the table dates the next one and the last one. It never deletes a file younger than an hour, or one that a work item, a workset, a shared state, a presentation, a report or a video still uses.
- **A deleted file** stays in its conversation as a card, "Deleted on {date} by the retention policy": only its content goes, and it can no longer be downloaded. See [Find the files agents created and received](/xtm-one/use/files/).

Shortening a retention, or giving days to a category kept **Never**, deletes files at the next sweep. **Save** then waits for **Preview**, which counts what the next sweep would delete, per category and in all, and the files kept because they are in use.

::screenshot{id=xtm-one-administer-settings-storage alt="The Storage & retention section of Settings > Usage and quotas: six categories of stored files, from Agent working files to Check pictures. Agent working files and Agent deliverables are kept 60 days after the last use, Check pictures 7 days after creation, the other three Never; each row gives its files and their size."}

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
- **Usage and quotas** > **Storage & retention**: **Chat uploads** at `365` days, as the company's data policy asks. **Preview** finds nothing older to delete, and she saves.
- **Agents and automations** > **Platform Health**: she turns on **Enable Platform Health** with **Log every write instead of executing it**, to read what it would change before letting it act.
- **Organization** > **Profile**: hospitality and Europe on **Identity and risk**, the payment platform and the booking engine as crown jewels on **Business**. She imports the hotels' assets from OpenAEV with **Draft** > **From the platforms**, which fills the **Exposure** tab, and keeps them in sync every day from **Knowledge and sources**.

The next morning, the audit shows no `Critical` finding, and Julien Bernard sees the SOC's usage in **Model usage**.
:::

## What's next

- [Configure single sign-on](/xtm-one/administer/sso/)
- [Protect your data and approve what agents do](/xtm-one/administer/ai-governance/)
- [Keep secrets in your vault](/xtm-one/administer/secret-vaults/)
- [XTM One configuration reference](/reference/xtm-one/configuration/)
