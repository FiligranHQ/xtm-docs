---
title: Automate with playbooks
description: What OpenCTI playbooks do for a CTI team, how data flows through
  them, what to decide before building one, and when to use them rather than
  other automation.
sidebar_position: 1
doc_type: explanation
products:
  - opencti
edition: ee
personas: [cti-lead, cti-analyst, admin]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/playbook-automation/
---

A playbook does for your team what it would otherwise do by hand, the same way, every time. Every new report on the hospitality sector can be marked as in progress, assigned to the analyst on duty and announced to the CTI team, without anyone touching it. A playbook starts on an event, passes the data from step to step to filter, enrich or change it, and ends by saving the result, telling people, or handing it to an AI agent. Nobody writes code: you draw it on a canvas.

You find playbooks in :menu[Data > Processing > Automation]. They require an Enterprise Edition license. Building and running them takes the **Manage Playbooks** capability, and the buttons that create or import a playbook also need **Create / Update knowledge**; enrolling an entity in a running playbook takes **Use Playbooks**.

:::only{opencti=">=7.261119"}
A stopped playbook opens in **Edit mode** for users with **Manage Playbooks**. A running playbook opens in view only, so that nobody changes live automation by accident: turn on the **Edit mode** switch of its header to change it, knowing that each change applies at once. Users without **Manage Playbooks** see playbooks in view only, without the switch.
:::

## What playbooks do for a team

| Outcome for the team | Example |
| --- | --- |
| New intelligence is triaged before anyone opens it | Every new `hotel-sector` report gets the `IN_PROGRESS` status and an assignee |
| Incoming data is qualified the same way | Indicators from a trusted partner get a higher score, everything a feed creates gets a label |
| Enrichment happens only where it is worth it | New IP addresses of a Velvet Harbor report go to a reputation connector, the others do not |
| Data turns into work | Every critical incident opens an incident response case, with its tasks |
| Sharing follows your rules | Reports for a hotel chain are shared with its organization, cases are limited to their participants |
| The right people know | The duty analyst is notified, the assignees of a case get an email when it changes |
| Defenses are tested against new threats | A security coverage is created for every new Velvet Harbor report, and OpenAEV tests it |

## How a playbook works

Every playbook moves a **STIX 2.1 bundle**, a package of STIX objects, from one component to the next. The **main element** is the object that started the run, for example the report whose creation started it. An enrichment connector or a rule can add more objects to the bundle along the way.

::diagram{id=opencti-playbook-flow}

Five rules explain most of what you will see:

1. **One entry point.** A playbook starts in one of four ways: **Listen knowledge events** (as data is created, updated or deleted), **Listen PIR events** (as entities enter, change in or leave a priority intelligence requirement), **Query knowledge on a regular basis** (on a schedule), or **Available for manual enrollment / trigger** (when a user enrolls an entity).
2. **Outputs make branches.** A component sends the bundle to one of its outputs. **Match knowledge** uses `out` when something matched and `no-match` otherwise. A run that reaches an output with nothing attached stops there, without error. You can attach several branches to the same output.
3. **Scope decides what a step touches.** Components that change data act on the main element unless you widen their scope, as described under this list.
4. **Nothing is saved until Send for ingestion.** A branch that ends with a notifier or an email only informs. To save and notify, end one branch with **Send for ingestion** and another with the notifier.
5. **A playbook does not trigger itself.** It ignores the events its own changes cause, so it cannot loop on itself. It does react to changes made by another playbook: that is how you chain two playbooks, and how two playbooks with overlapping filters can feed each other endlessly.

:::only{opencti=">=7.261119"}
Every step but the entry point hangs from the output of another step. OpenCTI refuses a change, an import or a duplicate that would add a second entry point or leave a step without a parent.
:::

:::only{opencti=">=7.260811"}
The scope is the **Apply to** setting: **Only main element in the bundle** (the default), **All elements in the bundle**, or **All except main element in the bundle**. **Apply with filters** narrows it further, for example to the IP addresses of the bundle.
:::

:::only{opencti="<7.260811"}
The scope is a check box of each component, such as **Manipulate all elements included in the bundle**: unchecked, the step acts on the main element only.
:::

::screenshot{id=opencti-playbook-canvas alt="A playbook that listens for new reports, adds their content, then branches: reports about the hospitality sector are marked in progress, saved and sent to the CTI team; the others notify the duty analyst." caption="Each branch ends with Send for ingestion or a notification."}

