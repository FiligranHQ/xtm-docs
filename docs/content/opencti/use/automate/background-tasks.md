---
title: Change many objects at once
description: Update, enrich, share, merge or delete many OpenCTI objects at once with a background task, then follow its progress in the task list.
sidebar_position: 3
doc_type: how-to
products:
  - opencti
personas: [cti-lead, cti-analyst, admin]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/background-tasks/
---

A background task applies the same action to many objects, so you can relabel, enrich, share or delete hundreds of entities without opening them one by one. This page shows how to start a bulk action from any list, follow it in :menu[Data > Processing > Tasks], and what each user can do with tasks.

## Kinds of background task

| Kind | Started by | What it does |
| --- | --- | --- |
| List task | Selecting objects one by one in a list | Applies the actions to the selected objects |
| Query task | Selecting all objects of a list | Applies the actions to every object that matches the list's filters and search, in batches of 500 |
| Rule task | Turning an inference rule on or off | Scans the whole knowledge base to create or remove inferred data. See [Inference rules](/opencti/configure/rules/) |

Notifications have their own bulk actions: marking many alerts as read, unread or deleted from your [alerts list](/opencti/use/automate/notifications/) also runs as a background task.

## Before you start

- Your role needs the capability that matches what you change:

  | You act on | Capability |
  | --- | --- |
  | Knowledge (entities, observables, relationships) | **Create / Update knowledge**; **Delete knowledge** to delete, restore or remove; **Merge knowledge** to merge |
  | Labels, vocabularies and other settings lists | **Manage labels** |
  | Users | **Manage credentials**, or administrator of their organization |
  | Your own notifications | None; **Manage credentials** for other users' notifications |
  | Imported files and workbenches | **Import knowledge** |
  | Dashboards, investigations, public dashboards | **Delete dashboards**, **Delete investigations**, **Manage public dashboards** |

- To see :menu[Data > Processing > Tasks], your role needs **Create / Update knowledge**.

## Run a bulk action

:::steps

1. Open a list, for example :menu[Analyses > Reports], and filter it down to the objects you want to change.
2. Select the objects with the checkboxes. To select every object that matches the filters, including those on other pages, use the checkbox in the column header. A toolbar appears with the number of selected elements.
3. Click the action in the toolbar. Depending on the list, you can:
   - **Update** fields: add, replace or remove labels, markings, an author, assignees, a score or another attribute;
   - send the objects to an **Enrichment** connector, or to **Indicators/observables generation**;
   - **Rule rescan** the objects so that inference rules run on them again;
   - **Add in container** or **Remove from the container**;
   - **Share with organizations** or **Unshare with organizations** (Enterprise Edition);
   - **Enroll in playbook**;
   - **Merge** the selected entities into one;
   - delete the objects.
4. Fill the action's form. With **Update**, click the **+** button under the first step to chain several changes in one task.
5. Check the summary in **Launch a background task**: each step, its field and its values, and the number of selected elements. A warning appears above 1,000 elements.

6. Click **Launch**. A message confirms the task and links to the task list.

:::

:::only{opencti=">=7.261117"}
**Enrichment** skips the summary: in **Entity enrichment**, turn on the connectors under **Selected connectors** and click **Enrich**, and the task starts at once. **Launch a background task** opens only when more than 1,000 elements are selected, so that you confirm before enriching that many.
:::

:::only{opencti=">=7.261109"}
The **Field** list of **Update** also offers the [custom fields](/opencti/configure/taxonomies/#custom-fields) of the selected type, by their label: **Replace** and **Remove** work on every custom field, **Add** only on a multiple selection list.
:::

:::example[Label and share the Velvet Harbor indicators]
Example CERT published indicators of the Velvet Harbor campaign, and the CTI team wants them labelled for the hospitality sector and shared with the member hotel chain Grand Meridian Hotels.

1. Open :menu[Observations > Indicators] and add the filters **Author** = Example CERT and **Indicator type** = malicious-activity.
2. Select every indicator with the checkbox in the column header.
3. Click **Update**. In **Update entities**, set **Action type** to **Add**, **Field** to **Labels** and **Values** to `hotel-sector`. Click **+** and add a second step: **Add**, **Marking definitions**, `TLP:AMBER`. Click **Launch** in **Launch a background task**.
4. With the same selection, click **Share with organizations**, choose `Grand Meridian Hotels` in **Values**, click **Share**, then **Launch**.

Two tasks appear in :menu[Data > Processing > Tasks]: one adds the label and the marking to every matching indicator, the other shares them, so the analysts of Grand Meridian Hotels see them in their own lists. Sharing needs an Enterprise Edition license.
:::

:::warning
A query task acts on what the filters match when it runs, not on what you saw. Check the filters before you launch a deletion with every element selected.
:::

In a [draft](/opencti/use/ingest/drafts/), bulk actions run inside the draft and appear in its **Draft processes**, not in the task list.

## Follow a task

:::steps

1. Open :menu[Data > Processing > Tasks]. The most recent tasks come first.
2. Read the task card: the **Targeted entities** and how they were chosen (the filters, the search or the **List of entities**), the **Actions**, the **Initiator**, the **Task start time** and the **Scope**. Once the task is done, **Task end time** gives when it finished; until then, **Task last execution time** gives when it last processed a batch. A rule task reads **All rule targets** and **APPLY RULE**: it covers the whole knowledge base.
3. Check the **Status**: **Waiting**, **In provisioning** while OpenCTI collects the objects, **Processing**, then **Complete**. The **Progress** bar counts the objects processed.
4. If the task reports errors, click the **errors** button: each line gives the **Timestamp** of a failure and its **Message**.

   ::screenshot{id=opencti-background-tasks-list alt="The Tasks list shows a completed task that added a label to twelve reports, with its initiator, start and end times, the KNOWLEDGE scope and a full progress bar."}

:::

To stop a task that has not finished, click **Delete** on its card. The objects already processed keep their changes.

## Who sees and deletes tasks

:::only{opencti=">=7.260811"}
- You see the tasks you started. Users with the **Bypass all capabilities** capability see every task.
- You can delete every list or query task you see. Deleting a rule task takes **Delete knowledge**.
:::

:::only{opencti="<7.260811"}
- You see the tasks you started, the tasks of others whose scope you have the capability for, as listed in [Before you start](#before-you-start), and the tasks open to everyone, such as rule tasks and file imports. Users with the **Bypass all capabilities** capability see every task.
- You can delete every list or query task you see. Deleting a rule task takes **Delete knowledge**.
:::

If :menu[Data > Processing > Tasks] shows a message instead of the list, the task manager is turned off on your platform: an administrator enables it with `task_scheduler:enabled` in the [configuration](/reference/opencti/configuration/).

:::example[Turn the task manager back on]
The platform runs with Docker Compose, and the Tasks screen says the task manager is disabled. The administrator sets its environment variable on the `opencti` service and restarts it:

```yaml
services:
  opencti:
    environment:
      - TASK_SCHEDULER__ENABLED=true
```

After the restart, the waiting tasks start, and the list shows their progress again.
:::

## What's next

- [Automate with playbooks](/opencti/use/automate/playbooks/): apply the same changes automatically to every new object.
- [Delete and restore](/opencti/use/delete-restore/): get back objects deleted by mistake.
- [Inference rules](/opencti/configure/rules/): the rule tasks behind the rules engine.
