---
title: Find anything on the platform by its name
description: Find an agent, an assignment, a conversation, a document or any other object you can see in XTM One by its name, from the top bar, then narrow the results by type.
doc_type: how-to
products: [xtm-one]
personas: [cti-analyst, soc-analyst, ai-builder]
owner: docs-xtm-one
last_reviewed: 2026-11-12
---

The **Search the platform** field, at the top of every page, finds any object you can see by its name: an agent, a fleet, an assignment, a conversation, a work item, a document, a prompt, a file, and about twenty types in all. You open the brief an agent wrote last week, or the agent behind it, without knowing which menu it lives in. The **Search** page lists every match, narrows it to one type, and keeps the search in its address, to bookmark or send to a colleague.

::screenshot{id=xtm-one-search-results alt="The Search page for velvet harbor: at the left, All with 4 matches, then Agents 1, Assignments 2 and Conversations 1; at the right, the table of the four matches, Velvet Harbor watcher, Weekly Velvet Harbor brief, SOAR case to CTI brief and Velvet Harbor's new infrastructure, with their Type, Context and Updated."}

## Before you start

Search needs no AI: it is open to every user, in every edition. It finds only what you may open, by the same rule as each list:

- agents, fleets, flows, knowledge bases, prompts, skills, templates, presentations, videos, integrations, MCP servers and custom tools: your own, those shared with you or one of your groups, and the company-managed ones, unless their access is restricted;
- assignments: those of the agents you see;
- conversations: your own, those shared with you or one of your groups, and those whose link is open to the organization;
- work items: those you can read on the **Work Items** tab of :menu[Work > Operations];
- documents: those of the knowledge bases you see;
- files: those you may download, as on :menu[Work > Files];
- reports: your own;
- intents: all of them;
- users and groups: for administrators only.

Administrators get no wider view: besides the users and groups, their results hold what any other user would see, and a colleague's private agent is not among them.

## Search from the top bar

:::steps
1. Press `Ctrl K` (`Cmd K` on a Mac), or `/` when you are not typing in another field. The cursor goes to **Search the platform** and selects what it held.
2. Type at least two characters of the name. The list under the field shows:
   - the matching objects, under the name of their type, three per type at most, the type with the best match first. The start of an object's description follows its name;
   - the matching **Pages** of the navigation, four at most. An entry with a lock needs the Enterprise Edition or AI switched on;
   - **See all results**, with the number of matches.

   ::screenshot{id=xtm-one-search-typeahead alt="The list under the search field of the top bar for velvet harbor: the agent Velvet Harbor watcher, the assignments Weekly Velvet Harbor brief and SOAR case to CTI brief and the conversation Velvet Harbor's new infrastructure, each under its type, then See all results with 4."}
3. Click a result, or reach it with the arrow keys and press `Enter`, to open it.
4. To see every match, click **See all results**, or press `Enter` without moving in the list: the **Search** page opens.
:::

"Searching..." shows while the list loads, and "No match" when nothing matches. `Escape` closes the list; a second `Escape` leaves the field.

A result opens where you work on it:

| Result | Opens |
| --- | --- |
| A conversation | The conversation, in :menu[Chat] |
| A work item | The **Work Items** tab of :menu[Work > Operations], with the item open |
| A document | Its knowledge base |
| A file | :menu[Work > Files] |
| A flow | Its builder |
| An intent | The **Intents** list |
| Any other object | Its page |

## Narrow and sort the results

The **Search** page lists every match you can see. **About search**, next to its title, says what it covers.

- At the left, **All** counts every match, and each type that holds one follows with its count. Click a type to list only its matches; the browser's back button returns to the previous choice.
- The table shows the **Name** of each match with its type's icon, its **Type**, its **Context**, the start of its description (hover it for the whole text), and when it was **Updated**. On a narrow screen, **Context** and **Updated** give way first. **Columns** hides or shows **Type**, **Context** and **Updated**.
- The matches come by relevance: a name equal to your search first, then the names that start with it, then the others, the most recently changed first within each. Click the **Name** or **Updated** header to sort by it instead; **Updated** starts with the most recent.
- The field above the table changes the search without leaving the page. Show 10, 25, 50 or 100 matches per page: the page remembers your choice.
- **All** goes through the first 200 matches. Beyond, choose a type: its list goes to the last match.

The address keeps the search and the type, such as `/search?q=velvet+harbor&type=assignment`. Bookmark it, or send it: whoever opens it sees only the matches they may open.

## How a name matches

- A result matches when each word you type appears, whatever its case, in its name or its description. The order does not matter: `harbor velvet` finds **Velvet Harbor watcher**, and so does `velv harb`. Only the first eight words count.
- What the search reads as a name depends on the type: the title of a conversation, a presentation, a video or a report, the file name of a document or a file, the full name and email address of a user. Conversations, documents, presentations, videos, reports and MCP servers are found by their name alone, not by a description.
- The search reads names, not contents: neither the text of messages nor the contents of documents. To search inside a knowledge base's documents, use its **Test Search**: see [Give agents your documents with knowledge bases](/xtm-one/build/knowledge-bases/#check-what-agents-will-find).
- Workspaces, runs, variables, relays and browser profiles are not searched: find them in their own lists.

## Run a command or jump to a tab

Start the search with one of two characters to change what the field finds:

| Start with | The list shows |
| --- | --- |
| `>` | The commands open to you on the page you are on, each after the name of its area, such as **Create an agent** and **Import an agent from a file** for agents, or **Invite a user** for users. Type a few letters to narrow them |
| `@` | The tabs and sections of the page you are on: choose one to jump to it. On a page without any, the list says "This page has no sections to jump to" |

A command that needs the Enterprise Edition with AI switched on carries a badge when your platform lacks it: choosing it opens a dialog that says why.

:::example[Find everything on Velvet Harbor]
On 12 April 2026, Camille Roux, CTI analyst at Grand Meridian Hotels, wants the assignments that brief the CTI team on Velvet Harbor. She presses `Ctrl K` and types `velvet harbor`. The list shows:

| Type | Match |
| --- | --- |
| Agents | **Velvet Harbor watcher**, which Sophie Laurent shared with the CTI team |
| Assignments | **Weekly Velvet Harbor brief**, and **SOAR case to CTI brief**, found by its description: "When the SOAR of the SOC opens a case that mentions Velvet Harbor..." |
| Conversations | **Velvet Harbor's new infrastructure**, her own conversation with the watcher |

She presses `Enter`. The **Search** page opens on `/search?q=velvet+harbor`, with **All** at 4. She clicks **Assignments**: the address ends with `&type=assignment`, and she opens **Weekly Velvet Harbor brief** to check its next run.

She sends the address to Paul Garnier, of the SOC. The watcher, its two assignments and Camille's conversation are not among his results: the watcher is shared with the CTI team only, and the conversation is Camille's.
:::

## What's next

- [Get answers from an agent in the chat](/xtm-one/use/chat/), to go on with a conversation you found.
- [Find the files agents created and received](/xtm-one/use/files/), to filter files by where they come from.
- [Share agents and resources](/xtm-one/build/sharing/), so that your colleagues find them too.
- [Start with XTM One](/xtm-one/use/getting-started/), for a tour of the navigation.
