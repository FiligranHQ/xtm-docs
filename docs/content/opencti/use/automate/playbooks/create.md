---
title: Create a playbook
description: Build an OpenCTI playbook that triages new reports for the team,
  test it, offer it on chosen entities, and duplicate, export or import
  playbooks, including from the XTM Hub.
sidebar_position: 2
doc_type: how-to
products:
  - opencti
edition: ee
personas: [cti-lead, cti-analyst, admin]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/playbook-creation/
---

This page builds a playbook that triages every new hospitality report for your team: it marks the report as in progress, assigns it to the analyst on duty and alerts the CTI analysts, so no report waits unnoticed. The same steps build any playbook. The page then covers testing, offering a playbook on chosen entities, and moving playbooks between platforms. If you are new to playbooks, read [Automate with playbooks](/opencti/use/automate/playbooks/) first for how data flows through them and what to decide before you build.

## Before you start

- Your platform has an Enterprise Edition license.
- Your role has the **Manage Playbooks** capability, and **Create / Update knowledge** to see the buttons that create or import a playbook. Users who enroll entities in playbooks need **Use Playbooks**.
- You know what should start the playbook, which data it may change and who must be told. The [component chooser](/opencti/use/automate/playbooks/choose-a-component/) maps common tasks to components.
- The values the playbook sets exist on the platform: the status in the [workflow](/opencti/use/automate/entity-workflows/) of the entity type, the labels, the users or groups to assign or notify.

## Build and start a playbook

::::steps

1. Open :menu[Data > Processing > Automation] and click **Create Playbook**.
2. Give the playbook a name and a description that says what it does, then click **Create**. The playbook opens on an empty canvas, stopped.

   ::screenshot{id=opencti-playbook-create-dialog alt="The Create a playbook panel with an empty Name field, an empty Description field, and the Cancel and Create buttons."}

3. Open the list of entry points from the middle of the empty canvas.

   :::only{opencti=">=7.261119"}
   Click **Add an entry point**.
   :::

   :::only{opencti="<7.261119"}
   Click **Add component**.
   :::

   Choose **Listen knowledge events**, **Listen PIR events**, **Query knowledge on a regular basis** or **Available for manual enrollment / trigger**, and set its filters so the playbook only runs on the data you mean; the [filter key reference](/reference/opencti/filter-keys/) lists the keys and operators they accept.

   ::screenshot{id=opencti-playbook-entry-points alt="The Add component panel listing the four entry points under Start Playbook: manual enrollment, knowledge events, PIR events and scheduled knowledge queries."}

4. Add the steps that filter, enrich or change the data. On the canvas:

   :::only{opencti=">=7.261119"}
   - to add a step after a component, click the dashed **+** next to its output, **Add component**, then type a few words in **Search components**: the list keeps the components whose name or description contains them all;
   - to insert a step between two components, click **Insert a component on this link**, the **+** on the link between them. A step that ends a playbook, such as **Send for ingestion**, goes on a new branch of the same output, and the steps after the link stay in place;
   - to start a second branch from the same output, open the menu of the step already there, with its three dots or a right click, and choose **Add a sibling node**;
   - to swap a component for another, open its menu and choose **Replace**;
   - to move a step, with the steps after it, under another output, drag that output onto the step, or the input of the step onto that output.

   Each setting the component requires must be filled, such as the **Notifiers** and **Authorized members** of a notifier: until then the form shows **This field is required** and does not save.
   :::

   :::only{opencti="<7.261119"}
   - to add a step after a component, click **Add component** under its output;
   - to insert a step between two components, click the **+** on the link between them;
   - to start a second branch from the same output, click **Add a sibling node**, the arrow at the bottom-right of a component;
   - to swap a component for another, open its menu and choose **Replace**.
   :::

   ::screenshot{id=opencti-playbook-component-palette alt="The Add component panel opened after an entry point, listing the components that transform, enrich, share and end a playbook."}

5. End every branch. Use **Send for ingestion** wherever the playbook changed data, otherwise the changes are lost. Add **Send to notifier** or **Send email from template** where people should be told. To both save and notify, add a sibling branch.

   ::screenshot{id=opencti-playbook-sibling-branch alt="The out output of Mark as in progress splits into two branches: one ends with Send for ingestion, the other with Notify the CTI team."}

6. Open the menu next to the playbook name, choose **Start** and confirm. The header shows **Playbook is running**. From now on, every matching event runs it.

::::

::::example[Triage new hospitality reports automatically]
Barbara leads the CTI team of Example CERT. Every new report labelled `hotel-sector` must be marked as in progress, assigned to the analyst on duty this week, Ana Duval, and announced to the CTI analysts. She creates the playbook `Triage new hospitality reports` with four components:

