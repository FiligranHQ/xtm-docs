---
title: Explore cases
description: Find the incident responses, requests and tasks that need you in the OpenCTI Cases menu, read a case at a glance, review workloads and handle user feedbacks.
sidebar_position: 5
doc_type: how-to
products:
  - opencti
personas: [soc-analyst, cti-lead, cti-analyst]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/exploring-cases/
---

The **Cases** menu is the queue of your team's work: the incidents being handled, the questions asked to the CTI team, the takedowns under way, and every task with its owner and due date. This page shows how to find the cases and tasks that need you, read a case in a few seconds, review who is late, and answer user feedbacks. To open and run a case, see [Handle incidents and requests in cases](/opencti/use/analyze/cases/).

## Before you start

- Your role has the **Access knowledge** capability.
- Changing a case needs **Create / Update knowledge**, and deleting one **Delete knowledge**.

## What the Cases menu holds

| Entry | What it is |
| --- | --- |
| **Incident responses** | The context and actions of the response to one incident. It does not represent the incident itself, which lives in **Events** |
| **Requests for information** | A question asked to the CTI team, about an incident or a threat, and its answer |
| **Requests for takedown** | A request to remove attack infrastructure, such as a domain that impersonates your organization or a phishing mailbox |
| **Tasks** | Every task of every case, to review due dates or see what is assigned to someone |
| **Feedbacks** | What users of the platform told the CTI team about the knowledge |

Like reports, cases are containers: they hold the knowledge of the investigation and become the record of what happened. They are not STIX 2.1 objects.

## Find the cases that need you

:::steps

1. Open :menu[Cases > Incident responses], or the entry for the kind of case you want.
2. Filter the list on **Priority**, **Severity**, **Assignees** or **Processing status**. The columns show the same fields, so you can also sort on them.
3. Click a case to open it on its **Overview**.

:::

:::example[Find the serious hotel incidents your team is handling]
At the morning meeting, you review only the incident responses that matter most. In :menu[Cases > Incident responses], add two filters:

- **Priority**, operator **Equals**, values **P1** and **P2**.
- **Severity**, operator **Equals**, values **critical** and **high**.

The list keeps **Booking system intrusion at a hotel chain**, priority P2 and severity high. **Sighting of booking-velvetharbor.example**, also P2 but of low severity, is left out. Save the filter to find the same view tomorrow.
:::

## Read a case at a glance

The **Overview** of a case answers, without opening another tab, what happened and where the work stands:

- **Details**: the **Priority**, the **Severity**, the type of the case (**Incident response type** for an incident response), and **Correlated containers**, the reports and cases that share objects with this one.
- **Tasks**: the tasks of the case, with their due dates, assignees and status.
- **Origin of the case**: the incident, sighting or report that started it.
- **Observables** and **Other entities**: what the case contains, split in two lists.
- **Most recent history** and the notes: who did what last.

::screenshot{id=opencti-cases-incident-response-overview alt="The Overview of an incident response: its priority and severity, the list of its tasks with due dates, the incident at the origin of the case, and the observables it contains."}

On the booking system intrusion, **Tasks** shows **Collect the booking platform logs**, **Reset the front-desk accounts** and **Notify the payment card brands**; **Origin of the case** shows the incident **Unauthorized access to booking database**; **Correlated containers** shows that the domain of the case was already seen in another hotel.

:::only{opencti=">=7.261030"}
The other tabs are those of any container: **Knowledge**, **Content**, **Entities**, **Observables**, **Relationships** and **Data**, see [Explore knowledge](/opencti/use/explore/#tabs).
:::

:::only{opencti="<7.261030"}
The other tabs are those of any container: **Knowledge**, **Content**, **Entities**, **Observables** and **Data**, see [Explore knowledge](/opencti/use/explore/#tabs).
:::

**Knowledge** offers the same four views as a report: graph, timeline, correlation and matrix. The timeline puts the content of the case in date order to rebuild the chain of events. **Display relationships** shows the relationships instead of the objects, and **Use functional dates** orders them by the dates the knowledge itself carries, such as the start time of a relationship, instead of the date they entered the platform. See [Explore analysis](/opencti/use/explore/analysis/#read-a-report-in-its-knowledge-tab).

A case missing from your list may have been kept to some people with **Manage access restriction**: ask its owner. See [Sharing models](/opencti/administer/sharing/).

## Review tasks

:menu[Cases > Tasks] lists the tasks of all cases. Sort on **Due Date** or filter on **Due date** to find late tasks, whose due date shows in red, or on **Assignees** to see one person's workload before you hand them a new case. A task page has **Overview**, **Content**, **Data** and **History**: the history shows who changed its status and when.

:::example[List your tasks due this week]
In :menu[Cases > Tasks], add **Assignees**, operator **Equals**, value **@me**, and **Due date**, operator **Within**, **From** `now` **To** `now+7d`. The list shows the tasks assigned to you that fall due in the next seven days, such as **Collect the booking platform logs** of the booking system intrusion. Save the filter to find the same view every Monday: the relative dates move with the calendar.
:::

As a team lead, remove the **Assignees** filter and set **Due date** to **From** `now-30d` **To** `now`: the list shows the tasks that fell due in the last 30 days, their due dates in red. Those whose **Processing status** is not final are late.

## Handle a feedback

Any user can send the CTI team a feedback: in the top bar, open **Profile** and choose **Feedback**. The feedback holds a description, a **Rating** and the **Entities** it is about. It arrives in :menu[Cases > Feedbacks] as a case to solve, often about a gap or an inconsistency in the knowledge.

:::steps

1. Open :menu[Cases > Feedbacks] and click a feedback.
2. Read the description and the **Rating** in **Details**, and open the entities it points to.
3. Fix the knowledge, write a note on the feedback to record what you did, and update its status with **Update**.

:::

## What's next

- [Handle incidents and requests in cases](/opencti/use/analyze/cases/), to open cases, apply templates and follow workflows.
- [Explore events](/opencti/use/explore/events/), for the incidents and sightings that start a case.
- [Read the home dashboard and AI summaries](/opencti/use/explore/insights/), for an AI summary of a case.
- The [data model reference](/reference/opencti/data-model/#cases), for the attributes and relationships of each type of the Cases menu.
