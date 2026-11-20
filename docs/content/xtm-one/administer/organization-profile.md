---
title: Set up the organization profile agents read
description: Fill the organization profile XTM One agents read, import and sync it from OpenCTI, OpenAEV and OpenCRQ, list your assets, set the prioritization policy, and choose what every agent receives.
doc_type: how-to
products: [xtm-one]
personas: [admin]
owner: docs-xtm-one
last_reviewed: 2026-11-19
---

Agents rank, scope and estimate with what they know of your organization: its sector and regions, its crown jewels, its tech stack, its assets and what the internet can reach. They read it in the organization profile, in :menu[Administration > Settings > Organization], split into five tabs. This page fills the profile from your platforms or by hand, keeps it in sync, sets the rules the CTEM funnel applies, and chooses how much of it every agent receives.

::screenshot{id=xtm-one-administer-organization-import alt="The Import from the platforms dialog of Settings > Organization at its first step, What to import: the sections Assets, Asset groups, AI systems, Tech stack, Crown jewels and PIRs, and Quick setup to fill the empty fields from every connected platform."}

## Before you start

- Your account is an administrator. Only administrators open Settings, and only they change the profile.
- The profile is shared context. The info bubble next to its title, **About the organization profile**, says it: "Shared context: the whole profile is visible to every user and every agent of this XTM One platform."
- An import reads the source with your own permissions. The source's access rules, such as organization segregation and markings, are not carried over. This applies to the profile only: integrations, dashboards and every other feature keep each user's permissions.
- **From the platforms** needs OpenCTI, OpenAEV or OpenCRQ connected, and readable by you. It uses no AI and works on the Community Edition.
- **With AI** and **From a connected integration** need AI switched on. See [Run XTM One with AI switched off](/xtm-one/administer/running-without-ai/).

## What the profile holds

Each card of **Profile** is one part of what agents read, and each card sits on one of five tabs. A list card shows how many rows it holds, and a badge when some rows need attention.

| Tab | Card | What agents learn from it |
| --- | --- | --- |
| **Identity and risk** | **Identity**, **Risk appetite and costs**, **Ticketing and intelligence** | Size, regions, money thresholds, the ticketing project and the OpenCTI PIRs to follow |
| **Business** | **Crown jewels and business services** | What matters most, its owner, its criticality and the assets it runs on |
| **Business** | **Tracked organizations** | The providers, customers, partners and subsidiaries you depend on: see [Track the organizations your company depends on](/xtm-one/administer/tracked-organizations/) |
| **Technology** | **Tech stack** and **AI systems inventory** | The products you run, and the AI systems you expose |
| **Exposure** | **Assets and exposure** and **Asset groups** | What runs where, and what the internet can reach |
| **Exposure** | **Prioritization policy** | The rules the CTEM funnel and its agents apply to findings |
| **Knowledge and sources** | **Enterprise knowledge** | Your procedures, which agents search on demand |
| **Knowledge and sources** | **Syncs** and **Organization context in agents** | Not read by agents: the imports run on a schedule, and how much of the profile every agent receives |

Each tab has its own address, so you can bookmark it or send it to a colleague: `/settings/organization` for **Identity and risk**, then `/settings/organization/profile/` followed by `business`, `technology`, `exposure` or `knowledge`. Add `?card=` and a card's key, such as `?card=tech_stack`, to open the tab on that card. The CTEM pages link there when a stage lacks its data: **Add tech stack** opens the **Tech stack** card of **Technology**.

::screenshot{id=xtm-one-administer-organization-profile-tabs alt="The Technology tab of the organization profile of Grand Meridian Hotels. Above the tabs, the line that counts the fields not filled, with Continue onboarding in chat. On the tab bar, Identity and risk 4/10, Business 0/1 and Technology 1/2. At the top of the tab, AI systems inventory among the fields not filled, then the Tech stack card with Example Edge Gateway."}

Agents read some fields of the profile as a contract: the organization name, sector, regions and size, the crown jewels, the tech stack, the AI systems, the PIRs, the money thresholds, the hourly rate, the currency and the ticketing target. A tab that holds some of them shows how many are filled, such as `1/2`. While fields are not filled, a line above the tabs counts them, with **Continue onboarding in chat**, and each tab lists its own at the top under "Fields not filled, reported by every agent that needs them:". Click one to open the form on that field.

Each save writes a new version of the profile. Agents cite the fields they used and name the ones not filled; they never guess them.

**Edit** opens the whole profile, on every tab. Switching tabs keeps what you typed, and one **Save** at the bottom saves every tab. Until then, a tab shows a dot when it holds changes not saved yet, a sparkle when a draft filled some of its fields, and an alert when one of its fields is refused: **Save** opens the tab of the refused field. Closing or reloading the browser page with an edit not saved asks you first.

## Import a section from the platforms

An import fills one section from one platform. Nothing is saved before you import, and the rows typed by hand are kept.