:::only{opencti=">=7.261109"}
Playbooks work with [custom fields](/opencti/configure/taxonomies/#custom-fields) too: filter on them in entry points and in **Match knowledge**, and set them with **Manipulate knowledge**, whose **Field** list offers them by their label. **Replace** and **Remove** work on every custom field, **Add** only on a multiple selection list.
:::

:::only{opencti=">=7.261117"}
**Manipulate knowledge** also sets the **Report types** of reports: **Add** a type, **Replace** them all, or **Remove** one. A playbook can type every report a partner feed brings as `threat-report`, so that analysts filter on it. The action leaves the other entities of the bundle untouched.
:::

:::only{opencti=">=7.261119"}
Each entity type has its own workflow, so a **Status** set by **Manipulate knowledge** belongs to one type. You choose the **Entity type** first, then one of its statuses, and the action only changes the elements of that type: a report status is never written on the case of the same bundle. An action saved in an earlier version, without a type, keeps applying to every element in its scope.
:::

## What to decide before you build

A playbook changes data without asking anyone. Settle these points with your team, then [create the playbook](/opencti/use/automate/playbooks/create/).

| Decision | What it changes |
| --- | --- |
| What starts it | Listening to knowledge events acts within seconds of each event. A schedule acts in batches, for example a weekly grouping. Manual enrollment leaves the choice to an analyst, for one-off actions such as opening a case for a report |
| How narrow the filters are | Filters decide which data the playbook touches. Without them, a playbook that listens to knowledge events runs on every creation of the platform |
| What it may change | Status, assignees, labels, markings, score, sharing. A change is written for good once it reaches **Send for ingestion**. Test on your own data first, with a filter on **Creator** |
| Who is told, and how | A notifier alerts in the platform, by email or by webhook. Notify a group rather than a person, so the alert follows the duty rota |
| Who may build playbooks | A playbook reads and writes as the platform's automation identity, not as its author. It sees every object, including those restricted to organizations or markings its author cannot see. Give **Manage Playbooks** to the few people who design the team's automation |

## Playbooks and the other automation tools

| You want to | Use |
| --- | --- |
| Process data step by step with conditions, enrichment and changes | A playbook |
| Derive new relationships automatically across the whole knowledge base | The [rules engine](/opencti/configure/rules/) |
| Alert people about events, without changing data | [Notifications and triggers](/opencti/use/automate/notifications/) |
| Track who works on what, and where each report stands | [Track the progress of reports and cases](/opencti/use/automate/entity-workflows/) |
| Change hundreds of existing objects once | A [background task](/opencti/use/automate/background-tasks/) |
| Bring data in from an external source on a schedule | A [connector or a feed](/opencti/use/ingest/) |

A playbook can call a connector, apply one rule or send a notification as a step, so the tools combine rather than compete.

:::::only{opencti=">=7.260811"}
::::xtm-one[Playbooks with XTM One]
A playbook can hand its bundle to an XTM One AI agent, for work that is easier to describe in words than to build from components.

- **Transform with AI agent** gives the bundle and your **Additional instructions (optional, prepended to the STIX bundle)** to an agent, then passes the STIX 2.1 bundle the agent returns to the next step. It offers the agents of your XTM One catalog that transform STIX bundles. When the agent could not run or did not return a STIX bundle, the original bundle leaves through `unmodified`; a bundle the agent returns, even unchanged, leaves through `out`.
- **Send to AI agent** ends a branch: the agent receives the bundle with your **Instructions (optional, prepended to the STIX bundle)** and carries on alone, for example to summarize, alert or dispatch. Nothing comes back to the playbook. It offers the agents of your catalog that consume STIX bundles.

Both have a **Run as** setting: the account the agent acts as, in XTM One and when it writes back to OpenCTI. You can pick yourself or a service account, nobody else; left empty, the agent acts as the platform's initial administrator, so set it. Each run uses your XTM One quota, and the step is skipped when XTM One is not connected to the platform. [Create a playbook](/opencti/use/automate/playbooks/create/) gives a complete example.

:::only{opencti=">=7.261117"}
OpenCTI waits up to 10 minutes for the agent's answer, enough for an agent that builds a large bundle or calls several tools. Past that, the step counts as one where the agent could not run.
:::

:::only{opencti="<7.261117"}
OpenCTI waits up to 5 minutes for the agent's answer. Past that, the step counts as one where the agent could not run.
:::
::::
:::::

## For administrators: the playbook manager

Playbooks run in the playbook manager, which needs an Enterprise Edition license and `playbook_manager:enabled` (`PLAYBOOK_MANAGER__ENABLED`, on by default). It follows knowledge events as they happen and checks scheduled playbooks every minute, an interval fixed in the code: `playbook_manager:interval` (`PLAYBOOK_MANAGER__INTERVAL`) is declared in the default configuration but has no effect. Every key of the manager is in the [Engines, Schedules and Managers](/reference/opencti/configuration/#engines-schedules-and-managers) section of the configuration reference.

## What's next

- [Create a playbook](/opencti/use/automate/playbooks/create/), step by step, with a complete triage playbook.
- [Choose a component](/opencti/use/automate/playbooks/choose-a-component/) for the task you have in mind.
- [Monitor and troubleshoot](/opencti/use/automate/playbooks/monitor/) a running playbook.
- [Playbook components reference](/reference/opencti/playbook-components/), every setting and output.
