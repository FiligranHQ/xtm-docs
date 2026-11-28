---
title: Prepare changes in a draft
description: Create and change OpenCTI knowledge in a draft, separate from the knowledge base, review every change it holds, and approve it to send its content for ingestion.
sidebar_position: 13
doc_type: how-to
products:
  - opencti
personas: [cti-analyst, cti-lead, admin]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/draftWorkspaces/
---

A draft is a workspace where you create, change and delete knowledge without touching the knowledge base. You use it to check what an import found in a vendor report before your colleagues see it, to propose changes that a senior analyst approves, or to prepare a large update in one go. Everything done in it waits until someone approves the draft, which sends its changes for ingestion. This page shows how to work in a draft, review it and approve it. File imports and forms create drafts for you; you find every draft in :menu[Data > Import], tab **Drafts**.

## Before you start

- Your role has the **Create / Update knowledge** capability, to create a draft and change data in it.
- Your role has the **Delete knowledge** capability, and you can edit the draft, to approve or delete it.
- Your role has the **Import knowledge** capability, to import files inside a draft.
- An administrator can grant capabilities that only apply inside drafts, for example to let a junior analyst propose changes that someone else approves. See [Grant rights with roles](/opencti/administer/roles/).

## How a draft works

::diagram{id=opencti-ingest-drafts-lifecycle}

Inside a draft you see the whole platform as it is, plus the changes of the draft. The rules of the platform still apply:

1. **Your rights do not change.** Capabilities, [confidence levels](/opencti/concepts/reliability-confidence/), [markings](/opencti/administer/markings/) and [organization segregation](/opencti/administer/organizations/) apply in a draft as they do outside it. Something shared with you after the draft was created stays hidden in that draft.
2. **Each change has an operation.** A draft records what it does to each object:

   | Operation | Meaning |
   | --- | --- |
   | Create | The object is new and exists only in the draft |
   | Update | The object exists in the knowledge base and the draft changes it |
   | Delete | The object exists in the knowledge base and the draft deletes it |
   | Update linked | The object is not changed, but something linked to it is, for example a new relationship to it or its addition to a container |
   | Delete linked | The object is not deleted, but it will be, because an object it depends on is |

3. **What does not change does not appear.** When an import brings an object that already exists, identical, the draft does not list it. The same object imported by another user appears, because its creators change.
4. **Approval sends only the changes.** Created objects are ingested whole. For updated objects, only the changes made in the draft are applied, so updates made meanwhile in the knowledge base are kept. Deleted objects are deleted; linked deletions are not applied.

:::only{opencti=">=7.261127"}
When an import in the draft renames an object the knowledge base already has, such as an Attack Pattern imported with its MITRE ID under a new name, the draft adds the previous name to its aliases. Approval keeps that alias, so a search for the old name still finds the object.
:::

## Create a draft and work in it

:::steps

1. Open :menu[Data > Import], tab **Drafts**, and click **Create Draft**.
2. Enter a **Name** and, if you want, a description, assignees, participants and an author. To restrict the draft to some people, set its authorized members. Click **Create**.
3. Click the draft in the list. You enter draft mode: a bar with the draft name stays at the top of every screen, as a reminder that what you do only changes this draft.

   ::screenshot{id=opencti-ingest-drafts-toolbar alt="In draft mode, a bar at the top of the screen shows the draft name, the processes running in it, and the Exit draft and Approve draft buttons."}

4. Work as usual: create an entity, an observable or a relationship, edit fields, add objects to containers, run enrichment connectors, import files. Every change goes to the draft.
5. Click **Exit draft** to return to the knowledge base. Click the draft name in the bar to come back to its overview.

:::

To enter a draft from the list without opening it, open its actions menu (⋮), choose **Switch to Draft**, and confirm in **Switch to Draft Mode**. A draft that is no longer open, such as a validated draft, cannot be entered from this menu.

### Who works on a draft

The draft's **Overview** shows its **Details**, its name and description, and its **Basic information**: the **Assignees** who must review it, the **Participants** who contribute, and its **Author**. Add people with **Add new assignees** and **Add new participants**, so that the team knows who reviews the draft. The edit button opens **Update a Draft**, to change its name and description. When the draft has authorized members, only they can open it.

### Other ways a draft is created

| From | What the draft holds |
| --- | --- |
| A [file import](/opencti/use/ingest/files/), with the validation mode **Draft** | What the import connector found in the file |
| A [form](/opencti/use/ingest/forms/) set to create a draft | The entities and relationships of the submission |
| An [analyst workbench](/opencti/use/ingest/workbench/), with **Convert to draft** | The content of the workbench |
| The **Drafts** section of an entity's **Data** tab, with **Add** | Changes about that entity. After approval, OpenCTI opens the entity |