| Component | Name on the canvas | Settings |
| --- | --- | --- |
| **Listen knowledge events** | `Listen to new reports` | **create** on, **update** and **delete** off. **Filters**: **Entity type** = Report and **Label** = `hotel-sector` |
| **Manipulate knowledge** | `Mark as in progress` | Scope left on the main element, the report. First action: **Action type** **Replace**, **Field** **Status**, value `IN_PROGRESS`. Second action, with **Add action**: **Add**, **Assignees**, `Ana Duval` |
| **Send for ingestion** | `Send for ingestion` | No settings. Linked to the output of `Mark as in progress` |
| **Send to notifier** | `Notify the CTI team` | **Notifiers**: **User interface**. **Authorized members**: the **CTI analysts** group. Added with **Add a sibling node** on `Send for ingestion`, so both branches start from the output of `Mark as in progress` |

:::only{opencti=">=7.261119"}
For the status, Barbara first sets **Entity type** to Report, then picks `IN_PROGRESS` among the statuses of the report workflow.
:::

When a partner publishes a report with the `hotel-sector` label, the report is saved with the `IN_PROGRESS` status and Ana Duval as assignee, and every CTI analyst gets an alert in the platform. A report without the label does not start the playbook. When the duty changes, Barbara replaces `Ana Duval` in the second action.
::::

::::only{opencti=">=7.260811"}
:::xtm-one[Hand the result to XTM One]
An AI agent can take on a step that is hard to express as filters, such as reading a report's description. Insert **Transform with AI agent** between the entry point and the next step: the agent returns the bundle, changed or not, and the playbook carries on.

For example, to catch the hospitality reports that partners forget to label, the triage playbook above listens to every new report (**Filters**: **Entity type** = Report only), then runs a `Classify hospitality reports` step:

- **AI agent**: the STIX transformation agent of your XTM One catalog; the list only offers agents that return a STIX bundle
- **Run as**: the `playbook-automation` service account, so the work is not tied to a person who may leave
- **Additional instructions (optional, prepended to the STIX bundle)**: `Add the label hotel-sector to the report when its description is about hotels, hotel booking systems or hotel front desks. Change nothing else.`

