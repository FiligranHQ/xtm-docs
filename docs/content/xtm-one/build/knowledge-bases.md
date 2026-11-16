---
title: Give agents your documents with knowledge bases
description: Create a knowledge base, fill it with files or keep it in sync with a source, check what agents will find, lock it as ground truth, and bind it to agents.
doc_type: how-to
products: [xtm-one]
personas: [ai-builder, cti-analyst]
owner: docs-xtm-one
last_reviewed: 2026-11-12
edition: ee
migrated_from:
  - xtm-one:/usage/resources/knowledge-bases/
---

A knowledge base is a collection of your documents that XTM One splits into passages and indexes, so the AI agents bound to it search it while they answer. An agent with your incident response policy quotes the policy instead of guessing what a hotel chain usually does. This page creates a knowledge base, fills it, checks what a search finds, and binds it to an agent. How knowledge bases fit with the other blocks is in [What agents are made of](/xtm-one/concepts/agents/).

::diagram{id=xtm-one-build-knowledge-bases-retrieval}

## Before you start

- Knowledge bases are an AI area: AI is switched on and the platform runs the Enterprise Edition. Automatic import sources also need an OpenCTI or OpenAEV in Enterprise Edition registered with XTM One.
- Any user can create a private knowledge base. Only an administrator can create a **Company-managed knowledge base**, open to every user and editable by administrators only.
- Search the list first: a collection that already exists can be shared with you instead of copied.

## Create it and add documents

:::steps
1. Open :menu[Agent Studio > Knowledge] and click **New Knowledge Base**.
2. Fill **Name**, **Tags** and **Description (optional)**. Agents and colleagues pick the knowledge base by its name and description, so say what it holds.
3. As an administrator, turn on **Company-managed knowledge base** when every user's agents should reach it.
4. Click **Create Knowledge Base**. Its page opens.
5. Open **Documents** and click **Upload Files**, then drop the files or click to browse. PDF, Word, Excel, Markdown, JSON, CSV and code files are accepted. To add a text you have at hand, click **Paste Text**, fill the filename and the content, and click **Add Document**.
6. Wait until the documents are indexed: **Overview** shows how many are ready and how many are still processing. A document that failed can be sent again with **Reprocess**.
:::

::screenshot{id=xtm-one-build-knowledge-bases-documents alt="The knowledge base Grand Meridian security policies open on its Documents tab, with Upload Files and Paste Text, and Export, Test Search and Re-embed at the top."}

To follow the documents of every knowledge base you can see at once, click **View Documents** on the list of knowledge bases. Each row gives the document, its knowledge base, its status and when it was added: search by file or knowledge base name, filter with **Status**, and sort by any of these columns. Hover an **Error** badge to read why the document failed, and click **Retry embedding** to send it again. **View Docs** on a warning under the list of knowledge bases opens this view on the knowledge bases concerned.

XTM One keeps each document whole, not only its passages: downloading it returns the file you added, and an agent that needs the full text reads the original. Keep very large documents out of a knowledge base agents should read whole. Above about 200,000 characters, a document is never loaded whole into an agent's context: the agent is steered to search it for passages instead. Splitting it into meaningful documents works better.

Only the text layer of a PDF is indexed by default: what a diagram, a chart, a screenshot or a scanned page shows is never found by a search. To index it too, open **Configuration**, click **Edit**, turn on **Describe PDF figures**, and click **Save changes** before you upload the PDFs: the switch applies to PDFs uploaded from then on, so a PDF already there is described only if you upload it again. The platform's vision model reads each page that carries a figure or has no text layer, and its description is indexed next to the page's text. Each such page costs one model call, up to 30 pages per document. On **Documents**, the PDF shows **Figures pending** until its pages are read, then how many pages had their figures described. **Figures partly described** and **Figures not described** say why when you hover them, for example when no model was available to read the pages.

## Keep it in sync with a source

On **Import Sources**, click **Configure Source**, pick the source type, fill its fields, choose the **Sync Schedule** (**Manual only**, **Every 6 hours**, **Every 12 hours**, **Daily** or **Weekly**), and click **Add Source**. **Sync now** starts a sync at once, and each source shows when it last synced and how many documents it brought.

| Source | What it imports | Needs |
| --- | --- | --- |
| **Website** | The pages of a public site, from a **Starting URL**, within **Max pages** and **Max depth**. | Nothing |
| **GitHub Repository (Public)** | Code and documentation of a public repository, a branch, a subfolder. | Nothing |
| **GitHub Repository** | The same, from a private repository. | A connected GitHub integration |
| **Notion** | A page, a database, or the result of a search, with sub-pages if you want. | A connected Notion integration |
| **OneDrive / SharePoint**, **Google Drive** | The files of a folder. | A connected Microsoft or Google integration |
| **Drata Policies** | Your compliance policies, in the **Policy statuses** you list (`ACTIVE` when you list none). | A connected Drata integration |

