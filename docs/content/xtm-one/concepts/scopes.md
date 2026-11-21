---
title: "Who sees what: containers, scopes and nesting"
description: How the personal, shared and company-managed scopes decide who sees and changes an XTM One object, what the three share permissions allow, and how containers and nested objects follow them.
doc_type: explanation
products: [xtm-one]
personas: [ai-builder, admin]
owner: docs-xtm-one
last_reviewed: 2026-11-08
migrated_from:
  - xtm-one:/usage/foundations/containers-and-scopes/
  - xtm-one:/usage/foundations/relationships-and-nesting/
---

Every agent, knowledge base, prompt, skill, integration or MCP server in XTM One has a **scope**: it is personal to its owner, shared with chosen people and groups, or company-managed for everyone. The scope decides who sees the object, who may use it in their own agents, and who may change it. Some objects are **containers** that hold others, such as a workspace holding conversations, and objects nested in a parent follow the parent's scope. This page explains the three scopes, the permissions a share grants, and the two questions to ask before you change anything. The steps to share are in [Share agents and resources](/xtm-one/build/sharing/).

::diagram{id=xtm-one-scopes-visibility}

## Three scopes

| Scope | Who sees it | Who changes it | Use it for |
| --- | --- | --- | --- |
| Personal | Its owner | Its owner | Building and testing: your changes reach nobody else |
| Shared | Its owner, and the people and groups it is shared with | Its owner, and the people and groups granted **Can configure** or **Full access (incl. chats)** | A team's agents and resources, maintained by the team |
| Company-managed | Every user of the platform, unless access is restricted | Administrators only | The approved standard every team builds on |

An object starts personal. Its owner turns on **Share with people and groups** in its configuration and picks people and groups; until then it reads "Private to you". An administrator makes it company-managed with the switch **Company-managed** followed by the kind of object, such as **Company-managed agent**: "Shared with all users. Only admins can edit." On a company-managed object, **Restrict access** limits it to the people and groups listed: everyone else on the platform stops seeing it.

::screenshot{id=xtm-one-scopes-sharing alt="The sharing of Night shift triage in edit mode: Share with people and groups on, visible to the people and groups below, with the SOC group listed."}

Administrators see every object of the platform, whatever its scope. By default, so do the agents working for them: through the agent builder, the work items and the XTM One MCP server, they reach every assignment. **Strict admin access for agent tools**, in :menu[Administration > Settings > Access and security > Security options], keeps those tools to the administrator's own, group-shared and company-managed assignments. A company-managed object keeps a real owner, whom an administrator can change.

## What a share grants

Each person or group an object is shared with gets one permission:

| Permission | Allows |
| --- | --- |
| **Read only** | View and use, but not edit the configuration. The object shows a **Read-only** badge and no **Edit** button |
| **Can configure** | View, use and edit the configuration |
| **Full access (incl. chats)** | For an agent only: also read other users' conversations with it and its traces, in its **Chats** and **Observability** tabs |

"Use" is what matters most: a person with **Read only** on a knowledge base can link it to their own agents, and an agent shared **Read only** can be chatted with. Give **Full access (incl. chats)** only to the people who must read what others asked the agent, since conversations may hold personal or sensitive content.

Presentations and workspaces use their own wording for the same two levels: **Can view** and **Can edit** for a deck; for a workspace, read-only grantees see it and what its agents built there, while grantees who can configure it also file their conversations into it, change, share and delete it.

## Containers: objects that hold others

| Container | Holds |
| --- | --- |
| Agent | Its assignments, its memory, its versions |
| Conversation | The messages, files and context of one chat |
| Workspace | Conversations on one topic, with the work items and shared state their agents build there |
| Knowledge base | Documents and the sources they are synchronized from |

Being in a container is not always ownership. An assignment belongs to its agent and is visible to whoever sees the agent. A prompt linked to an agent stays its own object, with its own scope: linking it does not share it, and the agent's viewers do not gain the right to change it.

Work items follow the work: one recorded by a company-managed agent is visible platform-wide, one recorded by a personal agent stays private to the person it works for.

## Before you change something

Two questions prevent most surprises:

1. **Am I changing this object, or a resource it links to?** A field of the form is the object itself. A card or a picker for a knowledge base, a prompt, a skill, an integration or an MCP server is a link: opening it and changing it changes every agent that links it.
2. **Who else sees this object?** A change to a personal object reaches only you. A change to a shared one reaches the people it is shared with. A change to a company-managed one reaches every user. When only you need a different behavior, duplicate it instead: see [How XTM One reuses what already exists](/xtm-one/concepts/deduplication/).

:::example[One agent for the SOC, one policy base for everyone]
At Grand Meridian Hotels:

- Sophie Laurent builds **Night shift triage** as a personal agent and tests it alone.
- She turns on **Share with people and groups** and adds the group **SOC** with **Read only**, and Julien Bernard, the SOC manager, with **Full access (incl. chats)**. Paul Garnier can chat with the agent on the night shift but not change it; Julien can read every night-shift conversation in the agent's **Chats** tab.
- Hélène Marchetti, the administrator, makes the knowledge base **Grand Meridian security policies** a **Company-managed knowledge base**: only administrators now edit the incident response policy in it.
- She turns on **Restrict access** on it and lists the groups **CTI team**, **SOC** and **Crisis cell**: their agents link it, and the other departments of the company do not see it.
:::

## What's next

- [Share agents and resources](/xtm-one/build/sharing/): the steps.
- [How access works in XTM One](/xtm-one/concepts/access/): users, groups and administrators.
- [How XTM One reuses what already exists](/xtm-one/concepts/deduplication/)