## Review a draft

Open the draft from the list, or click its name in the draft bar. The overview counts the changes, and one tab per kind lists them with their operation: **Entities**, **Observables**, **Relationships**, **Sightings** and **Containers**. In the **Files** tab, you upload and import files that only exist in this draft: click **Import data**, and what the connector finds goes to this draft. Follow each run with **Show the imports**, where a failed run shows **Error**, as in [Import from files](/opencti/use/ingest/files/#run-an-import-again).

::::only{opencti=">=7.260811"}
The **Review** tab compares the draft with the knowledge base. On the left, it lists every changed object, which you filter by type, operation or name. Select one to see its changes on the right:

- for a created or deleted object, all its fields and its **Relations**, plus, for a container, the objects it contains;
- for an updated object, each changed field with its **Original Value** and its new value, then its **Relations** and the **Containers** it belongs to;
- for a linked update, the relationships and containers that changed, without fields.

::screenshot{id=opencti-ingest-drafts-review alt="The Review tab of a draft lists the changed objects on the left; on the right, an updated report shows each changed field with its original and new value."}
::::

The draft bar shows how many processes run in the draft, such as enrichments and background tasks. Click the count to see them in **Draft processes**, with their **Works** and **Tasks**.

## Approve a draft

:::steps

1. Enter the draft and check that **No processes running** shows in the draft bar.
2. Click **Approve draft**. The button is available once the draft holds at least one change.
3. Confirm with **Approve**. OpenCTI warns you when processes still run: what they would have changed is lost.

:::

The draft becomes read-only and its content is sent for ingestion. Open the draft from the list to follow the **Validation progress** and to see the changes it held.

:::example[Review new phishing indicators before they reach the SOC]
Your SOC blocks every indicator of the knowledge base with a score above 60. A junior analyst found new infrastructure of the Velvet Harbor phishing campaign and must not push it to the SOC before a senior analyst checks it.

1. **Create Draft** with the **Name** `Hotel IOC review` and the **Description** `New indicators of the hotel-booking phishing campaign, before they reach the knowledge base.`
2. In the draft, the junior analyst creates the IPv4 addresses `203.0.113.21` and `203.0.113.22` with a score of 70, and the indicator `frontdesk-booking-confirm.example` with a score of 85, all authored by Example CERT.
3. Still in the draft, the analyst opens the report "Velvet Harbor targets hotel booking systems", adds the new domain to its description and the label `under-review`.
4. The senior analyst opens the draft: its tabs list three objects with the operation Create and the report with Update. From version 7.260811, the **Review** tab also shows the new description of the report next to its **Original Value**.
5. The senior analyst finds that `203.0.113.22` belongs to a hotel booking provider, deletes it from the draft, and clicks **Approve draft**, then **Approve**.

Only the address `203.0.113.21`, the indicator and the change to the report reach the knowledge base, and the SOC starts blocking them. The provider's address never does.
:::

::::only{opencti=">=7.260811"}
When your administrator has set up a [draft workflow](/opencti/use/automate/draft-workflow/), the draft bar shows its status and the transitions you can take instead of **Approve draft**. A transition with the **Validate draft** action approves the draft.
::::

## What a draft cannot do

Some actions are not available in a draft, because they act beyond the data or have no draft version yet:

- running playbooks and inference rules, and enrolling entities in a playbook;
- sharing with organizations and changing authorized members of an entity;
- generating and downloading exports, and disseminating files;
- merging entities;
- creating an analyst workbench.

::::only{opencti=">=7.260811"}
:::xtm-one[AI agents propose, people approve]
Drafts are where XTM One AI agents put their work. A playbook that ends with **Send to AI agent** gives data to an agent, which writes what it creates to a draft. Import connectors that hand files to an AI agent also fill a draft. In both cases a person reviews the proposal in the **Review** tab and approves it. See [Automate with playbooks](/opencti/use/automate/playbooks/) and [XTM One in OpenCTI](/xtm-one/use/in-opencti/).
:::
::::

## Check that it works

After approval, the draft shows the status **Validated** and its validation progress completes. The created and changed objects appear in the knowledge base.

## What's next

- [Approve drafts step by step](/opencti/use/automate/draft-workflow/), to control who can move a draft to approval.
- [Import from files](/opencti/use/ingest/files/) and [Create knowledge with forms](/opencti/use/ingest/forms/), which fill drafts.
- [Grant rights with roles](/opencti/administer/roles/), for the capabilities granted only in drafts.