A source that needs an integration stays visible but disabled until one is connected. You can only choose an integration you can see; the source then syncs with it on every run, whoever starts it. When a sync fails, the people who can manage the knowledge base see the full error, which may name internal hosts; everyone else sees a short description.

## Check what agents will find

Click **Test Search** in the page header, type a question the way an agent's user would ask it, and click **Search**. The results are the passages a bound agent receives, sorted by relevance with their match percentage. If the passage you expect is missing, check **Documents** before blaming the agent: the file may be absent, failed, or an old version. **Test Search** stays disabled until the knowledge base has indexed content.

<!-- shot: xtm-one-build-knowledge-bases-test-search -->

## Lock it as ground truth

On **Configuration**, click **Edit**, turn on **Read-only content**, and click **Save changes**. Do it when the collection is the authoritative version of something, such as a policy other work is checked against. It stays fully searchable, but no agent can write to it, whatever its configuration grants, its documents cannot be added, changed or deleted, and the knowledge base cannot be deleted until you turn the switch off. Its import sources keep refreshing it, and **Re-embed** still works: disable the sources too to freeze it completely. The lock lives on the knowledge base, so it holds whatever an agent is given later.

**Index HTML as rendered text** (off by default) indexes HTML documents as the text a reader sees instead of their markup. Turn it on for exported pages whose markup outweighs their text, then click **Re-embed** to apply it to the documents already there.

## Bind it to an agent

In the agent wizard, pick it in **Knowledge bases to search**. On an existing agent, open **Configuration** > **Knowledge**, click **Edit** in **Knowledge Bases**, pick it, and **Save**. Each bound knowledge base reads **Read-only (search only)**: turn on its switch to let the agent store documents in it too. Tell the agent in its persona when to search it. See [Create an agent](/xtm-one/build/agents/).

A company-managed agent accepts only company-managed knowledge bases. Sharing a knowledge base with people and groups, at **Read only** or **Can configure**, works as for every resource: see [Share agents and resources](/xtm-one/build/sharing/).

:::example[The security policies the Velvet Harbor watcher quotes]
Sophie Laurent creates the knowledge base the **Velvet Harbor watcher** and the **Crisis briefer** agents search:

| Field | Value |
| --- | --- |
| **Name** | `Grand Meridian security policies` |
| **Tags** | `policy`, `incident-response` |
| **Description (optional)** | `Grand Meridian Hotels' incident response policy and crisis communication playbook: escalation rules, roles, who speaks to guests and press.` |
| **Documents** | `Incident response policy.pdf`, `Crisis communication playbook.pdf`, uploaded with **Upload Files** |
| **Read-only content** | On: the policies change only through Hélène Marchetti, the CISO |
| Sharing | **CTI team**, **SOC** and **Crisis cell**, **Read only** |

In **Test Search**, she asks `When must a guest data breach be escalated to the crisis cell?`: the first passage is the escalation section of the incident response policy. She binds the knowledge base to **Velvet Harbor watcher**, which now quotes that section when a new Velvet Harbor report names a Grand Meridian hotel.
:::

:::xtm-one[Agents can file what they learn]
An agent with write access to a knowledge base stores documents in it and revises them, for example a running list of the indicators seen in Velvet Harbor reports. A revised document is re-indexed and shows **pending** for a moment, and **Activity** records every change with its author.
:::

## Read the page

| Tab | What it is for |
| --- | --- |
| **Overview** | How much is indexed, the recent documents, the embedding model, the agents bound to it. Start here. |
| **Documents** | The files themselves: add, download, delete, reprocess, and whether the figures of a PDF were described. |
| **Import Sources** | The sources kept in sync, their schedule and last status. Not shown for a system-managed knowledge base. |
| **Configuration** | Name, description, tags, the read-only and indexing switches, sharing, owner. |
| **Activity** | What changed, which syncs ran, who edited the configuration. Check it when answers change. |

The badges explain missing actions: **System managed** means the platform fills it, from an assignment's runs; **Content locked** (**Locked** on the list) means **Read-only content** is on; **Company-managed** with **Read-only** means you may read it but not administer it.

## What's next

- [Create an agent](/xtm-one/build/agents/) that searches it.
- [Share agents and resources](/xtm-one/build/sharing/).
