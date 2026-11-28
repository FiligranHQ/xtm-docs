---
title: Review what a file import found
description: Review, correct and complete the objects an import connector found in a file, in an OpenCTI analyst workbench, then validate it or convert it to a draft.
sidebar_position: 14
doc_type: how-to
products:
  - opencti
personas: [cti-analyst]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/workbench/
---

An analyst workbench holds the objects found in one imported file, outside the knowledge base, so you can correct them before you import them. This page shows how to open a workbench, edit its content and validate it. Workbenches predate [draft workspaces](/opencti/use/ingest/drafts/), which are the default way to review imports: use a workbench when your team already works with them, and convert it to a draft when you need more.

## Before you start

- Your role has the **Upload knowledge files** and **Import knowledge** capabilities, to import a file into a workbench.
- Your role has the **Create / Update knowledge** capability, to validate a workbench or convert it to a draft.
- An import connector that reads STIX bundles, such as `ImportFileStix`, runs on the platform: validating a workbench sends its content to that connector.

:::only{opencti=">=7.261127"}
The **Analyst workbenches** tab and the workbenches in it open only for a role with **Import knowledge**. Without it, the tab is hidden, and :menu[Data > Import] opens the first tab your role can open, such as **Drafts**. A link to a workbench sends you there too.
:::

## Workbenches and drafts

| | Analyst workbench | Draft workspace |
| --- | --- | --- |
| Holds | The objects of one file, as a STIX bundle | Any change: creations, updates, deletions |
| You edit it | In the workbench's own tables | Anywhere in the platform, in draft mode |
| Enrichment, background tasks | No | Yes |
| Several files at once | No | Yes |

## Get a workbench

A workbench comes from one of three places:

- **A file import.** In the step-by-step import, choose **Workbench** as the **Validation mode**. This is offered when you import a single file. See [Import from files](/opencti/use/ingest/files/).
- **An empty workbench.** In :menu[Data > Import], open the **Analyst workbenches** tab and click **Create Workbench**. Give it a **Name**, optional **Labels** and **File marking definition levels**, then click **Create**.
- **An entity.** The **Analyst workbenches** section of an entity's **Data** tab lists the workbenches of files imported on that entity, and its **Add** button creates one.

You cannot create a workbench while you are in a draft.

## Edit and validate a workbench

:::steps

1. Open :menu[Data > Import], tab **Analyst workbenches**, and click the workbench.
2. Check each tab: **Entities**, **Observables**, **Relationships**, **Sightings** and **Containers**. Each lists the objects of that kind found in the file.
3. Fix what the connector got wrong. Click an object to change its type or fields, select objects to delete them or apply a marking, and use **Add entity**, **Add observable** or **Add container** to complete the file.
4. Link what belongs together. When you add or change an entity, click **Add and complete** or **Update and complete**: the next step proposes the other objects of the workbench it can be linked to. Choose them and click **Add context**. For a container, choose the objects it contains and click **Update context**.
5. Click **Validate this workbench**.
6. In the **Validate and send for import** window, choose the **Connector** that imports the content, then click **Create**. When the workbench belongs to an entity, turn on **Refresh entity** to start from the entity as it is now, rather than as it was when the workbench was created.

:::

### What the lists tell you

Each list shows the **Type** of each object, its name or value, its **Labels** and its **Marking definitions**. **Already in plat.** says whether your knowledge base already holds it, which tells you what the import will do:

| Already in plat. | Meaning |
| --- | --- |
| **Yes** | The object exists: the import updates it, within the limits of confidence, rather than creating a duplicate. See [Deduplication](/opencti/concepts/deduplication/) |
| **No** | The object is new: the import creates it |
| **Pending** | The platform is still checking |
| **Not applicable** | The object has no name or value to compare, shown as **Unknown** |

A file that names an intrusion set you already follow, such as Velvet Harbor, shows **Yes**: what the file adds, such as new aliases or relationships, joins your existing knowledge.

### Link entities as you go

The step after **Add and complete** depends on the type of the entity, and proposes only the objects of the workbench that fit:

- an intrusion set, a threat actor, a campaign or an incident: what it is **attributed to**, among the intrusion sets, threat actors and campaigns;
- the same types, malware, tools and channels: what it **targets**, such as sectors, organizations, locations and vulnerabilities, and what it **uses**, such as malware, tools and attack patterns;
- a sector, an organization or a location: who **targets** it.

A field marked **(reversed)** goes the other way: on a malware, **uses (reversed)** lists the intrusion sets and campaigns that use it.

::screenshot{id=opencti-ingest-workbench-entities alt="An analyst workbench of the file velvet-harbor-iocs.json lists the entities found in the file, such as the intrusion set Velvet Harbor and the malware Velvet Loader, with tabs counting the observables, relationships, sightings and containers, and the Convert to draft and Validate this workbench buttons at the top."}

The connector imports the content in the background, and the objects appear in the knowledge base with the confidence level of your user. See [Reliability and confidence](/opencti/concepts/reliability-confidence/).

:::example[Clean a partner's STIX file before it reaches the knowledge base]
Saltmarsh CERT, a fictional sector CERT, emails your team `velvet-harbor-iocs.json`, a STIX 2.1 bundle about the latest Velvet Harbor wave. It holds the intrusion set, the malware Velvet Loader, three IPv4 addresses and the domain `frontdesk-booking-confirm.example`, but no marking, and one address, `192.168.10.25`, is an internal address of the hotel that reported the attack.

1. Import the file with **Step-by-Step Import**, connector `ImportFileStix`, **File marking definition levels** `TLP:AMBER`, and **Validation mode** **Workbench**.
2. Open the workbench `velvet-harbor-iocs.json` in the **Analyst workbenches** tab.
3. In **Observables**, select `192.168.10.25` and delete it: it says nothing about the attacker and would expose the hotel.
4. In **Entities**, Velvet Harbor shows **Yes** in **Already in plat.**: the file will add to the intrusion set you follow, not create a second one. The bundle does not say whom the wave targets: click **Add entity**, create the sector `Hospitality`, click **Add and complete**, choose Velvet Harbor in **targets**, and click **Add context**.
5. In each tab, select all the objects and apply the marking `TLP:AMBER`, the level Saltmarsh CERT asked for in its email.
6. Click **Add container**, create the report `Velvet Harbor wave of January 2026` with Saltmarsh CERT as author, and choose all the objects as its content.
7. Click **Validate this workbench**, choose `ImportFileStix` as **Connector**, and click **Create**.

The report, the malware, two addresses, the domain and the new targeting relationship reach the knowledge base, all marked `TLP:AMBER`, and Velvet Harbor gains what Saltmarsh CERT knew; the internal address never does.
:::

## Convert a workbench to a draft

Click **Convert to draft** at the top of the workbench to move its content into a new [draft](/opencti/use/ingest/drafts/). In **Convert this workbench to a draft**, choose the **Connector** that reads the content and click **Create**. As with validation, **Refresh entity** starts from the current version of the entity the workbench belongs to. In a draft you can enrich the objects, run background tasks, change existing data, and approve the result.

## Check that it works

In the **Analyst workbenches** tab, click **Show the imports**, the icon at the end of the workbench's line. **File imports** lists the validation with its status and the number of objects processed, or **Error** when it failed: hover the status to read the message. A conversion to a draft has **Navigate to draft**, which opens that draft. **Delete this work** removes a line from the list, not what was imported. You can also follow the validation in the works of the connector. The objects then appear in their lists.

## What's next

- [Prepare changes in a draft](/opencti/use/ingest/drafts/), the default way to review imports.
- [Import from files](/opencti/use/ingest/files/), to bring in more data.
