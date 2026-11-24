---
title: Follow the background jobs and run one again
description: Follow the runs, the queue, the health and the logs of OpenCRQ's background jobs in a tenant, find why one failed, and start a run again.
sidebar_position: 4
doc_type: how-to
products:
  - opencrq
personas: [admin, platform-engineer]
owner: docs-opencrq
last_reviewed: 2026-11-20
---

Most of what OpenCRQ shows is computed in the background: the price of each risk scenario, the score of each finding, the threats OpenCTI streams in, the imports of each connector. :menu[Settings > Jobs] shows every run of these jobs in a tenant, what waits in the queue, the health and logs of each kind of job, and when each one is due. This page reads them to answer the usual questions: did the figures take the last change into account, why did an import fail, and how do I run it again. What each kind of job does is in [OpenCRQ background processes](/reference/opencrq/processes/#job-kinds).

## Before you start

- Your role has **Manage Security** in the tenant, or your platform role has **Platform Administration Access**. XTM One's administrator, who signs in to OpenCRQ as its super administrator, holds both. See [Give people access with users, groups and roles](/opencrq/administer/users-and-roles/).
- Jobs belong to a tenant: :menu[Settings > Jobs] shows the runs of the tenant you are in.

## How a run comes about

A run is booked for one kind of job in one tenant. Its **Triggered by** says why:

| Triggered by | When |
| --- | --- |
| **Data change** | Something the kind reads has changed: a scenario saved, a control posture declared, a threat streamed in. The changes of a burst fold into one run |
| **Scheduler** | The daily run of the platform's recomputes, or a connector's sync on its schedule |
| **User** | A person asked for it: **Run** on a risk scenario, **Sync now** on a connector |
| **Service account** | An integration asked for it, as its service account |
| **System** | OpenCRQ asked for it itself |

A worker takes the run from the queue. When an attempt fails, the run waits and tries again, up to its number of attempts (**1 / 3** reads first attempt of three). When the last one fails, the run lands in the **Dead-letter queue** and stays there. An error another attempt cannot fix sends it there at once, whatever attempts remain, such as a connector not configured, credentials the remote service refuses, or a file that is not a valid STIX bundle. Some kinds run one at a time per tenant: a second run waits until the first ends.

The price of a scenario (`crq-simulation`) can start before the weights of the attackers of its outcome are computed. It then goes back to the queue without an error and without spending an attempt, and tries again from 2 to 30 seconds later. It waits so for at most `recompute.debounce_cap_ms` plus five minutes from its first start, 10 minutes by default; past that, it lands in the dead-letter queue.

## Follow the runs

:::steps

1. Open :menu[Settings > Jobs]. The **Runs** tab lists every run of the tenant, the newest first, with its **Kind**, **Status**, **Triggered by**, what it is **Related to**, its **Parameters**, when it was **Queued at** and **Finished at**, and its **Attempts**.

   ::screenshot{id=opencrq-background-jobs-runs alt="The Runs tab of the Grand Meridian Hotels tenant on 12 April 2026, all Succeeded: opencti-mirror-rebuild, finding-score-recompute, crq-price-recompute and threat-list-projection-recompute triggered by Data change, then daily runs such as jobs-retention and attack-outcome-mixture-recompute triggered by the Scheduler with their date as parameter."}

2. Narrow the list with the filters above it: **Status**, a date window on **Queued at**, **Started at** or **Finished at**, **Kind**, **Triggered by**, or a search.
3. Click a run to open it. Its page shows its **Identity** (kind, status, attempt, priority), its **Chronology** (when it was queued, started, claimed and finished, how long it **Waited** and **Ran for**), its **Trigger**, its **Parameters**, and its **Errors** and **Logs**.

:::

The **Status** of a run is **Queued**, **Running**, **Succeeded**, **Cancelled** or **Dead-letter queue**. An info mark next to it, **Absorbed an ask**, means that another request arrived while the run was working: a follow-up run answers it.

## Read the queue

The **Queue** tab shows the tenant's runs in four lists:

- **Pending**: the runs waiting, in the order the workers take them, with the time they may **Run after** and their **Claimability**. **Next** is taken first; **Waiting for schedule** waits for its time; **Blocked by sibling** waits for a run of the same kind to end.
- **Running**: the runs a worker has **Claimed at** a time.
- **Failed** and **Succeeded**: the runs that ended, with when they **Finished at**.

A run that stays in **Pending** with **Next** means no worker takes it. The **Integrations** page then says "No worker is answering. Queued runs will not advance.": the worker process is down or has jobs turned off (`jobs.enabled` and the `process_manager` profile in the [configuration](/reference/opencrq/configuration/#jobs)).

## Check the health of each kind

The **Kinds** tab lists every kind of job with its **Health**, its **Last run**, its **Duration** and its **Attempts**:

| Health | Meaning |
| --- | --- |
| **Healthy** | The last run went through, or was cancelled on purpose |
| **Failing** | The last run landed in the dead-letter queue |
| **Running** or **Queued** | A run is under way, or booked and waiting for a worker |
| **Never ran** | No run was ever recorded for this kind in this tenant, such as the sync of a connector not configured |

When OpenCRQ is registered with XTM One (the `xtm_one` keys of the [configuration](/reference/opencrq/configuration/#xtm-one)), its registration, every 5 minutes, also sends XTM One one health per kind for the whole platform: the worst that any tenant reads. A kind whose last run landed in the dead-letter queue in one tenant is sent as failing, whatever the other tenants read.

Click a kind to open it. **Observed** marks a kind whose reads are recorded, so that a change of that data books it again; **Daily** marks a kind also booked once a day. Its **Overview** tab shows its last run, and for a connector's sync, a link to the connector: **Open the connector**. Its **Runs** tab lists its runs in progress and its most recent completed ones. Its **Logs** tab shows the lines its last runs wrote, filtered by text or by **Levels** (**trace**, **error**).

::screenshot{id=opencrq-background-jobs-logs alt="The Logs tab of the daily jobs-retention job, Healthy: its runs of 12 and 10 April 2026, both Succeeded, each with the three lines Attempt 1 of 3 started, Job runs purged and Job completed successfully, and the note that older runs are not kept."}

## See when each job is due

The **Schedules** tab lists the kinds that run on a schedule, with their **Origin**, their **Schedule**, their **Next run** and their **Last run**.

- **Platform**: the daily recomputes, at `recompute.daily_at` (03:00 UTC by default). Each tenant starts at its own minute within the hour that follows. It is set in the deployment configuration, not here.
- A connector (**OpenAEV**, **Microsoft Defender**, **Drata**, **NIST NVD CVE**, **FIRST EPSS**, **CISA Known Exploited Vulnerabilities (KEV)**, **OpenCRQ Default Threats**): its sync, every hour, every 6 hours or every day at 06:00 UTC by default. Change it with **Frequency**, **Day**, **Time** and **Time zone** in the connector's **Configure integration** panel. A connector without a configuration has no **Next run**. **OpenCRQ Default Threats** needs no configuration. A tenant created from the platform, and the tenant OpenCRQ creates at its first boot, start with it switched off: the scheduler skips it until someone with **Manage Security** clicks **Enable** in the **Threats** step of **Get started**, or turns on the **Enabled** switch of its **Configure integration** panel. Switching it on books its `default-threats-sync` at once, triggered by **User**. A tenant where it is on but none of its threats is loaded yet also gets that run at once from the scheduler, instead of waiting for its schedule. Switched off, it has no **Next run**.

## Run a job again

The jobs pages only read. You start a run again where its work is:

| To run again | Do this |
| --- | --- |
| A connector's sync | Open **Integrations**, open the connector, click **Sync now**. While a run is under way, **Cancel** stops it |
| A connector's failed sync | On the connector's page, click **Retry**: its last run, which landed in the dead-letter queue, gets another attempt |
| The price of a risk scenario | Click **Run** on the scenario. See [Build and price a risk scenario](/opencrq/use/quantify/risk-scenarios/) |
| A recompute (scores, prices, threats, reach) | Nothing to do: it runs again at the next change of the data it reads, and every day for the **Daily** kinds |

**Cancel** and **Retry** are for those who hold **Manage Security** in the tenant or **Platform Administration Access**. A member without them sees these buttons only on a run they started themselves with **Sync now**. Once another request has joined that run, its **Cancel** stays greyed out with the hint "Other requests are waiting on this run, so you can no longer cancel it.": an administrator can still cancel it.

## How long runs and logs are kept

| What | Kept | Setting |
| --- | --- | --- |
| Finished runs, with their errors and logs | 30 days, deleted each day by the `jobs-retention` job, except the last run of each kind, kept whatever its age | `jobs.run_retention_days` |
| Log lines of a kind | Those of its last 3 runs, per tenant | `jobs.log_retention_runs` |
| Log lines of the OpenCTI stream | The last 500 | `jobs.log_retention_lines` |

A run whose lines are no longer kept shows "Older runs are not kept." at the end of its logs. To keep 90 days of runs and the lines of the last 10 runs of each kind, in the `environment` of the `opencrq` service:

```yaml
JOBS__RUN_RETENTION_DAYS: 90
JOBS__LOG_RETENTION_RUNS: 10
```

See the `jobs` keys of the [configuration](/reference/opencrq/configuration/#jobs).

:::example[Check that Ember Tide's return reached the prices]
On 10 April 2026, OpenCTI reports Ember Tide's new campaign against hotel chains. Claire Morel, cyber risk manager of Grand Meridian Hotels, checks that the ransomware scenario took it into account.

- In **Runs**, with **Triggered by** set to **Data change**, the runs of 10 April come in order: `opencti-mirror-rebuild` (the threat streamed in), `threat-activity-recompute`, `threat-list-projection-recompute` and `attack-outcome-mixture-recompute` (Ember Tide's activity, and its weight among the attackers of each scenario), then `finding-score-recompute` and `crq-price-recompute` (the scores and the prices). Each is **Succeeded** on attempt **1 / 3**.
- In **Kinds**, `opencti-mirror-rebuild` is **Healthy**, and its **Logs** show, for each run, "OpenCTI mirror rebuilt" and "Job completed successfully".

The price of the ransomware scenario is current. For its quarterly audit, Example CERT also wants three months of run history instead of 30 days: its platform engineer sets `jobs.run_retention_days` to `90` in the deployment configuration of OpenCRQ. The runs already deleted, those before 11 March, do not come back: from then on, the `jobs-retention` job keeps each run 90 days, so the history reaches three months in early June.
:::

## Check that it works

After a change, open **Runs**: the runs it booked appear as **Queued**, then **Running**, then **Succeeded**, from under a second to a few minutes later depending on how long the last run of each kind took (the `recompute.debounce_*` keys of the [configuration](/reference/opencrq/configuration/#recompute)). A kind whose **Health** stays **Failing** has its reason in the **Errors** of its last run.

## What's next

- [OpenCRQ background processes](/reference/opencrq/processes/), what each kind of job does and when it runs.
- [Let integrations sign in with service accounts](/opencrq/administer/service-accounts/), the accounts the connectors' syncs run as.
- [Configure OpenCRQ](/deploy/configure/opencrq/), for the worker and the retention settings.