Its `out` output leads to a **Match knowledge** step on **Label** = `hotel-sector`, followed by `Mark as in progress` as above; its `unmodified` output ends there. Each run uses your XTM One quota. To end a branch with an agent instead, for example to have it post a summary, use **Send to AI agent**. See the [component reference](/reference/opencti/playbook-components/#playbook-ai-agent-transform-component).
:::
::::

## Check that it works

:::steps

1. Before you start the playbook, add a filter on **Creator** set to your own user to its entry point, so other people's data does not trigger it while you test.
2. Create or update an entity that matches the entry point filters, for example a test report with the `hotel-sector` label.
3. Open the execution traces of the playbook and check that every step ran. See [Monitor and troubleshoot](/opencti/use/automate/playbooks/monitor/) if a step is missing or red.
4. Remove the **Creator** filter. The playbook now runs on everyone's data.

:::

::::only{opencti=">=7.261119"}
## Change a playbook

A playbook follows the team: the analyst on duty changes, a step turns out to be useless. Open the playbook from :menu[Data > Processing > Automation].

A stopped playbook opens in **Edit mode** when your role has **Manage Playbooks**. A running playbook opens in view only: you can move around the canvas, zoom, change the layout and open the last execution traces, but not change a step. To change it while it runs, turn on **Edit mode** in the header. The canvas then warns that "This playbook is running: your changes apply immediately."

::screenshot{id=opencti-playbook-editor alt="The triage playbook in Edit mode: each card shows the kind of its step, such as Trigger or End of playbook, and a summary of its settings, and the toolbar at the top right holds Undo, Redo, the two layouts, the zoom buttons and Fit view."}

Each card shows the name of the step, its kind and a summary of its settings, such as `2 filters`. The kind is **Trigger** for the entry point, **End of playbook** for a step without outputs, and the category of the component for the others. A step with a required setting left empty, for example after an import, shows **Configuration required**; a step whose component no longer exists shows **Component unavailable**. Once the playbook has run, a badge on each step gives the result and the duration of its **Last execution**. Click a card to open its settings.

- **Delete** a step from its menu. When steps follow it, choose **Delete this step only**, the default, to link them to the step before it, or delete them with it. Deleting the entry point deletes every step after it.
- **Undo** and **Redo**, at the top right of the canvas, take back or replay your last 50 changes until you leave the page. Ctrl+Z and Shift+Ctrl+Z do the same, with ⌘ on macOS. Deleting a step with the steps after it, inserting a step on a link, or replacing a component by one with fewer outputs cannot be undone, and empties the history.
- Drag an output onto a step, or the input of a step onto an output, to move that step and the steps after it under that output. OpenCTI refuses a link that would make a loop or leave a step without a parent.
- **Vertical layout** and **Horizontal layout** draw the playbook from top to bottom or from left to right, and your browser remembers the choice for each playbook. **Fit view** brings the whole playbook into view. Press **?** on the canvas to list the keyboard shortcuts.

:::example[Hand the triage over to a new duty analyst]
Ana Duval goes on leave and Leo Martin takes the duty while `Triage new hospitality reports` is running. Barbara opens the playbook: it shows in view only. She turns on **Edit mode**, clicks the `Mark as in progress` card, replaces `Ana Duval` with `Leo Martin` in the **Assignees** action and clicks **Update**.

The next hospitality report is assigned to Leo Martin; the reports triaged before keep Ana Duval. Had she picked the wrong analyst, **Undo** would bring back the previous settings of the step.
:::
::::

## Run a playbook on selected entities

Some actions are a choice for an analyst, not a rule: open a case for this report, send these IP addresses to a sandbox. Offer them as a playbook that analysts with **Use Playbooks** run on the entities they pick.

:::only{opencti=">=7.260811"}
A playbook appears in the **Enroll in playbook** list of an entity when it is running and either starts with **Available for manual enrollment / trigger**, or starts with **Listen knowledge events** with filters that match the entity and its `canEnrollManually` switch on (the default). The settings panel of the entry point shows this switch under its technical name. Turn it off for a playbook that must only react to events, so analysts cannot run it by hand.
:::

:::only{opencti="<7.260811"}
A playbook appears in the **Enroll in playbook** list of an entity when it is running and starts with **Available for manual enrollment / trigger** or **Listen knowledge events**, with filters that match the entity.
:::

1. Open the entity, open its actions menu (⋮) and choose **Enroll in playbook**. To enroll many entities at once, select them in any list and use the same action in the toolbar.
2. Choose the playbook. It runs on each entity as its main element.

::screenshot{id=opencti-playbook-enroll-select alt="The Available playbooks panel lists the running playbooks with a manual or event entry point that match the entity, each with a run button."}

To offer a playbook on any entity, leave the entry point filters empty and put a **Match knowledge** step right after it.

When the run button of a playbook, **Trigger this playbook now**, shows an error instead of **Playbook successfully completed.**, the playbook or the entity usually changed after the list opened. [When a playbook does not run on an entity](/opencti/use/automate/playbooks/monitor/#when-a-playbook-does-not-run-on-an-entity) explains the message and what to do.

## Duplicate, export and import

**Duplicate** a playbook to start a variant: in the list, open the row menu and choose **Duplicate**. The same action exists inside an open playbook.

**Export** a playbook to move it to another platform or to attach it to a support request: open the row menu and choose **Export**. You get a JSON file.

**Import** it on the target platform with **Import playbook** in :menu[Data > Processing > Automation]. An imported playbook is stopped. Before you start it, check that everything it refers to exists on this platform, because missing objects show as deleted:

| Where | What to check |
| --- | --- |
| Filters of entry points, Match knowledge, Reduce knowledge and Manipulate knowledge | Labels, vocabularies, marking definitions, specific entities or observables, authors and creators |
| Enrich through connector | The connector exists and is running. Its ID is in the URL of its monitoring page |
| Container wrapper | The case template and its task templates |
| Send to notifier, Send email from template | The notifier or email template, and every recipient |

:::only{opencti=">=7.261119"}
In a step that shares or unshares with organizations, an organization this platform does not have, or no longer has, shows in **Target organizations** with `- deleted` after its name, and the field says **A selected item no longer exists: remove it and pick another**. Replace it before you start the playbook.

OpenCTI refuses to import or duplicate a playbook with more than one entry point, a step without a parent, or a step whose settings or filters are not valid JSON. Fix the playbook where it comes from, then export it again.
:::

:::only{opencti=">=7.261127"}
It also refuses a file without the list of steps and links of the playbook, and a step saved without any settings.
:::

### From the XTM Hub

The XTM Hub hosts ready-made playbooks. Click **Import from Hub** in :menu[Data > Processing > Automation] to browse them; the button only shows when your platform can reach the XTM Hub. If your platform is [connected to the XTM Hub](/opencti/configure/xtm-hub/), you can also deploy a playbook from the Hub in one click and land on it in OpenCTI. Either way, the playbook arrives stopped: check it as above, then start it. [XTM Hub content](/opencti/configure/xtm-hub/content-library/) explains how deployment works.

## Design tips

- **Notify groups, not people.** A notifier or an assignment aimed at a group follows the duty rota when its members change; one aimed at a person needs an edit each time.
- **Chain playbooks with labels.** A first playbook adds a label such as `weekly-digest`; a scheduled second playbook collects everything with that label into a report.
- **Filter early.** Put **Match knowledge** or **Reduce knowledge** before connectors and rules, which fail on elements outside their scope.

## What's next

- [Monitor and troubleshoot](/opencti/use/automate/playbooks/monitor/) your playbook once it runs.
- [Playbook components reference](/reference/opencti/playbook-components/) for every setting.
