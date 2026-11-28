---
title: Containers
description: How reports, groupings and cases keep what a source said together without owning the knowledge they hold, and what that changes when you add, share or delete.
sidebar_position: 4
doc_type: explanation
products:
  - opencti
personas: [cti-analyst, cti-lead]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/containers/
---

A container keeps together what one source said or one piece of work found. The report "Velvet Harbor targets hotel booking systems" holds the intrusion set, its malware, its techniques, its indicators and the relationships Example CERT asserts between them. The objects themselves stay independent: Velvet Harbor is the same Intrusion Set in every report and case that mentions it. This page explains which container fits which work, how a container holds knowledge, and what that changes when you add, find, share or delete.

## Which container for which work

| Container | Use it to | Menu |
| --- | --- | --- |
| Report | Keep what a source published at a date, as it said it: a vendor advisory on Velvet Harbor, your own monthly brief on the hospitality sector | :menu[Analyses > Reports] |
| Grouping | Keep objects together before you can say what they mean, such as the domains of a phishing wave you are still analyzing | :menu[Analyses > Groupings] |
| Note | Add your text, analysis or caveat to one or more objects or relationships | :menu[Analyses > Notes] |
| Opinion | Say how much you agree with an object someone else created | On the object's **Overview** |
| Observed Data | Record raw observations: which observables were seen, how many times and over which period | :menu[Events > Observed datas] |
| Incident response, Request for information, Request for takedown | Carry an investigation or a request to its end, with tasks and assignees. See [Handle incidents and requests in cases](/opencti/use/analyze/cases/) | :menu[Cases] |
| Task | Track work to do by a date, usually inside a case | :menu[Cases > Tasks] |
| Feedback | Hold a user's feedback on an object | :menu[Cases > Feedbacks] |

Reports, groupings, notes, opinions and observed data come from Structured Threat Information Expression (STIX) 2.1. Cases, tasks and feedbacks are OpenCTI's own, and are exported as STIX custom objects.

## How a container holds knowledge

::diagram{id=opencti-containers-refs}

A container points to its content; it does not own it. Each object it holds is linked to it, as authors and markings are linked to every object (see [Nested objects](/opencti/concepts/nested-objects/)). Three consequences follow:

- **One object, many containers.** Velvet Harbor is in Example CERT's report and in the incident response "Booking system intrusion at a hotel chain". Whatever you learn about it, you learn once, and every container shows it.
- **A container can hold relationships.** The report contains "Velvet Harbor uses Velvet Loader", not only the two objects: the report states that link. When two sources state the same link, both reports contain the same relationship, and the page of the relationship lists both.
- **A container records who said what.** The author, the date and the confidence of a report are those of the source. The objects inside keep their own, so a weak link, such as "Velvet Harbor uses Harbor Stealer" at confidence 30, stays weak inside a confident report.

## Add knowledge to a container

You add to a container from its tabs. The **Knowledge** tab shows the content as a graph, a correlation view or an ATT&CK matrix, and as a timeline on reports and cases, and whatever you create there joins the container. The **Content** tab holds the text and files of the container: map words of the text to objects, and the objects join it. The **Entities** and **Observables** tabs list the content by family, and **Add entity** adds existing objects or creates new ones.

:::only{opencti=">=7.261030"}
The **Relationships** tab lists the relationships the container holds, with filters.
:::

::screenshot{id=opencti-containers-entities alt="The Entities tab of the report Velvet Harbor targets hotel booking systems lists the intrusion set Velvet Harbor and the sector Hospitality, with the Add entity button above the list."}

When you create an object from a container, the **Markings** field starts with the markings of the container. Check them before you save: a container's markings do not travel to its content, so each object is visible according to its own markings. An indicator taken from a TLP:RED report is usually TLP:RED too. The tabs, views and the content mapping are described step by step in [Explore analysis](/opencti/use/explore/analysis/).

## Find every source on a subject

Every entity has an **Analyses** tab that lists the containers it is in: from the malware Harbor Stealer, you reach every report that mentions it. On a report, a grouping or a case, **Correlated containers** lists the other containers that share part of its content, a quick way to find the earlier reports on the same campaign. Container lists can also be filtered on their content. See [Pivot and investigate](/opencti/use/analyze/pivot/).

## Share a container

What travels with a container depends on how you restrict or share it:

- **Markings** apply to each object on its own. A reader who lacks TLP:AMBER does not see a TLP:AMBER report, but may see the objects inside that carry other markings.
- **Share with an organization** requires the Enterprise Edition. On a report, a grouping, an observed data or a case, it shares the container and all its content with the organization, and every object added later is shared too. See [Separate data between organizations](/opencti/administer/organizations/#share-knowledge-with-an-organization).
- **Authorized members** restrict the container itself to chosen users, groups or organizations. See [Restrict one object to chosen people](/opencti/administer/sharing/).

How the three combine is in [How OpenCTI decides who sees what](/opencti/administer/segregation/), and how to send a report out of the platform is in [Export knowledge to a file](/opencti/use/share/export/).

## Remove and delete

- **Removing an object from a container** deletes only the link. The object stays in the platform and in its other containers.
- **Deleting a container** keeps its content by default, and the container goes to the [trash](/opencti/use/delete-restore/). When you delete a report and some of its objects are in no other container, the dialog counts them and offers **Also delete these elements**, so that you do not leave orphans behind.

::::xtm-one[Summarize a container]
On a container, **AI Insights** writes a **Container summary** of its content. It needs an Enterprise Edition license and a configured AI service.

:::only{opencti=">=7.260811"}
When XTM One is connected, you choose the AI agent that writes the summary with **Select agent**.
:::
::::

## What's next

- [Data model](/opencti/concepts/data-model/): the objects a container holds.
- [Explore analysis](/opencti/use/explore/analysis/): read and build reports, groupings, notes and opinions.
- [Handle incidents and requests in cases](/opencti/use/analyze/cases/): work on incident responses and requests with tasks.
- [Import from files](/opencti/use/ingest/files/): turn a document into a report with its content.
- [Export knowledge to a file](/opencti/use/share/export/): export a report and its content as a STIX 2.1 bundle.
