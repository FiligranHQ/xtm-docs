---
title: Find the files agents created and received
description: Find again, in XTM One's Files page, every document you gave an agent and every report, export or image an agent produced, filter them, download them and open the conversation or run they came from.
doc_type: how-to
products: [xtm-one]
personas: [cti-analyst, soc-analyst]
owner: docs-xtm-one
last_reviewed: 2026-11-16
migrated_from:
  - xtm-one:/usage/files/
---

**Files** is the one place that lists every file you and your AI agents exchanged: the documents you attached in a conversation, and what agents produced, such as reports, CSV exports, images, spreadsheets and decks exported to PDF. You find a deliverable again without remembering which conversation or which assignment made it, download it, and open the place it came from to ask for a new version.

::diagram{id=xtm-one-files-sources}

## Before you start

You see exactly the files you may download, and nothing else:

- the files you attached, and those generated for you, in your own conversations;
- the files of conversations shared with you;
- the outputs of assignment runs you can open: your own agents' runs, and those of company-managed agents.

An administrator can turn on **Hide activity from non-admins** for a company-managed agent or fleet. The outputs of its runs then leave **Company** for everyone but administrators and its owner, and the runs made as you stay in **Mine**. See [Create an agent](/xtm-one/build/agents/).

Administrators get no wider view here: they reach another person's conversation files only through the audited access on the conversation itself.

## Find a file

:::steps
1. Open :menu[Work > Files].
2. Type part of the file's name in the search field.
3. Click **Filters** and choose a segment in each of its three filters. The button then counts the filters that narrow the list.

   | Filter | Segments | Question it answers |
   | --- | --- | --- |
   | **Context** | **Everywhere**, **Conversations**, **Runs**, **Other** | In a chat, from an assignment run, or elsewhere (datasets, tool tests) |
   | **Origin** | **Any origin**, **Uploaded**, **Generated** | Did a person attach it, or did an agent produce it |
   | **State** | **Any state**, **Available**, **Deleted** | Can you still download it, or did the retention policy delete its content |

4. Choose whose files, beside **Filters**: **Mine**, your own files (attached by you, in your conversations or from your runs); **Company**, the outputs of company-managed agents' runs; **Shared**, the other files you may read, such as those of a conversation shared with you; or **All** of them.
5. Read the counter at the right of the filters: the number of files for the current filters.

::screenshot{id=xtm-one-files-list alt="The toolbar of the Files page with the Filters panel open: Context with Everywhere, Conversations, Runs or Other; Origin with Any origin, Uploaded or Generated; State with Any state, Available or Deleted. Beside it Mine, Company, Shared or All, and the Group switch turned on."}
:::

Each of **Mine**, **Company**, **Shared** and **All** shows how many files it holds under the other filters, whatever the **State**. With none chosen, the page opens on the first one that holds a file. When **Mine** is empty while the others are not, **Show all** moves to every file you can read.

**Group**, on by default, folds the list by conversation or run, with its title and owner. Switch it off for a flat list, the most recent first, where each row names its context.

The filters, the search, the grouping and the page are kept in the page's address: send it to a colleague and they see the same view, limited to what they may read.

What an agent reads on the web with its web fetch tool is kept as a file of the conversation too: a PDF or a spreadsheet as the document itself, a web page as its HTML. Reading the same page twice with the same content keeps one file.

## Act on a file

Each row shows the file's name, type and size, when it was added and by whom, and **Uploaded** or **Generated**. Its menu offers:

- **Download**, the file itself.
- **Open conversation** or **Open run**, the place the file came from, to read it in context or ask the agent for a new version: see [Get answers from an agent in the chat](/xtm-one/use/chat/#get-files-back-and-ask-for-changes).
- **Delete**, the file and its stored copy. The file's owner, the owner of its conversation, or whoever can manage the run that produced it can delete it; someone a conversation was only shared with can read the file but not delete it. A message or a deck that referred to a deleted file shows it as no longer available.

A link to a file from a chat message or a run's attachments opens **Files** already narrowed to that conversation or run, with the file highlighted. The **Showing files of** chip names the scope: close it to return to the full list.

Two kinds of files are elsewhere: presentations have their own page, [Turn an agent's work into a presentation](/xtm-one/use/presentations/), but a deck exported to PDF or PowerPoint is a file here; and the logos and images of presentation templates stay with their template.

:::example[Find the indicator export Night shift triage produced]
On 12 April 2026, Julien Bernard, SOC manager at Grand Meridian Hotels, needs the CSV of Velvet Harbor indicators that **Night shift triage** produced for Paul Garnier during the night. Paul shared the conversation with him. Julien opens :menu[Work > Files]:

| Filter | Value |
| --- | --- |
| Search | `velvet` |
| **Filters** > **Context** | **Conversations** |
| **Filters** > **Origin** | **Generated** |
| Whose files | **Shared** |

The list shows Paul's conversation with **Night shift triage**, and under it `velvet-harbor-indicators-block.csv`, **Generated**. Julien clicks **Download** and loads the file into the EDR's block list. He can read the file but not delete it: the conversation is Paul's.
:::

## Know when a file is deleted for you

An administrator decides how long each kind of stored file is kept, in **Storage & retention**: see [Set up the platform in Settings](/xtm-one/administer/settings/#usage-and-quotas). By default, the files agents work on and the files they deliver to you go after 60 days without use; the files you attach are kept for good.

- **Before**: the row of a file the retention policy will delete reads **Expires {date} if unused**. Hover it for the time. For the files agents work on or deliver, each download, preview or reading by an agent moves the date forward. A file that an open work item, a workset, a presentation or a report still uses is kept past it.
- **After**: only the content goes. The row stays, greyed, with "Deleted on {date} by the retention policy", and offers no **Download**; **Open conversation** or **Open run** still leads to its place. **State** > **Deleted** lists these files.
- **In the conversation or the run**: the file shows as a card with a dashed border, its name, its size and the same sentence, in place of the file or its preview. An old link to it says the file was deleted by the retention policy.
- **For the agent**: an agent that reads the file again is told it was deleted, and when, instead of getting its content. Ask it to produce the file again if you still need it.

**Delete**, in a row's menu, is different: it removes the file and its row from the list at once.

## What's next

- [Review what agents did](/xtm-one/use/work-history/), for the runs whose outputs are listed here.
- [Let an agent run code in a sandbox](/xtm-one/use/sandbox-workspace/), where an agent's working files live between calls.
- [Give agents your documents with knowledge bases](/xtm-one/build/knowledge-bases/), to make a file searchable by every agent.