:::steps
1. Open :menu[Administration > Settings > Organization], click **Draft**, then **From the platforms**.
2. In **What to import**, pick the section: **Assets**, **Asset groups**, **AI systems**, **Tech stack**, **Crown jewels** or **PIRs**.
3. In **Source**, pick the platform. A platform you cannot read is greyed out with its reason: connect it in :menu[Administration > Settings > Integrations].
4. In **Filters**, narrow what is read. The pickers list the source's own values: labels, markings, asset groups, tags, criticality, exposure, platform, creation date and more.
5. In **Mapping**, check which field of the source fills each column of the profile, with an example value.
6. In **Preview**, check the counts and the sample rows.
7. Click the import button. It names the rows it writes, such as "Import 42 rows".
:::

Each section has its own sources. Assets come from OpenAEV or OpenCTI, and crown jewels from OpenCRQ. Asset groups, AI systems and the security platforms of the tech stack come from OpenAEV, and PIRs from OpenCTI. When OpenCTI gives an asset no type, **Type when OpenCTI gives none** sets it. An OpenCRQ asset rated Very high becomes a **Critical** crown jewel.

The preview counts the rows in four kinds:

- **New**: rows the profile does not hold yet.
- **Updated**: rows a previous import from this platform wrote.
- **Unchanged**: rows the profile already holds as the platform does.
- **Skipped**: a row typed by hand has the same name, and it is kept as typed.

Rows a previous import wrote that the platform no longer holds are kept by default. A switch under the counts removes them.

**Quick setup**, on the first step, fills every empty field from every connected platform in one click. The values land in the form for you to review, and nothing is saved before **Save**. The form opens on the first tab the platforms filled, a sparkle marks each tab they filled, and a notice lists each field filled and its platform, tab by tab, with their number. A draft **With AI** groups its proposals by tab in the same way; applied during an edit, it opens the first tab it changed.

## Keep a section in sync

An import runs once. To run it again on a schedule, turn on **Keep in sync** on its preview, then set the schedule:

- **Frequency**: **Hourly**, **Daily** or **Weekly**. A run saves a new version only when something changed.
- **Minute past the hour**, for an hourly sync, or **Time** and **Day**, for a daily or weekly one.
- **Time zone**: your browser's by default.

The button then reads "Import n rows and keep in sync". The sync runs as you: the preview says so.

The **Syncs** card, on the **Knowledge and sources** tab, lists every scheduled import, with its section, source, filters, schedule, last and next run, and who it runs as. Its **Status** is **Enabled**, **Disabled**, or **Paused** with the reason. From a row:

- **Run now** runs it at once.
- **Edit** opens the import again. Saving makes the sync run as you, and resumes it when it was paused.
- **Delete** stops it. The rows it imported stay in the profile.

## Fill the assets from a connected integration

When your assets live in a tool XTM One connects to, an agent can read it and propose the rows.

:::steps
1. Click **Draft**, then **From a connected integration**.
2. In **Integration**, pick the connected integration to read. The agent gets read-only access to it.
3. To run it every day, turn on **Keep the assets in sync every day**.
4. Click **Open the chat**.
:::

The chat opens with the request written. The agent previews the assets in the chat, and nothing is saved until you confirm. With the daily sync, an assignment of the agent applies the changes every day at 06:00, and keeps the rows written by hand. Turn it off in the agent's assignments.

## Edit a list

Each list card has its own **Edit**: crown jewels, tech stack, AI systems, assets, asset groups. One card is edited at a time. The other cards wait, with "Save or cancel the edit first."

- On crown jewels, tech stack and AI systems, **Paste a list** adds many rows at once. Type one row per line, the fields in the order of the columns, separated by a comma, a tab, a semicolon or a dash. A CSV exported from the same list works too. The preview says, line by line, what is added, already in the list or not read.
- **Add a crown jewel**, **Add a product**, **Add an AI system**, **Add an asset** and **Add a group** add one row.
- Select rows to set a field on all of them: owner, criticality or business service of crown jewels, vendor of products, owner, kind or exposure of AI systems.
- **Owner** and **Vendor** suggest the values already in the list, so one owner is not typed three ways.

The card's **Save** writes the new version.

## List your assets and exposure

**Assets and exposure**, on the **Exposure** tab, holds what runs where, and what the internet can reach. The prioritization funnel needs it to tell internet-facing findings apart.

- **Type** follows OpenAEV's taxonomy, such as **Host**, **Web application**, **Cloud resource** or **Security platform**, with its subcategories.
- **Criticality** goes from **Low** to **Very high**, or **Criticality unknown**.
- **Exposure** is **Internet-facing**, **Internal only** or **Exposure unknown**.
- **Asset group** is a group of the **Asset groups** card, or none.

Link each crown jewel to its assets in the **Assets** column of **Crown jewels and business services**, on the **Business** tab: in edit, **Link assets** lists the asset groups and the assets.

To load many assets, click **Import CSV**, then **Choose a CSV file**. Its header row names the columns, and only `name` is required:

- `category` and `subcategory` (or the older `type`), `criticality`, `environment`, `owner` and `group`.
- `internet_facing`: yes, no or unknown.
- `tech_stack`, `tags` and `crown_jewels`: several values separated by `;`.

