---
title: Handle incidents and requests in cases
description: Open an incident response, a request for information or a request for takedown in OpenCTI, put the evidence in it, split the work into tasks and close it.
sidebar_position: 4
doc_type: how-to
products:
  - opencti
personas: [soc-analyst, cti-analyst]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/case-management/
---

A case is where your team works an incident: what started it, the observables and threats behind it, the tasks with their owners and due dates, and the report you write at the end. Opening one takes a minute: a name, a severity, a priority and a template of tasks. This page shows you how to open a case, fill it with evidence, run its tasks and close it.

## Choose the type of case

| Case | Open it when | Find it in |
| --- | --- | --- |
| Incident response | You investigate and remediate a security incident | :menu[Cases > Incident responses] |
| Request for information (RFI) | Someone asks the intelligence team a question | :menu[Cases > Requests for information] |
| Request for takedown (RFT) | You want a malicious domain, account or page removed | :menu[Cases > Requests for takedown] |

The three work the same way, and each can hold any entity and relationship, so the intelligence travels with the work. Requests for information also carry the requests of users who ask for knowledge they cannot see, see [Let users request access](/opencti/administer/request-access/). Feedbacks, the fourth kind of case, are in [Explore cases](/opencti/use/explore/cases/#handle-a-feedback).

## Before you start

- Your role has **Create / Update knowledge** to create and change cases and tasks, and **Delete knowledge** to delete them.
- To restrict a case to some people, your platform needs an Enterprise Edition license and your role **Manage authorized members**.
- Lists of tasks come from case templates. An administrator creates them in :menu[Settings > Taxonomies > Case templates], which needs **Manage case templates**. See [Case templates](/opencti/configure/taxonomies/#case-templates).

## Open a case

:::steps

1. Open :menu[Cases > Incident responses] and click **Create Incident response**. The **Create an incident response** panel opens.
2. Enter a **Name** the team recognizes in a list: the asset and what happened.
3. Set the **Incident date** to when the incident happened. Left empty, it is the creation time. The case lists show it as **Original creation date** and date filters use it, so a case opened the morning after a night-time intrusion should carry the night's date.
4. Set the **Severity** (**low**, **medium**, **high**, **critical**), how bad the impact is, and the **Priority** (**P1** to **P4**), how soon the team must act. Case lists and dashboards filter on both, so change them as the facts change.
5. Pick the **Incident type**, such as **data-leak** or **ransomware**. The **Details** card of the case shows it as **Incident response type**. Your administrator can add values in [vocabularies](/opencti/configure/taxonomies/#vocabularies).
6. Under **Default case templates**, pick the templates whose tasks the case needs. Their tasks are created with the case.
7. Add the **Assignees**, who own the case and find it under their name, and the **Participants**, who contribute to it.
8. To keep the case to the people working it, open **Advanced options** and set the authorized members.
9. Click **Create**.

   ::screenshot{id=opencti-cases-create alt="The Create an incident response panel with the Name, Incident date, Severity, Priority, Incident type and Default case templates fields."}

:::

The **Create a request for information** panel has a **Request For Information Date** and **Request for information types** instead of the incident date and type. The **Create a request for takedown** panel has a **Request For Takedown Date** and a **Request for takedown type**, such as **phishing** or **brand-abuse**. The rest of the form is the same.

If you paste a first write-up in **Content**, a **Create and map** button appears next to **Create**. It creates the case and opens its **Content mapping view**, where you turn the names in your text into objects of the case, see [Map text to knowledge](/opencti/use/explore/analysis/#turn-a-report-s-text-into-knowledge).

### Faster ways to open a case

- **From an investigation.** Open the investigation menu, choose **Add to a container** and create the case: everything you pivoted to goes into it. See [Pivot and investigate](/opencti/use/analyze/pivot/#capitalize-on-the-investigation).
- **From a list.** Select entities, such as the indicators of an alert, and click **Add in container** in the toolbar.
- **Automatically.** In a [playbook](/opencti/use/automate/playbooks/), the **Container wrapper** component puts new incidents or sightings in a case, with the tasks of the **Case templates** you choose.

## Put the evidence in the case

Each part of the case page holds one kind of evidence:

| Where | What to put there |
| --- | --- |
| **Overview**, card **Origin of the case** | What started the case: the incident, the sighting or the report. Click **Add** on the card |
| **Observables** tab | The technical values of the alert: IP addresses, domains, URLs, hashes, email addresses. Click **Add observable**: the **Add observables** panel finds existing values, and **Create an observable** adds a new one. To add the addresses of an alert in one go, pick the type in **Create an observable**, open **Create multiple observables** and paste one value per line |
| **Entities** tab | Everything else: the intrusion set or malware behind the attack, the affected organization, the attack patterns. Click **Add entity**: the **Add entities** panel works the same way, with **Create an entity** |
| **Knowledge** tab | The same objects and their relationships, in the [views of a container](/opencti/use/explore/#the-views-of-a-container): a graph, a timeline, the other containers they correlate with, and the MITRE ATT&CK matrix of their techniques. Draw relationships here, such as the domain that resolves to an address |
| **Content** tab | Your write-up: the incident report or the answer to the request |
| **Data** tab | The files: logs, screenshots, the original phishing email, and the exports of the case |

:::only{opencti=">=7.261119"}
The buttons at the top right of **Knowledge** switch between **Graph view**, **TimeLine view**, **Correlation view** and **Matrix view**.

A screenshot or a document you add in **Data**, in PNG, JPEG, PDF, text, HTML or Markdown, opens in the **Content** tab when you click its line, next to your write-up: you read the capture of the alert without downloading it. On every line, **Open in a new tab** shows the file in the browser and **Download this file** saves it. A program file (`.exe`, `.dll`, `.pif`) is never opened: you choose between **Raw file** and **Encrypted archive**. A click on another kind of file, such as the original email, does nothing: use these two buttons.
:::

:::only{opencti="<7.261119"}
The buttons at the top right of **Knowledge** switch between **Graph view**, **TimeLine view**, **Correlation view** and **Tactics matrix view**.
:::

The **Overview** splits what the case holds into **Observables** and **Other entities**. In **Details**, **Correlated containers** lists the reports and cases that share objects with this one: the fastest sign that the incident belongs to a known campaign. Click **Display the correlation graph** to see what they share.

Write notes as you go, to record what you checked and why. At the end, an opinion records how the case was handled, for lessons learned.

## Organize the work in tasks

The **Tasks** card of the **Overview** holds the tasks of the case.

:::steps

1. Click **Add a task to this container**. Enter a **Name**, a **Due Date** and the **Assignees**, then click **Create**.
2. To add a prepared list of tasks, click **Apply a new case template**, pick the templates and click **Apply**.
3. Move each task through its status as the work goes. Assignees find their tasks in :menu[Cases > Tasks], filtered by assignee and due date.

:::

::screenshot{id=opencti-cases-tasks alt="The Tasks card of an incident response with three tasks, their due dates and assignees, and the buttons to add a task and to apply a case template."}

A task has its own page, with **Overview**, **Content**, **Data** and **History**: attach the proof of what was done, such as an export of the reset accounts, to the task itself.

:::tip
Give each analyst a custom dashboard with a list of the tasks assigned to them, so the work comes to them. See [Build dashboards](/opencti/use/dashboards/).
:::

## Track and close

The **Processing status** of a case shows where it stands. Administrators define the statuses and their order per case type, see [Track the progress of reports and cases](/opencti/use/automate/entity-workflows/). Until they do, it shows **Disabled**.

:::steps

1. Check in the **Tasks** card that every task has reached its final status.
2. Write the final report in the **Content** tab, or click **Generate an export** in the header and fill a template, as [Generate a finished intelligence document](/opencti/use/share/export/#generate-a-finished-intelligence-document) explains.
3. Move the case to its final status with **Update**.

:::

The case stays as the record of the incident. Its objects remain linked to it, so the next analyst who opens the intrusion set finds the case in its **Analyses** tab.

:::example[Open the case for a booking system intrusion]
At 08:00, Grand Meridian Hotels reports that a front-desk account exported guest records from its booking platform at 02:10. You are the analyst on duty.

In :menu[Cases > Incident responses], click **Create Incident response** and enter:

- **Name**: `Booking system intrusion at a hotel chain`
- **Incident date**: `2026-01-09 02:10`, when the export happened
- **Severity**: **high**, since guest and payment data left the platform
- **Priority**: **P2**, to act today without waking the whole team
- **Incident type**: **data-leak**
- **Assignees**: yourself

Click **Create**. On the **Overview**, click **Add** in **Origin of the case** and pick the incident **Unauthorized access to booking database** the hotel reported. In **Observables**, click **Add observable** and add `203.0.113.10` and `booking-velvetharbor.example`, found in the hotel's proxy logs. In **Entities**, click **Add entity** and add the intrusion set **Velvet Harbor**.

**Correlated containers** now lists the report **Night-time exports from a hotel booking platform** and the case **Sighting of booking-velvetharbor.example**: another hotel saw the same domain the day before. In the **Tasks** card, add:

- `Collect the booking platform logs`, due `2026-01-16 17:00`, assigned to you
- `Reset the front-desk accounts`, due `2026-01-17 17:00`
- `Notify the payment card brands`

The tasks appear in :menu[Cases > Tasks], and the case appears in the **Analyses** tab of Velvet Harbor for the CTI team.
:::

### When a sighting opens the case

When your web proxy or SIEM reports that it saw an indicator from OpenCTI, it creates a sighting. The rule **Raise incident based on sighting** turns the sighting of a valid indicator into an incident, and a [trigger](/opencti/use/automate/notifications/) can notify you. Open the case from that incident and add the sighting to **Origin of the case**: it holds where and how many times the indicator was seen. See [How OpenCTI infers new relationships](/opencti/use/enrich/inferences/).

::::only{opencti=">=7.260811"}
:::xtm-one[Cases with XTM One]
On a case, **AI Insights** gives a summary of its content and activity. When XTM One is connected, a playbook can hand a new case to an AI agent with **Send to AI agent**, for example to draft the first analysis for review. See [XTM One in OpenCTI](/xtm-one/use/in-opencti/).
:::
::::

## What's next

- [Explore cases](/opencti/use/explore/cases/), to find the cases and tasks that need you.
- [Pivot and investigate](/opencti/use/analyze/pivot/), to gather the context of a case.
- [Automate with playbooks](/opencti/use/automate/playbooks/), to open cases with their tasks automatically.
- [Get alerted when knowledge changes](/opencti/use/automate/notifications/), to be told when a case or task changes.
