---
title: Share a workspace with your team
description: Group the conversations of one investigation in an XTM One workspace, give its agents instructions and sources, and share it so the team's agents work from the same knowledge and work items.
doc_type: how-to
products: [xtm-one]
personas: [cti-analyst, soc-analyst]
owner: docs-xtm-one
last_reviewed: 2026-11-09
edition: ee
migrated_from:
  - xtm-one:/usage/workspaces/
---

A workspace is a named place for one line of work: an investigation, an audit, a customer. You file conversations into it, and the AI agents of those conversations work from the same instructions, documents and knowledge bases, share the work items and shared state they create there, and can search what was said in the workspace's other conversations. Shared with your team, it keeps an investigation together; kept to yourself, it keeps one topic apart from the others. A conversation filed in no workspace works as before, with your own work items and shared state.

::diagram{id=xtm-one-workspaces-shared-context}

## Before you start

- Workspaces need the Enterprise Edition. Managing them calls no model, so they keep working when AI is switched off.
- You can file only your own conversations, into a workspace you can change.

## Create a workspace

:::steps
1. Open :menu[Work > Operations], then the **Workspaces** tab. Each workspace is a card: its name, what it holds and how many work items are still open, its owner and when it changed. Hover its name for the rest, such as its description.
2. Click **New workspace**, at the top right of the page.
3. Enter a **Name** and, if you like, a **Description** of what it is for.
4. Click **Create**. The workspace is yours: nobody else sees it until you share it.

::screenshot{id=xtm-one-workspaces-list alt="The Workspaces tab of Operations, one card per workspace: Camille Roux's default workspace General, with nothing filed yet, and Velvet Harbor investigation, Shared, holding 2 conversations and 1 knowledge base; the search and the Owner and Status filters above the cards, New workspace at the top right."}
:::

You can also create one from the chat: **New workspace**, next to **Workspaces** in the sidebar, takes a name and creates it on Enter.

**List view** shows the workspaces as a table instead of cards, with the **Name**, **Owner**, **Slug**, **Status** and **Updated** columns; **Columns** hides the ones you do not need, and **Grid view** goes back to the cards. Either way, **Search workspaces...** finds a workspace by its name, its slug, its description or its owner's name. **Owner** keeps the workspaces of one person (**You** for yours), and **Status** keeps the **Default**, **Active** or **Archived** ones. **Export CSV** downloads the rows as filtered and sorted. **Rename**, in a workspace's actions menu, changes its name; its **Handle**, the slug shown on its row, stays, so existing links keep working. In the chat's sidebar, you edit the name in place.

Everyone has one default workspace, **General**, created the first time they open the list, listed first and marked **Default**. Nothing is filed in it on its own: a new conversation is in no workspace until you file it. **Make default**, in another workspace's actions menu, makes that one your default; only a workspace you own, not archived and not company-managed, can be. The default workspace cannot be archived or deleted.

## Give the workspace its knowledge

Click the workspace's card or row, or **Open workspace** in its actions menu. Its page opens on the **Overview** tab, the knowledge its agents receive. **New conversation**, at the top right, starts a conversation already filed in it (not while AI is switched off). In a conversation of the workspace, **Conversation options** > **Workspace knowledge** opens the same knowledge, with **Open workspace** to reach the page.

::screenshot{id=xtm-one-workspaces-page alt="The page of the Velvet Harbor investigation workspace: its description and the Shared badge under its name, New conversation at the top right, the Overview, Operations and Conversations tabs, and on Overview the instructions for the CTI team and the bound knowledge base Grand Meridian security policies."}

Every agent that works in a conversation of the workspace receives:

| Part | What it is |
| --- | --- |
| **Instructions** | Up to 8,000 characters on how agents should work there: the scope, the tone, what to check |
| **Sources** | Documents the agents work with: files, pasted text, a website, synchronized sources such as Notion, Google Drive, OneDrive or SharePoint, GitHub. **Manage sources** adds them, with the formats and limits of a knowledge base. Small enough, they are given whole; beyond a share of the model's context, they are searched |
| **Knowledge bases** | Knowledge bases bound to the workspace. Binding copies nothing and grants nobody access: someone who cannot see a knowledge base does not get it |
| **Skills** | Skills linked to the workspace, loaded by the agent when it needs them |
| **Conversation search** | The agents can search the workspace's conversations once each has been quiet for a while (30 minutes by default): what people wrote, the answers and the text of attached files, never the results of tools |

To add a file attached in a conversation to the sources, open **Conversation files** and choose **Add to the workspace's sources**. How knowledge bases are built is in [Give agents your documents with knowledge bases](/xtm-one/build/knowledge-bases/).

## File conversations in it

The quickest way is the chat's sidebar, which lists your conversations under **Workspaces**, one group per workspace, then **Not in a workspace**. A workspace's group stays closed until you open it, unless it holds the conversation on screen or you are searching:

