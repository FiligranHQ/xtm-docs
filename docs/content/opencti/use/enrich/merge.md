---
title: Merge duplicates
description: Merge duplicate OpenCTI entities of the same type into one, keep their relationships, and know which fields survive before you confirm.
sidebar_position: 3
doc_type: how-to
products:
  - opencti
personas: [cti-analyst, soc-analyst]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/merging/
  - opencti:/administration/merging/
---

Sources name the same threat in different ways, so the same intrusion set or malware can exist several times in OpenCTI, each copy holding part of what is known. Merging folds these duplicates into one entity that keeps all their relationships, takes the other names as aliases, and receives what any source sends later under the old names. This page shows how to merge, and what happens to each field before you confirm, since a merge cannot be undone.

## Before you start

- Your role has the **Merge knowledge** capability.
- The entities are of the same type. You can merge two to four at a time; for more, merge in several rounds.
- You are not working in a draft: merging is not available there.
- Indicators, notes, opinions, tasks and labels cannot be merged. Built-in vocabulary entries cannot be merged either.

:::warning
A merge cannot be undone: the other entities are deleted and their relationships move for good. Check the selection before you confirm.
:::

## Merge entities

:::steps

1. Open the list of the type, for example :menu[Threats > Intrusion sets], or :menu[Data > Entities] to work across types. A list shown as cards has no checkboxes: switch it to the lines view.
2. Select the two to four duplicates with their checkboxes. The selection survives a filter, so you can filter the list to find each duplicate in turn.
3. Click **Merge** in the toolbar. The button stays greyed out with fewer than two or more than four entities selected, with entities of different types, or after **Select all**.
4. In the **Merge entities** dialog, choose the main entity under **Selected entities**. Pick the one whose name and description you want to keep.

   ::screenshot{id=opencti-merge-dialog alt="The Merge entities dialog with two intrusion sets under Selected entities, the first one chosen as the main entity, and the Merged entity section previewing the name, aliases, author and marking of the result."}

5. Check the **Merged entity** section, which previews the name, aliases, author and markings of the result, then click **Merge**.

:::

:::example[Fold Velvet Harbour into Velvet Harbor]
A partner's feed reports the intrusion set as `Velvet Harbour`, with the alias `Harbour Mantis`, while your own reports use `Velvet Harbor`. Both now exist in :menu[Threats > Intrusion sets].

1. In the lines view of the list, check Velvet Harbor and Velvet Harbour, then click **Merge**.
2. Under **Selected entities**, choose Velvet Harbor as the main entity: its description and its author, Example CERT, are the ones to keep.
3. Check that **Merged entity** shows the name `Velvet Harbor` and the author Example CERT, then click **Merge**.

Velvet Harbour disappears from the list. Velvet Harbor gains `Velvet Harbour` and `Harbour Mantis` as aliases, the relationships and the `TLP:GREEN` marking of Velvet Harbour, and the next bundle of the partner that refers to Velvet Harbour lands on Velvet Harbor, since its STIX identifier was kept. `Velvet Harbor Group`, a third duplicate, can join the same merge or a second round.
:::

OpenCTI runs the merge in a [background task](/opencti/use/automate/background-tasks/). With about a hundred relationships per entity on a healthy platform, it takes a few seconds; large entities take longer.

## What the merged entity keeps

| Field | Result |
| --- | --- |
| Relationships | All relationships of all entities move to the merged entity |
| Name | The name of the main entity. The names of the others become aliases, on types that have aliases |
| Description and other single-value fields | The value of the main entity. When the main entity has none, the value of another entity fills the gap |
| Labels, markings, aliases, external references and other multi-value fields | The values of all entities, combined |
| STIX identifiers | The identifiers of the other entities are kept, so data that refers to them later lands on the merged entity |

A description written on a non-main entity is lost when the main entity has its own. Copy what you need to the main entity before merging.

## Merge or deduplicate

You rarely need to merge objects that come in with the same identity: OpenCTI recognizes them at creation and updates the existing object instead. Merge is for duplicates that deduplication cannot see, such as an intrusion set known under two names. See [Deduplication](/opencti/concepts/deduplication/).

## What's next

- [Deduplication](/opencti/concepts/deduplication/), to understand when OpenCTI merges data by itself.
- [Delete and restore knowledge](/opencti/use/delete-restore/), when an object is wrong rather than duplicated.
- [Change many objects at once](/opencti/use/automate/background-tasks/), to follow a merge in progress.
