---
title: Track the progress of reports and cases
description: Track the progress of reports, cases and other OpenCTI entities with statuses, and assign the people responsible for them.
sidebar_position: 5
doc_type: how-to
products:
  - opencti
personas: [cti-lead, cti-analyst, admin]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/workflows/
---

An entity workflow gives an entity type a list of statuses, such as `NEW`, `IN_PROGRESS`, `ANALYZED` and `CLOSED`, so your team sees where each report, case or other entity stands. Together with assignees and participants, it tells everyone who works on what. This page shows how to set up the statuses of an entity type, use them day to day, and get notified of what you are assigned.

For drafts, which follow a stricter workflow with transitions and actions, see [Approve drafts step by step](/opencti/use/automate/draft-workflow/).

## How statuses work

- Each status of an entity type reuses a **status template**, a name and a color defined once in :menu[Settings > Taxonomies > Status templates]. The same template can serve several entity types, so `IN_PROGRESS` means the same everywhere.
- The **Order** of the statuses sets the order in which users see them. Users can move an entity to any status of its type: the order is not enforced.
- An entity type has a workflow as soon as it has one status. Its entities then show a **Processing status** in their overview, and a **Status** field in their edition form.
- Reports come with four statuses on a new platform: `NEW`, `IN_PROGRESS`, `ANALYZED` and `CLOSED`. Other types have none until you add them.

Workflows exist for domain objects, such as reports, cases, malware or intrusion sets, and for relationships and sightings.

## Before you start

- Your role has the **Manage customization** capability to set up the statuses of an entity type.
- To create or rename status templates, your role has the **Manage status templates** capability.

## Set up the statuses of an entity type

:::steps

1. Check that the status templates you need exist in :menu[Settings > Taxonomies > Status templates], and create the missing ones with a name and a color.
2. Open :menu[Settings > Customization > Entity types] and choose the entity type, for example **Incident response**.
3. On the **Workflow** tab, click the edit button of the **Workflow** card. The **Workflow of** panel lists the current statuses.

   ::screenshot{id=opencti-entity-workflows-statuses alt="The Workflow of Report panel lists the NEW, IN_PROGRESS, ANALYZED and CLOSED statuses in order, with the button to create a status."}

4. Click the create button to open **Create a status**, choose a status template and set its **Order**. Repeat for each status.
5. To change or remove a status, open the menu on its line.

:::

:::example[Add a peer review step to reports]
The CTI team of Grand Meridian Hotels wants a second analyst to read every report before it is marked as analyzed.

1. In :menu[Settings > Taxonomies > Status templates], check that `IN_REVIEW` exists, or create it with the **Color** `#ff9800`.
2. In :menu[Settings > Customization > Entity types], choose **Report**, then edit the **Workflow** card.
3. In **Create a status**, choose `IN_REVIEW` and set **Order** to `3`.
4. Open the menu of `ANALYZED` and choose **Update**: set **Order** to `4`. Do the same for `CLOSED` with `5`.

Reports now show `NEW`, `IN_PROGRESS`, `IN_REVIEW`, `ANALYZED` and `CLOSED`, in that order. When Ana Duval finishes the report *Night-time exports from a hotel booking platform*, she sets its **Status** to `IN_REVIEW` and adds Leo Martin to its **Participants**. Leo Martin is notified, reads the report, and sets it to `ANALYZED`. The order guides users: nothing stops someone from moving a report from `IN_PROGRESS` straight to `ANALYZED`. For a step nobody can skip, use the [draft workflow](/opencti/use/automate/draft-workflow/).
:::

:::only{opencti=">=7.261030"}
The **Entity status sync** switch, in the **Workflow** card, decides what happens to the status of entities that arrive through a stream synchronization from another OpenCTI instance, for example from a partner CERT. On, OpenCTI gives them the local status of the same name, such as `IN_PROGRESS`; off, which is the default, it drops their status. The switch is set per entity type, and is grayed out for the types it does not apply to.
:::

Requests for information of the request access type follow their own workflow, with the statuses to apply when a request is approved or declined. See [Let users request access](/opencti/administer/request-access/).

## Use statuses and assignments

:::steps

1. Open an entity of a type that has a workflow and click **Update**.
2. Choose its new **Status**.
3. Set its **Assignees**, the people responsible for it, and its **Participants**, the people who contribute. These attributes exist on reports, cases, tasks and other entities that people work on.

:::

To follow the work of the team, filter lists by **Status**, **Assignees** or **Participants**, or build a [dashboard](/opencti/use/dashboards/) with the same filters.

Administrators can make **Assignees** or **Participants** mandatory, or give them a default value, in the **Attributes** tab of the entity type. See [Customize entity types](/opencti/configure/entities/).

## Get notified of assignments

OpenCTI notifies you, through the **Personal notifiers** of your profile, when you become an assignee or a participant of an object, and then of every change to that object, status changes included, as long as you stay on it. You need no trigger for that.

:::only{opencti=">=7.260811"}
An administrator can turn these automatic notifications off for the whole platform with **Automatically notify users when they are assigned as participant or assignee**, in :menu[Settings > Customization > Notifiers]. Users then need a live trigger on **Modification** with a filter on **Assignees** to be told.
:::

To follow the status of objects you are not assigned to, create a live trigger on **Modification** with a filter on **Status**, or subscribe to the entity. See [Get alerted when knowledge changes](/opencti/use/automate/notifications/).

## For administrators

Every change to an entity, a status change included, takes a lock on it, so that two users, or a user and a playbook, cannot change it at the same time. A change that cannot get the lock is retried, then fails. By default it is retried 100 times, every 250 milliseconds plus up to 100 milliseconds at random, so a change waits 30 to 35 seconds at most. These defaults suit almost every platform. Change them only when users report failed updates under heavy automation: the retry count, delay and jitter are the concurrency keys of [Technical customization](/reference/opencti/configuration/#technical-customization) in the configuration reference.

:::only{opencti=">=7.261030"}
The same section holds `workflow:disable_read_repair` (`WORKFLOW__DISABLE_READ_REPAIR`), `false` by default. It applies to the [draft workflow](/opencti/use/automate/draft-workflow/): when the status a draft shows differs from its position in the workflow, for example after an interrupted transition, OpenCTI corrects the status the next time someone opens the draft, at most once every five seconds per draft. With `true`, OpenCTI no longer corrects it: a draft whose status diverged keeps showing that status, and users may act on a stage the draft has left. Keep the default, and turn it off only for the time you examine a draft whose status keeps changing back.
:::

## What's next

- [Approve drafts step by step](/opencti/use/automate/draft-workflow/): approval steps for drafts.
- [Customize entity types](/opencti/configure/entities/): attributes, mandatory fields and default values.
- [Status templates](/opencti/configure/taxonomies/): the names and colors statuses reuse.