A file holds up to 5,000 rows and 2 MB. The preview counts the assets read and the existing ones replaced, lists the crown jewel links, and gives each error by line. **Apply** writes them: a row replaces the asset of the same name, and the other assets stay.

## Set the prioritization policy

**Prioritization policy**, on the **Exposure** tab, holds the rules the CTEM funnel and its agents apply. Money thresholds stay in **Risk appetite and costs**, on **Identity and risk**. Without a policy, findings rank as before: click **Edit**, then **Use the recommended policy** to start from CVSS 7, EPSS 0.1 and known exploited vulnerabilities always in.

- **CVSS minimum**, from 0 to 10, and **EPSS minimum**, from 0 to 1. Empty: no threshold.
- **Minimum crown jewel criticality** keeps only findings on assets linked to a crown jewel at this level or above. **Any criticality** keeps them all.
- **Known exploited vulnerabilities always in**, on by default: a vulnerability in CISA KEV passes the thresholds whatever its scores.
- **Internet exposure required**, off by default, keeps only findings on internet-facing assets. It drops the findings on user endpoints too.

**Accepted risks** lists what the organization accepts until a date, each with its **Decision owner** and **Expires on**. **Add an accepted risk** adds one. A risk shows **Expires soon** within 30 days of its date, then **Expired**.

## Point agents to your procedures

**Enterprise knowledge**, on the **Knowledge and sources** tab, links the knowledge bases of your procedures, such as runbooks and patch policies. Click **Link a knowledge base**, pick it, and choose its **Kind**: **Standard operating procedure**, **Runbook**, **Patch policy**, **Reference** or **Other**.

Agents search them on demand, each within the knowledge bases its user can read. A knowledge base you cannot read shows **Not readable by you**.

## Choose what every agent receives

**Organization context in agents**, on the **Knowledge and sources** tab, is the default of every agent and assignment. Click **Edit** to change it.

- **Use organization context in every agent**, on by default, starts each agent with a summary of the profile. The summary gives the name, sector, regions, size and risk appetite, and one line per part of the profile.
- **Preloaded organization categories** adds whole parts of the profile to that summary. Empty, nothing more is preloaded: an agent reads each part with `get_organization_context` when it needs it.

An agent's **Configuration** > **General**, and an assignment's parameters, set **Organization context** to Inherit, Always or Off. Inherit follows this card. Off also takes the reading tool away. See [Create an agent](/xtm-one/build/agents/).

## Let an agent maintain the profile

The built-in tool group **Organization Profile Editing** lets an agent change the profile, run imports, and list, create and run syncs. Only an administrator binds it to an agent. Its tools are offered only on a turn that acts for an administrator. Each change is a new version authored by that administrator, with the agent and its basis kept.

The same group decides the proposals waiting for review. A proposal is a change nobody has applied yet: a batch of tracked organizations drafted with a model, or the fields the CTEM Assistant or Platform Health noted in a chat with an administrator, such as "add Salesforce to our tech stack". Neither writes the profile itself. The agent lists the proposals, newest first, by state (pending, accepted or rejected), then accepts or rejects one as the administrator it acts for:

- It names the fields, or the organizations, it keeps. Naming none rejects the whole proposal, for example one already applied by hand.
- An accepted field is checked against the current version of the profile, so a save made since the proposal is never overwritten by a field it did not name. The result is a new version.
- A proposal already decided cannot be decided again.

Each decision writes an entry in :menu[Administration > Audit logs], "Organization proposal accepted by agent" or "rejected by agent", followed by the agent's name. For the drafts of tracked organizations, see [Track the organizations your company depends on](/xtm-one/administer/tracked-organizations/).

:::example[Grand Meridian Hotels loads its attack surface]
Hélène Marchetti, CISO of Grand Meridian Hotels, wants the CTEM funnel to rank findings on the hotels' real assets.

- **Draft** > **From the platforms**: she picks **Assets** and OpenAEV, keeps the asset group of the Paris hotels in **Filters**, and reads 42 **New** rows in the preview.
- She turns on **Keep in sync**, **Daily** at 06:00, and clicks "Import 42 rows and keep in sync". The **Syncs** card now lists it, running as her.
- In **Crown jewels and business services**, she links the booking engine to its web servers with **Link assets**.
- In **Prioritization policy**, she clicks **Use the recommended policy**, then sets **Minimum crown jewel criticality** to **High**.
- In **Accepted risks**, she adds `CVE-2024-3094` on a legacy build server, with Marc Dupuis as **Decision owner**, until the server is retired.
- In **Enterprise knowledge**, she links "Grand Meridian security policies" as a **Runbook**.

The next morning, the sync finds no change and saves no version. The funnel now keeps only the findings on assets linked to a crown jewel rated High or above, such as the booking engine's web servers.
:::

## What's next

- [Track the organizations your company depends on](/xtm-one/administer/tracked-organizations/)
- [Set up the platform in Settings](/xtm-one/administer/settings/)
- [Create an agent](/xtm-one/build/agents/)
