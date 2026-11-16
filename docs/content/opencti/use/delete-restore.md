---
title: Delete and restore knowledge
description: Delete entities and relationships in OpenCTI, find them in the trash, and restore them or delete them permanently before the retention period ends.
sidebar_position: 90
doc_type: how-to
products:
  - opencti
personas: [cti-analyst, admin]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/delete-restore/
---

An entity, relationship or sighting you delete in OpenCTI goes to the trash, where anyone with the **Delete knowledge** capability can restore it, with its relationships and files, for 7 days by default. After that, or once someone deletes it permanently, it is gone. This page shows how to delete, restore and purge, what the trash does not keep, and how administrators change the delay.

| You deleted | Can you get it back? | Who can restore it | For how long |
| --- | --- | --- | --- |
| An entity, a relationship or a sighting, one at a time or in bulk | Yes, from :menu[Data > Trash], with its relationships and files | Any user with **Delete knowledge** who can see its markings, not only the person who deleted it | 7 days by default, as the administrator sets |
| An inferred relationship | No need: the rules engine creates it again while its facts hold | | |
| A dashboard, an investigation, a setting or another platform object | No: it is deleted at once | | |
| Knowledge deleted by a connector, by platform synchronization or by a retention policy | No: it is deleted at once | | |

## Before you start

- Your role has the **Delete knowledge** capability. It also opens :menu[Data > Trash].
- The trash is on by default. When an administrator turns it off, every deletion is permanent and the **Trash** menu disappears.

## What deleting does

Deleting an entity also deletes every relationship and reference attached to it. The deletion is final for the rest of the platform: it is written to the stream, so playbooks react to it and synchronized platforms delete the object too. The trash only keeps a copy you can restore.

Not everything goes to the trash, as the table above shows. The **More information** button next to the **Trash** title recalls these rules on screen. Deleting a dashboard also deletes its public dashboards, with no way back.

## Delete knowledge

- **One object**: open it, open its actions menu (⋮) and choose **Delete**.
- **Many objects**: select them in any list and click the delete button in the toolbar. OpenCTI deletes them in a [background task](/opencti/use/automate/background-tasks/).

## Restore or purge from the trash

Each line of :menu[Data > Trash] is one delete operation: the main object with all the relationships and references deleted with it. A line shows the type and name of the main object, who deleted it, when, and its markings. A line inherits the markings of its main object, so you only see what you could see before the deletion.

:::steps

1. Open :menu[Data > Trash]. Search, filter or sort to find the object.
2. Open the menu at the end of its line and choose **Restore**, or **Delete permanently**.

   ::screenshot{id=opencti-delete-restore-trash alt="The Trash list with the type, representation, deleted by, deletion date and marking columns, and the menu of one line open on Restore and Delete permanently."}

3. Confirm. The dialog says how many relationships and references come back or go away with the main object.

:::

To act on many lines, select them with their checkboxes and use the toolbar: OpenCTI restores or purges them in a background task.

:::example[Bring back an IP address deleted by mistake]
An analyst deleted the IPv4 address `203.0.113.45`, taking it for a typo. Two days later, a partner confirms that it is a Velvet Harbor server. In :menu[Data > Trash], add two filters:

- **Deleted entity type**, operator **Equals**, value **IPv4 address**
- **Deletion date**, operator **Within**, **From** `now-7d` **To** `now`

The line of `203.0.113.45` shows who deleted it and when. Open its menu, choose **Restore** and confirm: the address comes back with its relationships and files, and the reports that contained it show it again. If the restore fails because an object at the other end of a relationship is in the trash too, restore that object first.
:::

**Restore** creates the object again with the information it had, its relationships and its files. **Delete permanently** removes the object, its relationships and its files. It cannot be undone.

## When a restore fails

A restore brings back the object and all its relationships in one pass, or nothing. It fails when:

| The message says | What to do |
| --- | --- |
| A relationship targets deleted elements | The object at the other end is in the trash too. Restore it first, then retry |
| A target element has been permanently deleted | The other end is gone for good. The object cannot be restored |
| A duplicate entity or relationship exists | The same object was created again since the deletion. Merge or update that one instead |

Restoring one object never restores the other objects it was linked to: restore each of them yourself, in order. Avoid restoring an object with thousands of relationships, which puts the platform under heavy load.

## Change how long the trash keeps objects

This section is for administrators. The garbage collection manager deletes objects permanently 7 days after their deletion. Three settings shape the trash:

| Key (environment variable) | Default | Effect |
| --- | --- | --- |
| `garbage_collection_manager:deleted_retention_days` (`GARBAGE_COLLECTION_MANAGER__DELETED_RETENTION_DAYS`) | `7` | Days an object stays in the trash before it is purged |
| `app:trash:enabled` (`APP__TRASH__ENABLED`) | `true` | `false` turns the trash off: every deletion is permanent, and the purge stops too |
| `garbage_collection_manager:enabled` (`GARBAGE_COLLECTION_MANAGER__ENABLED`) | `true` | `false` stops the purge: the trash keeps everything until someone deletes it permanently |

See the [configuration reference](/reference/opencti/configuration/#engines-schedules-and-managers).

:::example[Keep deleted knowledge for 30 days]
Your team reviews deletions once a month, and seven days are too short to catch a mistake. Set the delay to 30 days in the environment of the platform container:

```yaml
services:
  opencti:
    environment:
      - GARBAGE_COLLECTION_MANAGER__DELETED_RETENTION_DAYS=30
```

Or in `config/production.json`:

```json
{
  "garbage_collection_manager": {
    "deleted_retention_days": 30
  }
}
```

After a restart, the trash manager purges an object 30 days after its deletion. The trash grows accordingly: it still holds every object deleted in the last month, with its relationships.
:::

:::warning
The trash protects against mistakes, not against data loss. Keep backing up your databases.
:::

## What's next

- [Change many objects at once](/opencti/use/automate/background-tasks/), to follow bulk deletions and restores.
- [Merge duplicates](/opencti/use/enrich/merge/), the better choice when an object is a duplicate.
- [Delete old data automatically](/opencti/configure/retention/), to delete old knowledge automatically.
