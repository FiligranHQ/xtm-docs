---
title: Repair data after an upgrade
description: The background operations OpenCTI runs to repair data consistency after an upgrade, how they are scheduled, and how to estimate, run or stop them.
sidebar_position: 8
doc_type: reference
products:
  - opencti
personas: [platform-engineer, admin]
owner: docs-opencti
no_screenshot: "The data sanity manager runs in the background: it is followed and controlled through the GraphQL API and the logs."
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/dataSanityManager/
---

:::only{opencti="<7.260811"}
The data sanity manager arrived in OpenCTI 7.260811. The version you are reading does not have it.
:::

::::only{opencti=">=7.260811"}
The data sanity manager is a background process that repairs data when a new version of OpenCTI changes how data is computed, for example how two entities are recognized as the same. After an upgrade, it is why some entities get merged without anyone asking. This page lists its operations, how they run, what they write to the logs, and the API calls that let you estimate, schedule or stop them.

## How operations run

::diagram{id=opencti-data-sanity-cycle}

- The manager runs on the platform nodes every 10 minutes by default, one node at a time: a lock keeps two nodes from running it together.
- Every operation is of type `run_once`: the manager runs it once, at its first cycle after the upgrade that brings it. It logs the estimated impact first, then applies the changes. It does not run it again unless you schedule a new run.
- An operation still marked as running after 24 hours, for example because the node restarted, is considered stale and runs again at the next cycle.
- A failure is recorded with its message, and the next operations still run.
- An operation whose result cannot be recorded stays marked as running: later cycles skip it until it turns stale. Stop it to release it sooner.

| Key (environment variable) | Default | Effect |
| --- | --- | --- |
| `data_sanity_manager:enabled` (`DATA_SANITY_MANAGER__ENABLED`) | `true` | Turns the manager off. The operations an upgrade brings then wait until it is on again |
| `data_sanity_manager:interval` (`DATA_SANITY_MANAGER__INTERVAL`) | `600000` | Time between two cycles, in milliseconds. A scheduled run waits up to this long |
| `data_sanity_manager:stale_running_threshold` (`DATA_SANITY_MANAGER__STALE_RUNNING_THRESHOLD`) | `86400000` (not in `default.json`) | Time, in milliseconds, after which a run still marked as running is considered stale. An invalid value falls back to the default with a warning |

The [configuration reference](/reference/opencti/configuration/#engines-schedules-and-managers) lists them with the other managers.

## Operations

### Case sensitive Duplicated ID

| | |
| --- | --- |
| Identifier | `caseSensitiveDuplicatedId` |
| Type | `run_once` |
| Entity types | Attack Pattern, Course of Action |

OpenCTI used to tell entities apart by case, so `Phishing` and `phishing` could exist as two attack patterns. The identifier is now computed without case, so these entities are duplicates. The operation recomputes the identifier of every Attack Pattern and Course of Action, groups the entities that now share one, and merges each group into a single entity.

The entity kept is the one with the most relationships, then the oldest, so the fewest relationships move. Relationships, markings, labels and STIX identifiers of the others are kept on it. A group that fails to merge is logged and skipped; the rest of the run goes on.

## Follow and control operations

### With the GraphQL API

The same actions exist in the GraphQL API, as the [GraphQL schema reference](/reference/opencti/graphql-schema/) lists them, with a token of a user who has **Bypass all capabilities**. Every call takes the identifier of the operation.

| Call | Kind | Does |
| --- | --- | --- |
| `dataSanityOperations` | Query | Lists the operations with their state and last run |
| `dataSanityOperationDryRun(operation_name)` | Query | Returns `estimated_impact`, a count per entity type, without changing data |
| `dataSanityOperationRequestRun(operation_name)` | Mutation | Schedules a run for the next cycle |
| `dataSanityOperationStop(operation_name)` | Mutation | Marks the operation as done and cancels a scheduled run. It does not interrupt a run in progress |

Estimate before you schedule: the dry run counts what a run would change, per entity type, and changes nothing. Stop an operation that a node restart left running when you do not want it to start again by itself once it turns stale, for example during a migration window; schedule it again with `dataSanityOperationRequestRun` when you are ready. `dataSanityOperations` then returns `last_run_success: false` and `last_run_message: "Operation stopped manually"` for it.

For example, to estimate the impact of the duplicate merge:

```graphql
query {
  dataSanityOperationDryRun(operation_name: "caseSensitiveDuplicatedId") {
    estimated_impact {
      key
      count
    }
  }
}
```

## Follow operations in the logs

Every line of the manager in the platform logs starts with `[DATA_SANITY_MANAGER]`. The ones to watch after an upgrade:

| Message | Means |
| --- | --- |
| `Estimating run_once impact before actual run` | The estimate, logged with its `result` before any change |
| `Executing data sanity operation` | The run starts, with the `operation` identifier |
| `Data sanity operation completed successfully` | The run ended, with `executionTimeMs` |
| `Operation marked as running but considered stale` (warning) | A run left running for longer than `data_sanity_manager:stale_running_threshold` is allowed to run again |

:::only{opencti=">=7.261030"}
| Message | Means |
| --- | --- |
| `Data sanity operation failed` (warning) | The run stopped on an error, given in `cause` |
| `Data sanity operation result could not be recorded` (error) | The run ended but its state was not saved: the operation stays marked as running |
:::

:::only{opencti="<7.261030"}
| Message | Means |
| --- | --- |
| `Data sanity operation failed` (error) | The run stopped on an error, given in `error`, or its state could not be saved. When the failure itself cannot be saved, the operation stays marked as running |
:::

A run you schedule with `dataSanityOperationRequestRun` logs the same messages, the estimate included.

The merge operation also logs its progress per entity type: the number of entities evaluated, the number of groups to merge, and each merge, so that you can follow a long run on a large platform.

## What's next

- [Configuration reference](/reference/opencti/configuration/) for every `data_sanity_manager` key.
- [GraphQL schema](/reference/opencti/graphql-schema/) for the types `DataSanityOperation` and `DataSanityExecution`.
::::
