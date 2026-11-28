---
title: Monitor and troubleshoot playbooks
description: Check that the OpenCTI playbooks of your team run, read their
  execution traces, and fix the most common reasons a playbook stops early or
  does not change data.
sidebar_position: 3
doc_type: how-to
products:
  - opencti
edition: ee
personas: [cti-lead, cti-analyst, admin]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/playbook-creation/#monitor-playbook-activity
---

Once your team relies on a playbook, you need to know that it runs, and why when a report was not triaged as expected. The playbook list shows which playbooks run and how much work waits for them; the execution traces show what a playbook did on each run, step by step, with the data each step produced.

## Before you start

- Your platform has an Enterprise Edition license.
- Your role has the **Manage Playbooks** capability.

## Check that your playbooks run

Open :menu[Data > Processing > Automation]. For each playbook, the list shows:

- **Playbook running**: **Yes** while the playbook is started. A stopped playbook ignores every event, and does not catch up on them when you start it again.
- **Messages**: the number of bundles waiting in the playbook's queue, mostly what **Send for ingestion** handed to the platform's workers. It goes back to zero within seconds on a quiet platform. A number that keeps growing means the workers do not keep up: tell your administrator, and check that the playbook's filters are not wider than needed.

::screenshot{id=opencti-playbooks-list alt="The playbook list, with the Messages column at zero for every playbook and the Playbook running column showing Yes for the manual triage playbook and No for the others."}

Inside a playbook, the header repeats both: **Playbook is running** or **Playbook is stopped**, and the number of **messages in queue**.

## Read the execution traces

:::steps

1. Open the playbook in :menu[Data > Processing > Automation].
2. Click **Open last execution traces**, the icon next to the message counter, at the top right. The **Last execution traces** panel lists the runs, the newest first, each with its **Execution at** time and the number of **steps executed**.
3. Click a run to see its steps. A green icon marks a step that succeeded, a red icon a step that failed; each line gives its message and the time the step ended.
4. Click a step to open its **Raw data**: the bundle the step produced, or the error when it failed.

:::

The panel keeps the last 20 runs of the playbook, for up to 90 days.

::screenshot{id=opencti-playbook-traces alt="The Last execution traces panel with one run expanded: three steps, from the manual entry point to Send for ingestion, each marked as successfully executed."}

## Fix common problems

| What you see | Likely cause | What to do |
| --- | --- | --- |
| A red step, with a message such as *Enrich through connector fail execution* | The step failed and the run stopped there | Open its **Raw data** to read the error. An enrichment connector fails on elements outside its scope: filter them out before it with **Reduce knowledge** |
| The run stops before the end, every step green | The step took an output with nothing attached: `no-match` of **Match knowledge**, `unmodified` of **Manipulate knowledge** or **Transform with AI agent** | Expected when the data did not match. If it should have matched, open the step's **Raw data** and compare it with the filters or actions |
| The run stops at an **Apply predefined rule** step, every step green | A rule only works on the main element, and leaves through `unmodified` when the main element is not of the type it handles, for example a container rule on an indicator, or when it found nothing to add | Put the rule right after an entry point that starts on the right entity type. To apply a rule to the objects a first rule added, do it in a second playbook, linked by a label |
| **Manage access restrictions** fails with *It should have at least one valid member with admin access* | None of the members it sets has the **can manage** access right, or none of the users behind them has the **Manage authorized members** capability | Give **can manage** to a group or a user who keeps managing the container, in addition to the dynamic members such as the assignees |
| The data does not change | Missing **Send for ingestion**, a wrong action, or a scope that excludes the elements | Check that the branch ends with **Send for ingestion**. In **Manipulate knowledge**, check the **Action type**, the **Field**, the value and the scope |
| Not sure which branch the bundle took | Several outputs look alike in the trace | Name each component after what it does, or temporarily add a **Manipulate knowledge** step on each branch that adds a distinctive label |
| The playbook never runs | It is stopped, or the entry point filters match nothing | Check that the header says **Playbook is running**, then loosen the filters |
| The playbook runs again and again on the same objects | Two playbooks change data the other listens to | Narrow the filters of one of them, for example on a label the other does not set. A playbook never reacts to its own changes |

::::example[Find why hotel reports keep the NEW status]
The CTI team receives an alert for every new `hotel-sector` report from the playbook `Triage new hospitality reports`, but the reports keep the `NEW` status and have no assignee.

1. Open the playbook and click **Open last execution traces**. Pick the run of the last report, *Night-time exports from a hotel booking platform*.
2. The run lists three steps, all green: `Listen to new reports`, `Mark as in progress` and `Notify the CTI team`. Click `Mark as in progress`: its **Raw data** holds the report with the `IN_PROGRESS` status and Ana Duval as assignee, so the change was made.
3. No **Send for ingestion** step follows: the change was never written. On the canvas, `Mark as in progress` leads only to `Notify the CTI team`.
4. Use **Add a sibling node** on `Notify the CTI team` and choose **Send for ingestion**.

   :::only{opencti=">=7.261119"}
   The playbook is running, so it opened in view only: turn on **Edit mode** in its header first. **Add a sibling node** is in the menu of the step, behind its three dots.
   :::

The next report shows four steps in its run, and opens with the `IN_PROGRESS` status and Ana Duval as assignee.
::::

To see the full bundle on the server side, an administrator can add a temporary **Log data in standard output** step; it writes the bundle to the platform logs at the **Log level** you choose.

## What's next

- [Create a playbook](/opencti/use/automate/playbooks/create/) to review the build steps.
- [Playbook components reference](/reference/opencti/playbook-components/) for the outputs of each component.
