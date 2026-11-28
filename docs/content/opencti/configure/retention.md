---
title: Delete old data automatically
description: Delete old OpenCTI knowledge, import files, workbenches and logs automatically with retention policies, and check what a policy will delete first.
sidebar_position: 4
doc_type: how-to
products:
  - opencti
personas: [admin, platform-engineer]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/administration/retentions/
---

A retention policy deletes data that has not changed for a given time: old knowledge that matches your filters, imported files, workbenches, and, in recent versions, history and activity logs. It keeps the platform lean and enforces your data retention rules. This page shows how to set up a policy safely.

:::warning
Retention deletes for good. What a policy deletes does not go to the [trash](/opencti/use/delete-restore/) and cannot be restored. Always click **Verify** to see how many elements a policy would delete before you save it.
:::

## Before you start

- Your role has the **Manage customization** capability.
- The retention manager is running. It is on by default (`retention_manager:enabled` in the [configuration reference](/reference/opencti/configuration/)); when it is off, the page says so and no policy runs.

## What a policy can delete

The **Scope** of a policy says what it deletes. The **Apply on** column of the list shows the filters of a knowledge policy, or what a policy without filters covers: **Everything** for a knowledge policy, **Global files** (the **Files contained in** Data/Import), **All workbenches**, **Knowledge history logs** or **Activity logs**.

:::warning
A knowledge policy that reads **Everything** has no filters: it deletes every entity and relationship of the platform, reports included, that has not been updated for its maximum retention. Check the **Apply on** column of every knowledge policy.
:::

:::only{opencti=">=7.260811"}
| Scope | Deletes | Filters |
| --- | --- | --- |
| Knowledge | Entities and relationships that match the filters and have not been updated for the maximum retention | Yes |
| File | Files in :menu[Data > Import] uploaded longer ago than the maximum retention, once their upload and every import from them are complete | No |
| Workbench | Workbenches not updated for the maximum retention, global ones and those attached to an entity | No |
| History | History log entries of knowledge older than the maximum retention | No |
| Activity | Activity log entries, such as logins, logouts and security events, older than the maximum retention | No |

You create Knowledge policies, as many as you need. The other four scopes have one built-in policy each: **Global files retention**, **All workbenches retention**, **History retention** and **Activity retention**, set to 30 days and inactive. Edit them and activate the ones you want. They cannot be deleted.

Before you activate **Activity retention**, check how long your security policy requires you to keep logins and administration events: the policy deletes them from the platform for good. Forward them to your SIEM first if they must be kept longer.
:::

:::only{opencti="<7.260811"}
| Scope | Deletes | Filters |
| --- | --- | --- |
| Knowledge | Entities and relationships that match the filters and have not been updated for the maximum retention | Yes |
| File | Files in :menu[Data > Import] uploaded longer ago than the maximum retention, once their upload and every import from them are complete | No |
| Workbench | Workbenches not updated for the maximum retention, global ones and those attached to an entity | No |

Two policies come with the platform and run from the start: **Global files retention** and **All workbenches retention**, both set to 30 days. Change their duration if you keep imported files or workbenches longer.
:::

The retention manager never deletes an individual that stands for a user, the platform main organization, or an organization that still has members or administrators: such objects are skipped and drop out of the **Remaining** count.

## Create a knowledge policy

:::steps

1. Open :menu[Settings > Customization > Retention policies] and click **Create Retention Policy**. The **Create a retention policy** panel opens.
2. Give the policy a **Name** that says what it deletes, for example "Indicators older than a year".
3. Choose the **Unit** (minutes, hours or days) and the **Maximum retention**: an object not updated for that long is deleted. The default is 31 days.
4. Add filters to target the objects to delete, for example an entity type and a label. Without filters, the policy applies to every entity and relationship of the platform and reads **Everything** in the list.
5. Click **Verify**. A message tells you how many elements the policy would delete now.
6. Adjust the filters until the count is what you expect, then click **Create**. **Create** stays disabled until you verify.

   ::screenshot{id=opencti-retention-create alt="The Create a retention policy panel with its Name, Unit, Maximum retention set to 31 and filters, and the Create button disabled next to Verify until the policy has been verified."}

:::

::screenshot{id=opencti-retention-list alt="The Retention policies list showing a knowledge policy and the four built-in policies, with their maximum retention, last execution, remaining count, scope and status."}

