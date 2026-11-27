---
title: Explore analysis
description: Find the reports on a threat, read a report as a graph, timeline or ATT&CK matrix, turn its text into knowledge, and record your assessment in notes and opinions.
sidebar_position: 4
doc_type: how-to
products:
  - opencti
personas: [cti-analyst]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/exploring-analysis/
---

The **Analyses** menu is where intelligence enters OpenCTI and where you trace every fact back to its source. A report holds the threats, victims, techniques and indicators it talks about, so you can read it as a graph, a timeline or an ATT&CK matrix, find the other reports that share its objects, and turn the words of its text into linked knowledge. Notes, opinions and external references let you add your assessment and keep the sources at hand. The tabs every page shares are described in [Explore knowledge](/opencti/use/explore/#read-an-entity-page).

## Before you start

- Your role has the **Access knowledge** capability.
- Writing notes and opinions needs **Access to collaborative creation**. Changing a report or mapping its text needs **Create / Update knowledge**, and deleting it **Delete knowledge**.
- Sharing a report with an organization needs **Restrict organization access**; limiting its access needs **Manage authorized members**.

## What the Analyses menu holds

| Entry | Use it to |
| --- | --- |
| **Reports** | Read finished intelligence, from a source or your own team: a vendor report, a blog post, a MISP event. Every fact it contains can be traced back to it |
| **Groupings** | Keep together objects that share a context but are not a finished product yet, such as an ongoing investigation |
| **Malware analyses** | Read the result of a sandbox or a manual analysis of a file, a domain or a URL |
| **Security coverages** | See how well your defenses cover a threat, measured with OpenAEV. See [Measure your defenses against a threat](/opencti/use/analyze/security-coverage/) |
| **Notes** | Review the notes analysts wrote on any object or relationship |
| **External references** | Find the source documents and links attached to objects |

Reports, groupings and cases are containers: see [Containers](/opencti/concepts/containers/) for what that means and why knowledge stays tied to the container it came from.

## Find the reports about a subject

:::steps
1. Open :menu[Analyses > Reports].
2. Type a name in the search box, or add filters, for example on **Report types**, **Author**, **Label**, **Report publication date** or **Contains**, the objects the report contains.
3. Click a report to open it on **Overview**.
:::

:::example[Find the reports about Velvet Harbor published in January 2026]
For a monthly briefing, you need every report that mentions Velvet Harbor and was published in January 2026. In :menu[Analyses > Reports], add two filters:

- **Contains**, operator **Equals**, value **Velvet Harbor**: the reports that hold the intrusion set, whatever their title says.
- **Report publication date**, operator **Within**, **From** January 1, 2026 **To** January 31, 2026, picked with the date pickers.

The list keeps two reports: **Velvet Harbor targets hotel booking systems**, published by Example CERT on January 12, and **Night-time exports from a hotel booking platform**, the incident report of Grand Meridian Hotels from January 9. Add **Report types**, operator **Equals**, value `threat-report` to keep only the first one.
:::

It is often faster to start from the threat: its **Analyses** tab lists every report and case that contains it.

The **Overview** of a report tells you how far to trust it and what else to read. **Entity details** gives its **Report types** and **Publication date**, the **Entities distribution** of what it contains, and the **Correlated containers**: other reports and cases that share objects with it, drawn as a graph by **Display the correlation graph**. **Basic information** gives the **Reliability (of author)** of its source, its **Confidence level**, and the **Assignees** and **Participants** who work on it.

::screenshot{id=opencti-report-overview alt="The Overview of a report: its description, report types and publication date, the distribution of the entities it contains, and its reliability, confidence and processing status."}

## The tabs of a report

A report opens on **Overview**. Its other tabs show the same content from other angles, and a grouping or a case has the same ones:

| Tab | Its views | Use it to |
| --- | --- | --- |
| **Overview** | | Judge how far to trust the report and find the reports that share its objects, as above |
| **Knowledge** | **Graph view**, **TimeLine view**, **Correlation view** and a matrix view | Read and build what the report contains: see [Read a report in its Knowledge tab](#read-a-report-in-its-knowledge-tab) |
| **Content** | **Content view**, **Editor view**, **Content mapping view** | Read or edit the report's text and files, and link the words of the text to objects: see [Turn a report's text into knowledge](#turn-a-report-s-text-into-knowledge) |
| **Entities** | | List the threats, victims, techniques and other entities of the report as a table, with search and filters |
| **Observables** | | List the IP addresses, domains, files and other observables of the report, to hunt for them or export them |
| **Data** | | Attach the source document, export the report, and read its history |

:::only{opencti=">=7.261030"}
A **Relationships** tab lists the relationships of the report as a table too.
:::

:::only{opencti="<7.261030"}
The relationships of the report are in its **Knowledge** tab only.
:::

Each table shows whether an object was added directly or by an inference rule.

:::only{opencti=">=7.261119"}
In **Data**, a click on the source document, such as the PDF of a vendor report or a PNG or JPEG capture, opens it in **Content**, to read it next to the report's text without downloading it. **Open in a new tab** shows it in the browser. See [Explore knowledge](/opencti/use/explore/#data).
:::

## Read a report in its Knowledge tab

**Knowledge** shows what the report contains, through the four views of a container, also described in [Explore knowledge](/opencti/use/explore/#the-views-of-a-container): a graph, a timeline, the correlations and the ATT&CK matrix.

:::only{opencti=">=7.261119"}
Switch between them with the icons at the top right: **Graph view**, **TimeLine view**, **Correlation view** and **Matrix view**.
:::

:::only{opencti="<7.261119"}
Switch between them with the icons at the top right: **Graph view**, **TimeLine view**, **Correlation view** and **Tactics matrix view**.
:::

**Graph view** draws the objects and the relationships of the report, and is where you add to it.

::screenshot{id=opencti-graphs-knowledge alt="The Graph view of the report Velvet Harbor targets hotel booking systems: the intrusion set at the center, its targeted sector and used malware around it."}

**TimeLine view** places the dated relationships and sightings of the report in time, to follow the attack it describes.

::screenshot{id=opencti-analysis-report-timeline alt="The TimeLine view of the report Velvet Harbor targets hotel booking systems: its relationships ordered by date."}

**Correlation view** adds the other reports, groupings and cases that contain the same objects, to see whether the threat was already seen elsewhere. It starts with the indicators and observables only, the objects that prove two attacks share an infrastructure; **Show all correlated entities** adds the containers that share a malware, an intrusion set or any other object.

::screenshot{id=opencti-analysis-report-correlation alt="The Correlation view of the report Velvet Harbor targets hotel booking systems: the address 198.51.100.7 links it to the incident report Night-time exports from a hotel booking platform, which saw the same proxy."}

Groupings offer the same views except the timeline. In the graph, **Open the suggestions** proposes relationships that would make the knowledge consistent, **Start an investigation** copies the graph to a new investigation, and **Export to PDF** saves it. Layouts, selection and creation tools are described in [Work with graphs](/opencti/use/analyze/graphs/).

In the matrix view, the techniques of the report show on the MITRE ATT&CK matrix, with the views and options of the **Attack patterns** section of a threat, described in [Explore knowledge](/opencti/use/explore/#attack-patterns). **Display only used techniques** keeps only the techniques of the report, **Courses of action view** lists their mitigations, what defenders need from your report, and **Compare with my security posture** shows which of them your security platforms should cover.

::screenshot{id=opencti-analysis-report-matrix alt="The matrix view of the report Velvet Harbor targets hotel booking systems, limited to its three techniques: spearphishing attachment, PowerShell and web protocols."}

The **Observables** tab lists what you can hunt for, with the type and value of each observable:

::screenshot{id=opencti-analysis-report-observables alt="The Observables tab of the report Velvet Harbor targets hotel booking systems: its observables in a table, with their type and value."}

Containers keep their history in **Data**, under the files, instead of a **History** tab.

## Turn a report's text into knowledge

The content mapping view links the words of a report to objects: select a passage, pick the object it refers to, and the object joins the report. A malware that a source names in its text without structuring it becomes a way into the report.

:::steps
1. Open the report, then **Content**, then **Content mapping view**. The description and main content are on the left, the objects of the report on the right, with the number of times each one is mapped.
2. Select a word or a passage in the text and pick the object it refers to, or create one. The object joins the report and the text is highlighted.
3. To let a connector propose objects, click **Ask new mapping**. This needs an analysis connector: when none is available, the button is greyed out and an icon next to it says so.
4. When the analysis is complete, turn on **Show suggested mapping**. Remove the suggestions you do not want, then click **Validate** and confirm to add the others to the report.
:::

::screenshot{id=opencti-analysis-content-mapping alt="The content mapping view of a report: its text on the left, and on the right the objects it contains, each with its mapping count, under the Ask new mapping and Clear mappings buttons."}

:::example[Add the backdoor that a report names but does not hold]
The text of the report **Velvet Harbor targets hotel booking systems** says that Velvet Loader fetches HarborGate backdoor from 203.0.113.10, but the report does not contain the malware **HarborGate backdoor** yet. In **Content mapping view**, select the words `HarborGate backdoor` in the text and pick the existing malware **HarborGate backdoor**.

The words are highlighted, and the malware joins the report: it now shows in the **Entities** tab, in the graph, and in the **Analyses** tab of the malware. Analysts who start from the malware find this report.
:::

**Clear mappings** removes the highlights only: the objects stay in the report. Content mapping is available on reports, groupings and cases.

## Keep an investigation together in a grouping

A grouping holds objects that share a context before you can say what they mean, for example the domains and addresses of a phishing wave you are still analyzing. Its **Context** says why they are together: `suspicious-activity`, `malware-analysis` or `unspecified`. When the analysis is finished, write a report from it. A grouping has the same tabs as a report, and its **Knowledge** the same views except the timeline.

## Record your assessment in notes and opinions

Notes and opinions let any analyst with **Access to collaborative creation** add an assessment to an object or a relationship, without changing the object itself.

- **A note** adds free text. On the **Overview** of any object, click **Write a note** under **Notes about this entity**. Write the **Content**, and optionally an **Abstract**, one or more **Note types** (`analysis`, `assessment`, `external`, `feedback`, `internal`) and a **Likelihood** from 0 to 100, how likely you think what the note says is. :menu[Analyses > Notes] lists every note for review; a note page has **Overview**, **Data** and **History**.
- **An opinion** says how much you agree with an object, from `strongly-disagree` to `strongly-agree`, with an **Explanation**. Click the button next to **Distribution of opinions** on the **Overview** to give yours or change it: each analyst has one opinion per object. The chart shows how the opinions of the team spread.

:::example[Flag a weak attribution]
A single source links the malware **Harbor Stealer** to Velvet Harbor, with a confidence level of 30. Open the relationship from the **Knowledge** tab of Velvet Harbor and write a note: **Abstract** `Single-source link`, **Note types** `assessment`, **Likelihood** 30, and in **Content** why one code overlap is not enough. Readers of the relationship see your caveat next to it, and the note shows in :menu[Analyses > Notes] for your team lead to review.
:::

## Trace a fact to its source

An external reference points to a document outside OpenCTI: a vendor report, a blog post, a MITRE page. It has a **Source name**, an **External ID** and a **URL**. **Browse the link** opens the URL after a confirmation, since the page is outside the platform. The page of an external reference lists its **Linked objects**, every object that cites it, and the files attached to it. **Launch an import** runs an import connector on such a file, and **Enrichment** sends the reference to an enrichment connector that can fetch the document.

## Read a sandbox result

A malware analysis records what a sandbox or an analyst found on a sample. Its **Details** give the **Product** that ran it, the **Report name**, the **Maliciousness** (`malicious`, `suspicious`, `benign` or `unknown`), the **Submission date**, when the **Analysis started** and **Analysis ended**, and the **Modules** used. When you record an analysis by hand, list them in **Modules (1 / line)**, one module per line. The **Version of the product**, **Configuration version**, **Analysis engine version** and **Analysis definition version** tell you whether two results are comparable. Below them come the sample analyzed and the observables found during the analysis. Its **Knowledge** tab lists the malware the analysis characterizes and the other objects linked to it. Malware analyses usually come from an enrichment connector, and the observables they cover list them in their **Malware analyses** panel.

## Restrict who sees a report

Two actions in the actions menu (⋮) of a report, grouping or case limit its audience:

- **Share with an organization** gives users of another organization access to the container and its content, when organization segregation is on. See [Separate data between organizations](/opencti/administer/organizations/).
- **Manage access restriction** restricts the container to chosen users, groups or organizations, with a level of access for each. See [Sharing models](/opencti/administer/sharing/).

Markings stay the first control: a reader who lacks the marking of a report, such as TLP:AMBER, does not see it at all.

## What's next

- [Read the home dashboard and AI summaries](/opencti/use/explore/insights/) for an AI summary of a report.
- [Work with graphs](/opencti/use/analyze/graphs/) to go further in the graph view.
- [Create knowledge manually](/opencti/use/ingest/manual/) to write your own report.
- [Explore cases](/opencti/use/explore/cases/), the other kind of container.
- The [data model reference](/reference/opencti/data-model/#analyses) for the attributes and relationships of each type of the Analyses menu.