- Drag a conversation onto a workspace, or use **Move to workspace** on its row; **Remove from workspace** takes it out.
- Start a conversation already filed with the **+** of a workspace (**New conversation in this workspace**).
- In an open conversation, **Conversation options** > **File in a workspace**, then pick a workspace, or **No workspace**.

The change applies from the next message: from then on, the agents of that conversation create and read their work items and shared state in the workspace. Filing into a workspace other people can see asks first, since their agents will be able to search what was written in the conversation.

An agent can file the conversation for you when you ask ("file this conversation in Velvet Harbor investigation"). Into a workspace other people can see, the move waits for your approval on an approval card, every time.

## Follow the work filed in it

The other tabs of the workspace's page show what it holds:

| Tab | What it shows |
| --- | --- |
| **Operations** | What is filed in the workspace, one list at a time: **Work Items**, **Shared State** or **Worksets**, with the filters of :menu[Work > Operations] |
| **Conversations** | **Your conversations**: your own conversations filed in the workspace; click one to open it in the chat. The conversations other people filed are not listed, but the agents can search them |
| **Team** | Only when the workspace hosts a [fleet](/xtm-one/build/fleets/): the fleet's members, and **Open the fleet graph** |

A fleet, a team of AI agents, works in one workspace, which keeps everything its agents produce. Whoever creates the fleet picks a workspace they manage, or leaves the default: a new workspace named after the fleet. Under the workspace's name, **Hosts fleet** and the fleet's name link to the fleet. Its **Operations** tab also lists what the fleet's agents create, even when they file it nowhere. See [Run agents as a team with fleets](/xtm-one/build/fleets/).

## Share it with your team

:::steps
1. In the workspace's actions menu, choose **Share**.
2. Switch on **Share with people and groups**.
3. Search for a person or a group, and add them.
4. Choose what each one may do:
   - **Read only**: they see the workspace and what its agents built there, but cannot file conversations into it or change it.
   - **Can configure**: they can also file their own conversations into it, change its knowledge, rename it, share it, archive it and delete it.
5. Click **Save**. **Who can see this workspace** says in words who the settings let in.
:::

The owner keeps the workspace whatever a member does; only an administrator can transfer it. Removing a person's access leaves their conversations filed, but their agents no longer read or write that workspace's work items and shared state.

A workspace shared with you shows **Shared by** with its owner's name, and **Read-only** when you may only read it. Its agents read the work items and shared state of every workspace you can see that is not archived, and write only where you can change the workspace.

Administrators also see **Company-managed workspace**: switched on, the workspace is visible to everyone and changed only by administrators. **Restrict access** then limits it to the people and groups listed.

## Archive or delete a workspace

- **Archive** hides the workspace from every picker and keeps everything it holds. Nothing can be filed into it until you restore it: choose **Archived** in the **Status** filter of the **Workspaces** tab, then **Restore** in its actions menu. A workspace that hosts a fleet cannot be archived until the fleet is archived or moved to another workspace.
- **Delete** removes the workspace after a confirmation. Its conversations are kept, in no workspace; its worksets are deleted with its instructions, sources and conversation search; bound knowledge bases and linked skills stay as they are. A workspace that still holds work items cannot be deleted: archive it, or clear its work items first. Neither can a workspace that hosts a fleet, even an archived one: move the fleet to another workspace, or delete it, first.

When a fleet or work items stop **Archive** or **Delete**, the actions menu shows it greyed out, with the reason under it.

:::example[Keep the Velvet Harbor investigation in one place for the CTI team]
On 12 April 2026, Camille Roux, CTI analyst at Grand Meridian Hotels, opens a workspace for the Velvet Harbor investigation:

| Where | Value |
| --- | --- |
| **New workspace**, **Name** | `Velvet Harbor investigation` |
| **Description** | `The January 2026 intrusion and what follows: reports, detections, crisis briefings` |
| **Instructions** | `Write for the CTI team. Name every Velvet Harbor technique with its ATT&CK identifier. Never share guest data in an answer.` |
| **Sources** | The OpenCTI report `Velvet Harbor: fake summit bookings against hotel front desks` exported to PDF |
| **Knowledge bases** | `Grand Meridian security policies` |
| **Share** | Group `CTI team` and Paul Garnier: **Can configure**; Hélène Marchetti: **Read only** |

She files her conversations with **Velvet Harbor watcher** into it. Paul files his conversation with **Night shift triage** about a new alert into the workspace: the agent finds Camille's notes through the conversation search, and the work items **Velvet Harbor watcher** created stay in this workspace, on its **Operations** tab, instead of mixing with the other topics.
:::

## What's next

- [Get answers from an agent in the chat](/xtm-one/use/chat/), where workspaces are organized from the sidebar.
- [Review what agents did](/xtm-one/use/work-history/): the work items and shared state of a workspace are also on the other tabs of :menu[Work > Operations].
- [Share agents and resources](/xtm-one/build/sharing/), for the same sharing rules on agents and knowledge bases.
