---
title: Teach agents a procedure with skills
description: Package a method as a skill, from a URL, pasted text or a package, attach it to agents that load it when a task calls for it, keep it up to date, and duplicate one to adapt it.
doc_type: how-to
products: [xtm-one]
personas: [ai-builder]
owner: docs-xtm-one
last_reviewed: 2026-11-08
edition: ee
migrated_from:
  - xtm-one:/usage/resources/skills/
---

A skill is a written procedure an agent follows for one kind of task: writing a detection rule, building a STIX bundle, drafting in a house style. You write it once as a Markdown file (a `SKILL.md`, in the open [Agent Skills](https://agentskills.io) format), attach it to the agents that need it, and each agent loads its full text only when a task calls for it. Ten attached skills cost an agent about as much prompt space as one. This page adds a skill, attaches it, keeps it current and adapts a copy of one; how skills sit beside prompts, knowledge bases and tools is in [What agents are made of](/xtm-one/concepts/agents/).

::diagram{id=xtm-one-build-skills-loading}

## How an agent uses a skill

An agent does not carry the text of every skill attached to it. Its prompt holds a catalog: for each skill, the name, the description, the compatibility note and the names of the files the skill bundles. When a task matches an entry, the agent loads the skill and its full instructions arrive at that point in the conversation; it can then read the bundled files it needs.

This is why the description matters most: it is all the agent sees until it decides to load the skill. Say when to use the skill, not only what it contains. Every load is a tool call in the run's trace, so the trace tells you which skill shaped an answer, and no load means the description did not match the task.

A platform engineer can cap how much one load returns and how many skills one run may load: see [Skill loading](/reference/xtm-one/configuration/#skill-loading) in the configuration reference.

## Before you start

- Skills are an AI area: the platform runs the Enterprise Edition and AI is switched on. Otherwise **Add Skill** opens the Enterprise Edition notice.
- Any user can create a private skill. Only an administrator can make it **Company-managed skill** (shared with all users, editable by administrators only).

## Add a skill

:::steps
1. Open :menu[Agent Studio > Skills] and click **Add Skill**. The **Create Skill** page opens.
2. Choose the **Source Type**:
   - **From URL**: paste the **Skill URL** of a `SKILL.md` file. XTM One keeps the skill in sync with it (see [Keep a skill up to date](#keep-a-skill-up-to-date)). For a file in a private GitHub repository, pick a connected GitHub integration under **Private repository access (optional)**, and optionally a **Tracked branch (optional)**: empty follows the default branch, a branch name follows that branch, a commit SHA pins one version.
   - **Paste Content**: paste the instructions in **Skill Content (markdown)**.
   - **Upload File**: a `.md` or `.txt` file.

   A YAML frontmatter with `name`, `description` and `compatibility` fills the matching fields; otherwise fill **Name (optional)**, **Description (optional)** and **Compatibility (optional)** yourself.
3. Add **Tags**, and, as an administrator, turn on **Company-managed skill** to offer it to everyone.
4. Click **Fetch & Create**, **Create Skill** or **Select & Upload**, depending on the source. The skill's page opens.
:::

::screenshot{id=xtm-one-build-skills-catalog alt="The Instructions tab of the skill write-a-sigma-rule: its frontmatter with the name and the description agents choose it by, above the four steps of the procedure, with Edit to change the SKILL.md content."}

To bring several skills at once, **Import Package** on the list takes a `.zip` (or a single `SKILL.md`). XTM One reads it first and shows the skills it found, the files it ignored and why, and a checkbox per skill. For a skill whose name you already have, choose **Skip** (the default), **Replace** or **Import as new**. Each skill is imported on its own, so one that fails does not undo the others.

## What a package brings with it

A skill package is a folder: its `SKILL.md`, and the folders the Agent Skills format defines. Two of them become platform objects when you import it:

- **`scripts/`**: each `.py` or `.js` script becomes a custom tool named after the skill and the file, so the agent can run it. They run only where custom tools are enabled: see [Write custom tools](/xtm-one/build/tools/).
- **`references/`**: each text file becomes a document of a knowledge base named `<skill name>-references`. The documents are searchable once the platform has indexed them.

These resources belong to the skill, not to the agent. Attaching the skill attaches them, marked **Via skill** in the agent's **Custom Tools** and **Knowledge Bases** cards; detaching or deleting the skill removes them. A packaged tool or knowledge base takes the skill's sharing, so a company-managed skill brings company-managed resources. The skill's **Resources** tab lists the tools it created.

## Attach it to an agent

On the agent's page, open **Configuration**, then the **Knowledge** section. In the **Skills** card, click **Edit**, select the skill and **Save**. **Quick Add by URL** creates a skill from a URL and attaches it in one go. The creation wizard offers the same choice under **Skills** in its advanced options: see [Create an agent](/xtm-one/build/agents/).

The pin next to an attached skill, **Always load this skill**, puts its full instructions in the system prompt on every turn instead of the catalog. Keep it for the one skill that defines how the agent works; pinning every skill rebuilds the oversized prompt the catalog avoids.

## Keep a skill up to date

A skill's page has five tabs:

| Tab | What it is for |
| --- | --- |
| **Catalog** | Name, description, compatibility, tags, the URL and its credential, the **License**, and two switches. **Enabled** off: the skill is no longer given to any agent, at once, even in a running conversation. **Hide from chat**: the skill stays out of chat contexts but remains available to agents. |
| **Instructions** | The `SKILL.md`, rendered, and **Edit** to change it. |
| **Resources** | The custom tools the package created. |
| **Versioning** | Every saved version of the content. Select two and click **Compare** to see what changed, **Side by side** or **Unified**. |
| **Activity** | The runs of the agents the skill is attached to, in a table (**Run**, **Status**, **Type**, **Agent**, **Started**, **Duration**), newest first. Filter them by status on the toolbar (**Running**, **Awaiting input**, **Completed**, **Failed**, **Cancelled**) or search them; click a row to open the run. |

A skill made from a URL is fetched again when its copy is more than a day old (the check runs every hour). **Refresh** in the header fetches it now, for example right after a change is merged. When the GitHub credential of a private repository stops working, the skill shows **Credential rejected**: agents keep using the last content fetched, and the next refresh after you re-authorize the integration in :menu[Tools & Integrations > Integrations] clears the badge.

**Export** downloads the skill as a package or as JSON, to move it to another platform. Sharing a skill with a group or with people is described in [Share agents and resources](/xtm-one/build/sharing/).

:::example[Teach the SOC agent the team's Sigma conventions]
The SOC of Grand Meridian Hotels writes detection rules for what the CTI team learns about Velvet Harbor. Sophie Laurent, the AI engineer, pastes the team's method as a skill:

| Field | Value |
| --- | --- |
| **Source Type** | **Paste Content** |
| **Name (optional)** | `Write a Sigma rule` |
| **Description (optional)** | `Use when asked to turn a technique, an indicator or an incident into a detection rule. Writes a Sigma rule with the Grand Meridian conventions: log sources, naming, tags, level and false positives.` |
| **Compatibility (optional)** | `xtm-one` |
| **Skill Content (markdown)** | The method: one rule per technique, `title` starting with `GM -`, the MITRE ATT&CK tag of the technique, the Windows process creation log source for workstations of the front desks, a `falsepositives` section, `level: high` only for a confirmed Velvet Harbor behavior |
| **Tags** | `soc`, `detection` |

She attaches it to **Night shift triage**, unpinned. When Paul Garnier asks at 02:00 for a rule detecting the PowerShell loader Velvet Harbor ran on 9 January 2026, the trace shows the agent loading **Write a Sigma rule**, and the rule follows the team's conventions.
:::

## Adapt a copy of a skill

A built-in skill is read-only, and so is a skill shared with you read-only. To change one for your own needs, duplicate it: the copy is a personal skill you edit freely, and the original does not change. Any skill you can open can be duplicated.

:::steps
1. On the skill's page, click **Duplicate**. The **Duplicate skill** dialog opens.
2. Keep or change the **Name**. Left empty, the copy is named `Copy of` followed by the original's name.
3. Optionally, pick a fleet you manage in **Use the copy in a fleet**. The agents of that fleet that use the original switch to the copy, each with its **Always load this skill** pin as it was. Agents outside the fleet, and the fleet's agents you cannot edit, keep the original. Leave **No fleet: keep every agent on the original skill** to attach the copy yourself later.
4. Click **Duplicate**. The copy's page opens; under its name, **Forked from** links to the original and gives the version copied. The confirmation counts the agents switched to the copy, and those you could not edit.
:::

::screenshot{id=xtm-one-build-skills-duplicate alt="The Duplicate skill dialog of the skill write-a-sigma-rule: the Name field holding Copy of write-a-sigma-rule, and Use the copy in a fleet on No fleet: keep every agent on the original skill, above the Duplicate button."}

The copy takes the original's instructions, description, compatibility, license, tags and bundled files, and starts its own history at version 1 in **Versioning**. It does not take the URL nor its GitHub credential: a copy of a skill made from a URL is never fetched again, so a refresh never overwrites your edits, and the platform never changes the copy of a built-in skill. The copy is private until you share it. A skill from a URL whose content was never fetched cannot be duplicated yet: **Refresh** it first.

:::example[A copy of the Sigma method for the booking servers]
**Write a Sigma rule** is shared with the SOC read-only, and its log source is the workstations of the front desks. Paul Garnier needs rules for the Linux servers of the booking platform. On the skill's page he clicks **Duplicate**:

| Field | Value |
| --- | --- |
| **Name** | `Write a Sigma rule - booking servers` |
| **Use the copy in a fleet** | **No fleet: keep every agent on the original skill** |

He clicks **Duplicate**. On the copy's page, **Forked from** names Sophie Laurent's skill; in **Instructions**, he clicks **Edit** and replaces the log source with the Linux process creation events of the booking servers. **Night shift triage** keeps loading Sophie's skill, unchanged.
:::

## Check that it works

The skill's card shows **Enabled**. In a chat with the agent, ask for the task the skill covers, then open the run's trace: a skill load names the skill. Without a load, rewrite the description so it says when to use the skill.

## What's next

- [Write reusable prompts](/xtm-one/build/prompts/), for short instructions an agent always receives.
- [Give agents your documents with knowledge bases](/xtm-one/build/knowledge-bases/), for material an agent searches rather than follows.
- [Create an agent](/xtm-one/build/agents/).