:::example[Purge scanner indicators after a year]
The Example Scanner Feed creates thousands of indicators a week, all labelled `scanner`. Decay revokes them within days, but revoked indicators still fill the search engine. Example CERT keeps them one year for investigations, then deletes them. Every other indicator, and every report, stays.

- **Name**: `Scanner indicators older than a year`
- **Unit**: `days`
- **Maximum retention**: `365`
- Filters, combined with AND:

```text
Entity type = Indicator
AND Label = scanner
```

Click **Verify** before **Create**: the message counts the scanner indicators not updated for a year. Compare it with the number of indicators labelled `scanner` in :menu[Observations > Indicators]: a count above it means a filter is missing. Once created, the policy deletes in batches at each run, and the **Remaining** column falls to 0. An indicator that the feed sends again is updated, which restarts its year.
:::

:::only{opencti=">=7.260811"}
A policy you create is active as soon as you save it: deletion starts at the next run of the retention manager. To prepare a policy without running it, deactivate it right after creating it, from the menu of its row.
:::

## Activate, pause or change a policy

Open the menu of a policy in the list:

- **Update** changes its name, duration and filters. Click **Verify** again before **Update**, which stays disabled until you do.

:::only{opencti=">=7.260811"}
- **Deactivate** pauses a policy without deleting it, for example during a migration or after an unexpected deletion; **Activate** resumes it. The **Status** column shows which policies run. The update panel has the same **Active** switch.
- **Delete** removes a knowledge policy. Built-in policies cannot be deleted; deactivate them instead.
:::

:::only{opencti="<7.260811"}
- **Delete** removes the policy.
:::

The list shows, for each policy, the **Last execution** and the **Remaining** count of elements still to delete. The retention manager deletes in batches, so a policy that matches many elements catches up over several runs.

:::only{opencti=">=7.261127"}
### Find out why a policy does not delete

When a run fails to delete an element, a red error icon appears next to the **Remaining** count of the policy. Hover the icon, or reach it with the keyboard, to read the **Last error**: the identifier of the element, then the error message. The element stays in the **Remaining** count, and the policy tries it again at the next run. The next run of the policy without an error removes the icon. A paused policy does not run, so its icon stays.

The platform logs give the same information at the `warn` level: one `Retention manager error` line per element that failed, then `Retention manager rule executed with errors` with the number of elements deleted and failed. Each run also logs `Retention manager executing rule` at the `info` level, with the number of elements the policy matches.

Each run that deletes something, or fails to, is recorded in the [activity log](/opencti/configure/activity/events/#audit-events), whatever the scope. The message gives the policy name, the number of entries deleted and, when some failed, the number that failed. For a knowledge policy, the event also counts the deleted elements per entity type.
:::

:::only{opencti=">=7.260811 <7.261127"}
Deletions of history and activity entries are themselves recorded in the activity log, with the number of entries deleted.
:::

## Tune the retention manager

The retention manager runs on one node of a cluster at a time. At each run, it takes every active policy in turn and deletes up to one batch of what the policy matches.

| Key | Environment variable | Default | Effect |
| --- | --- | --- | --- |
| `retention_manager:enabled` | `RETENTION_MANAGER__ENABLED` | `true` | Turns the retention manager on or off. Off, no policy runs |
| `retention_manager:interval` | `RETENTION_MANAGER__INTERVAL` | `30000` | Pause between the end of a run and the start of the next, in milliseconds |
| `retention_manager:batch_size` | `RETENTION_MANAGER__BATCH_SIZE` | `1500` | Maximum number of elements one policy deletes in one run |
| `retention_manager:max_deletion_concurrency` | `RETENTION_MANAGER__MAX_DELETION_CONCURRENCY` | `2` | Number of elements deleted in parallel |

A new policy on a large knowledge base can match millions of elements: with the defaults, it deletes at most 1,500 elements per run, with a 30-second pause between runs, so fewer than 180,000 an hour. Raise `batch_size` or `max_deletion_concurrency` to catch up faster, at the cost of more load on the search engine. The **Remaining** column shows how much is left.

## What's next

- [Delete and restore](/opencti/use/delete-restore/), for deletions you can undo.
- [Audit who did what](/opencti/configure/activity/), what the activity logs record.
- [Engines, Schedules and Managers](/reference/opencti/configuration/#engines-schedules-and-managers) in the configuration reference, for every `retention_manager` key of your version.
