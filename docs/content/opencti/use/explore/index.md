---
title: Explore knowledge
description: How the OpenCTI knowledge menus are organized, and how to read any entity page, from what it is and who says so to its links, sources, files and history.
sidebar_position: 1
doc_type: explanation
products:
  - opencti
personas: [cti-analyst, evaluator]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/overview/
---

OpenCTI organizes its knowledge in nine menus of the left navigation, from the reports and cases your team works on to the threats, victims and techniques they refer to. Every object, whatever its menu, opens on the same kind of entity page. It shows what the object is and who says so, what it is linked to, where that knowledge comes from, and what changed. This page explains that page once; the pages of each menu only add what is specific to their types.

You see a knowledge menu when your role has the **Access knowledge** capability, and a list only shows the objects your marking definitions allow. The home page, **Home**, is a dashboard described in [Read the home dashboard and AI summaries](/opencti/use/explore/insights/).

## How the menus are organized

The menus come in two groups. The first holds the knowledge that arrives every day and needs work. The second is the reference library that gives it context.

::diagram{id=opencti-explore-menus}

| Menu | What it holds | Page |
| --- | --- | --- |
| **Analyses** | Reports, groupings, malware analyses, security coverages, notes and external references | [Explore analysis](/opencti/use/explore/analysis/) |
| **Cases** | Incident responses, requests for information and for takedown, tasks and feedbacks | [Explore cases](/opencti/use/explore/cases/) |
| **Events** | Incidents, sightings and observed data | [Explore events](/opencti/use/explore/events/) |
| **Observations** | Observables, artifacts, indicators and infrastructures | [Explore observations](/opencti/use/explore/observations/) |
| **Threats** | Threat actors (group and individual), intrusion sets and campaigns | [Explore threats](/opencti/use/explore/threats/) |
| **Arsenal** | Malware, channels, tools and vulnerabilities | [Explore arsenal](/opencti/use/explore/arsenal/) |
| **Techniques** | Attack patterns, narratives, courses of action, data components and data sources | [Explore techniques](/opencti/use/explore/techniques/) |
| **Entities** | Sectors, events, organizations, security platforms, systems and individuals | [Explore entities](/opencti/use/explore/entities/) |
| **Locations** | Regions, countries, administrative areas, cities and positions | [Explore locations](/opencti/use/explore/locations/) |

The [data model reference](/reference/opencti/data-model/) lists the types of each menu with their attributes and the relationships they can start.

Each menu opens on its first entry, for example :menu[Analyses > Reports]. To find an object without knowing its menu, use the search bar at the top of every screen: see [Search for knowledge](/opencti/use/explore/search/).

## Lists

A list shows the objects of one type that you can access. From a list you can:

- **Search** with the search box above the list. The keyword also applies when you export the list.
- **Filter** on any attribute, relationship or date, and combine filters. **Clear filters** removes them all. The [filter key reference](/reference/opencti/filter-keys/) lists every key and operator a filter accepts.
- **Sort** by clicking a column header. Columns computed when the list loads, such as **Author**, **Creators** or **Marking**, are sortable only when your search engine supports runtime fields: Elasticsearch 7.12 or later, not OpenSearch. **Labels** are never sortable.
- **Switch the view.** Threat actors, intrusion sets, campaigns and malware open in **Cards view**, where each card summarizes aliases, used malware and targets; **Lines view** shows them as a table. **Bookmark this item** pins a card to the top of the list and to the bookmarks widget of your dashboards.
- **Export** the list, filtered as it is, to a file. See [Export knowledge to a file](/opencti/use/share/export/).
- **Act on several objects.** Select lines with their checkboxes to update, merge, enrich, share or delete them in one go. See [Change many objects at once](/opencti/use/automate/background-tasks/).
- **Create** an object with the button at the top right, when your role has **Create / Update knowledge**. See [Create knowledge manually](/opencti/use/ingest/manual/).

## Read an entity page

Click an object in a list to open its page. It opens on **Overview**, and each tab answers one question you ask of any piece of knowledge:

| Your question | Where to look |
| --- | --- |
| What is it, who says so, how sure are they, who may see it? | [Overview](#overview) |
| What is it linked to, and how strong is each link? | [Knowledge](#knowledge) |
| Which reports and cases say so? | [Analyses](#analyses), and the sources of each relationship |
| Where was it seen? | [Sightings](#sightings) |
| What have we written about it? | [Content](#content) |
| Which files come with it? | [Data](#data) |
| Who changed what, and when? | [History](#history) |

::screenshot{id=opencti-explore-entity-page alt="The page of an intrusion set: its name and actions at the top, then the tabs Overview, Knowledge, Content, Analyses, Data and History, and the Details and Basic information panels."}

In OpenCTI, the object holds what is true of it alone, such as the motivation of an intrusion set. Everything else is a relationship: that it targets a sector, uses a malware or is attributed to a group. Each relationship is a piece of knowledge in its own right, with its own author, confidence, markings and sources. This is why the **Knowledge** tab, and the page of each relationship, matter as much as the object itself.

### Header actions

The header names the object, lists its aliases (**add alias** adds one) and groups the actions you are allowed to run:

| Action | What it does |
| --- | --- |
| Export | Exports the object to a file |
| **Add in container** | Adds the object to a report, a grouping or a case |
| Subscribe (bell) | Notifies you when the object changes. See [Get alerted when knowledge changes](/opencti/use/automate/notifications/) |
| **Create Relationship** | Links the object to another one |
| **Update** | Opens the edit panel |
| **Share with an organization** | Shares the object with another organization of the platform. See [Separate data between organizations](/opencti/administer/organizations/) |
| **Enrichment** | Sends the object to an enrichment connector. See [Enrich knowledge from external sources](/opencti/use/enrich/connectors/) |
| **Enroll in playbook** | Runs a playbook on the object. See [Create a playbook](/opencti/use/automate/playbooks/create/) |
| **Delete** | Deletes the object. See [Delete and restore knowledge](/opencti/use/delete-restore/) |

The last four sit in the actions menu (⋮). Containers add **Manage access restriction** to the same menu, to limit who can see them: see [Sharing models](/opencti/administer/sharing/). Changing knowledge requires **Create / Update knowledge**, and deleting it requires **Delete knowledge**.

### Tabs

Each type shows the tabs that make sense for it, in this order:

| Tab | Shows | Present on |
| --- | --- | --- |
| [Overview](#overview) | Properties, latest relationships and containers, external references, history, notes | Every type |
| [Knowledge](#knowledge) | Everything linked to the object, or everything a container holds | Most types |
| [Content](#content) | Documents written or attached in the platform | Most types |
| [Analyses](#analyses) | Reports, groupings and cases that contain the object | Threats, arsenal, attack patterns, narratives, entities, locations, observables, indicators, infrastructures, incidents |
| [Sightings](#sightings) | Where the object, or an indicator about it, was seen | Observables, artifacts, indicators, entities, locations |
| [Entities, Observables, Relationships](#entities-observables-and-relationships) | Tables of what a container holds | Reports, groupings, cases, observed data |
| [Data](#data) | Files, exports, workbenches and drafts | Every type |
| [History](#history) | Every change to the object and its relationships | Every type except containers, whose history is in **Data** |

:::only{opencti=">=7.260811"}
Administrators can add [custom views](/opencti/use/dashboards/custom-views/) to an entity type. They appear as extra tabs, and the default one can replace **Overview** as the tab the page opens on. When a type has several views besides the default one, they are grouped in a **Custom view** menu.
:::

### Overview

**Overview** gathers what you need to qualify the object at a glance:

- **Details**: the properties specific to the type, such as the motivation of an intrusion set or the CVSS scores of a vulnerability. Containers call this panel **Entity details**. Pictures added in the **Data** tab show here as a carousel.
- **Basic information**: the properties every object shares.
- **Latest created relationships** and **Latest containers about the object**: what changed recently around the object, and the reports and cases that mention it.
- **External references**: links to the sources the object comes from.
- **Most recent history**: the last changes, as in the **History** tab.
- **Notes about this entity**: the free-text notes analysts wrote. **Write a note** adds one.

**Basic information** is where you judge the object before you use it:

| Field | What it tells you |
| --- | --- |
| **Marking** | Who may see the object, and how far you may share it |
| **Author** and **Reliability** | Who states this knowledge, and how reliable that source is. "(of author)" means the reliability comes from the author's identity |
| **Confidence level** | How credible the information itself is, on the scale your platform uses |
| **Distribution of opinions** | What analysts think of the object, from their opinions |
| **Original creation date**, **Modification date**, **Platform creation date** | When the source created and last changed the data, and when it reached your platform |
| **Processing status** | Where the object stands in your team's workflow |
| **Revoked** | Whether the source withdrew the object |
| **Labels**, **Creators**, **Standard STIX ID**, **Other STIX IDs** | Your tags, the users or connectors that created it, and its identifiers |

[Reliability and confidence](/opencti/concepts/reliability-confidence/) explains the two scales, and [Meaning of dates](/opencti/concepts/dates/) explains every date. Administrators can change the order and width of these cards for each type: see [Customize entity types](/opencti/configure/entities/#rearrange-the-overview).

Some types show buttons next to the tabs. **AI Insights** summarizes the object: see [Read the home dashboard and AI summaries](/opencti/use/explore/insights/). On intrusion sets, campaigns, incidents and containers, **Add Security coverage** tests whether your defenses detect them: see [Measure your defenses against a threat](/opencti/use/analyze/security-coverage/).

### Knowledge

For an entity, **Knowledge** lists everything linked to it, directly or through an [inference](/opencti/use/enrich/inferences/). For a report, a grouping or a case, it is where you build and read what the container holds.

#### The views of a container

The **Knowledge** of a report, a grouping or a case has views you switch with the icons at the top right:

:::only{opencti=">=7.261119"}
| View | Use it to | Containers |
| --- | --- | --- |
| **Graph view** | See and edit the objects and relationships: add an entity to the container, create relationships and sightings, filter by type or time range | All |
| **TimeLine view** | Follow an attack or a campaign in time, when the container dates its knowledge | Reports and cases, not groupings |
| **Correlation view** | Find the other containers that share its indicators and observables, or with **Show all correlated entities** any of its objects, such as a malware or an intrusion set | All |
| **Matrix view** | See the techniques of the container on the MITRE ATT&CK matrix, with the options of the [attack patterns](#attack-patterns) of a threat | All |
:::

:::only{opencti="<7.261119"}
| View | Use it to | Containers |
| --- | --- | --- |
| **Graph view** | See and edit the objects and relationships: add an entity to the container, create relationships and sightings, filter by type or time range | All |
| **TimeLine view** | Follow an attack or a campaign in time, when the container dates its knowledge | Reports and cases, not groupings |
| **Correlation view** | Find the other containers that share its indicators and observables, or with **Show all correlated entities** any of its objects, such as a malware or an intrusion set | All |
| **Tactics matrix view** | See the techniques of the container on the MITRE ATT&CK matrix, with the options of the [attack patterns](#attack-patterns) of a threat | All |
:::

Reading a report through these views, with the Velvet Harbor example, is in [Explore analysis](/opencti/use/explore/analysis/#read-a-report-in-its-knowledge-tab); working a case through them, in [Handle incidents and requests in cases](/opencti/use/analyze/cases/).

#### The side menu

The menu on the right splits the knowledge by type of linked object, with the number of relationships in parentheses: for an intrusion set, **Attribution (1)**, **Victimology (6)**, **Malware (3)**, **Attack patterns (5)** and so on. The number counts relationships, not objects: two campaigns linked by three relationships show **Campaigns (3)**. An entry without a number has nothing yet. **All** lists every relationship in one table. **Related entities** lists the objects linked by a `related-to` relationship, the link you use when you know two objects are connected but not how.

#### The knowledge overview

The first entry, **Overview**, summarizes the knowledge. For threats, malware, tools, channels, infrastructures and incidents, it shows:

- Three counters: **Total analyses** (the containers that mention the object), **Total indicators** (the indicators that point to it; **Total observables** for an incident) and **Total relations**. Each shows how much it grew over the last **30 days**: a quick sign that a threat is active again.
- **Distribution of reports**: the reports about the object, by author. One author means one source; many authors mean the knowledge is corroborated.
- **Diamond**: what is linked to the object, in the four quadrants of the diamond model. Adversary shows the **Aliases** and the **Last attributions**, the campaigns attributed to a threat, or **Last used by**, the threats that use a malware, a tool or a channel. Infrastructure, victimology and capabilities show the last IP addresses, domains, targets, attack patterns and malware. **View all** opens the matching list.
- **Timeline**: the relationships and sightings of the object, newest first.
- **Global kill chain**: the attack patterns the object uses, placed on the phases of their kill chains.

On **Timeline** and **Global kill chain**, you can filter the relationships, and the settings icon (**Open time field**) offers two options:

- **Date reference** orders the timeline by **Technical date** (the default), the day the relationship reached your platform. **Functional date** orders it by **Start time**, the day the fact began, as the source states it. Use the functional date to follow what the threat did, and the technical date to see what you learned recently.
- **Display nested relationships** adds the references stored inside objects, such as the reports that contain the object. See [Nested objects](/opencti/concepts/nested-objects/).

For victims such as sectors and countries, **Overview** shows charts of the reports, the main threats and the sightings about them instead.

#### Lists of linked objects

The other entries of the side menu list the linked objects. Most offer two views:

- **Entities view** lists the linked objects themselves: one line per malware, sector or attack pattern.
- **Relationships view** lists the relationships, with their **Start time**, **Stop time**, **Confidence** and author. Use it to see how strong and how recent each link is.

The **Indicators** and **Observables** entries add **Knowledge from related containers view**: the indicators or observables that appear in the same reports as the object, even when no relationship links them yet. It is a good place to find knowledge to link.

A wand icon at the end of a line marks an inferred relationship, created by a rule rather than by a source. To link the object to something new, use **Create Relationship** in the header: in **Create a relationship**, **Reverse** swaps the source and the target when the relationship reads the wrong way. You can pick several targets at once: the form then shows **Multiple entities selected** instead of a name, and creates one relationship to each target with the same type, dates, confidence and markings. See [Create knowledge manually](/opencti/use/ingest/manual/#create-a-relationship).

#### Victimology

**Victimology** shows who the object targets. Three charts count the targeted sectors, regions and countries (**Victimology (sectors)**, **Victimology (regions)**, **Victimology (countries)**), and a map shows the countries. Below, the list of targets adds **Nested view**, which groups the targeted organizations and individuals under their sector. Under each sector, **Direct targeting of this sector** is the relationship to the sector itself; the other lines are its organizations and individuals. An inferred line names its rule when you hover its icon: a threat that targets a hotel chain, part of the hospitality sector, is inferred to target the sector.

#### Attack patterns

**Attack patterns** shows how the object operates, in five views:

| View | Use it to |
| --- | --- |
| **Matrix view** | See the techniques on a matrix such as MITRE ATT&CK. **Kill chain** picks the matrix, **Display only used techniques** hides the empty cells (**Display the whole matrix** shows them again), **Expand all** opens the sub-techniques |
| **Matrix in line view** | List the techniques in a table, with their kill chain phase and ATT&CK ID, to sort, filter and export |
| **Kill chain view** | List the techniques phase by phase, each a link to its attack pattern |
| **Courses of action view** | List the mitigations of each technique, to tell defenders what to put in place |
| **Relationships view** | List the `uses` relationships themselves, with their author, to open the one you want to check |

In **Matrix view**, **Compare with my security posture** takes one or more security platforms from :menu[Entities > Security platforms]. Each technique then shows whether the selected platforms should cover it, from their `should-cover` relationships, so you see at once which techniques of a threat your defenses miss. **Export to image** and **Export to PDF** save the current view for a briefing, and **Export to CSV**, in **Courses of action view**, saves the mitigations to hand to defenders. These buttons require **Can use web interface export functions (PDF, PNG, etc.)**.

#### The page of a relationship

Click a line in a list to open the relationship. Its page reads like a statement: the source object, the relationship type, the target object. It shows:

- **Description**: why the source states the link.
- **Start time** and **Stop time**: when the link was true, such as the months an intrusion set used a malware. Empty when the source does not say.
- **Kill chain phases**, for a `uses` relationship to an attack pattern.
- **Marking**, **Organizations sharing**, **Confidence level** and **Author**: who may see the link, and how much to trust it.
- **Linked entities**: the objects linked to the relationship itself rather than to either end, such as an observable that supports it. **Add relationship** links more.
- **Latest containers about the object**, **External references** and **Notes about this relationship**: the sources of the link, and what analysts wrote about it.

:::only{opencti=">=7.261109"}
In the **Create a relationship** panel that **Add relationship** opens, select one or several objects in the table, which you can search and filter, then click **Continue** and choose the relationship type: OpenCTI creates one relationship per object, and offers only the types valid for all of them. **Create an entity** and **Create an observable** add an object that does not exist yet.
:::

An object you are not allowed to see shows as **Restricted**: the relationship exists, but its marking keeps the other end from you. An inferred relationship shows **Inferred knowledge** as its description and an **Inference explanation** that draws the facts and the rule behind it. It cannot be edited: change the facts instead. See [How OpenCTI infers new relationships](/opencti/use/enrich/inferences/).

::screenshot{id=opencti-explore-relationship-page alt="The page of the relationship Velvet Harbor uses Harbor Stealer: its description says a single source links them, its confidence level is low, and the report it comes from is listed under Latest containers about the object."}

### Content

**Content** holds the deliverables you write about the object, such as a briefing for your stakeholders, and the files you attach. The panel on the right lists them:

- **Files**: the documents of the object. The upload button attaches a file; **Add a file** creates an empty one in **HTML**, **Markdown** or **Text** to write in the platform.
- **Generated finished intelligence**: the documents produced from a FINTEL template of the type, a ready-made structure filled with the knowledge of the object. **Generate an export based on a template** creates one. This part requires the Enterprise Edition. See [Generate a finished intelligence document](/opencti/use/share/export/#generate-a-finished-intelligence-document).
- **Mappable content**, on reports, groupings and cases: **Description & Main content**, the text of the container itself. Its menu offers **Generate a PDF export**.

**Content view** previews the selected file, **Editor view** edits it, and **Save** keeps your changes. On a PDF or HTML file, **Disseminate** emails it to a dissemination list: it requires the Enterprise Edition and the **Disseminate files by email** capability. See [Send intelligence by email](/opencti/use/share/dissemination-lists/).

:::only{opencti=">=7.261119"}
**Files** lists the PDF, text, HTML, Markdown, PNG and JPEG files of the object, and its exports in PDF. A PNG or JPEG image, such as the screenshot of a phishing page, previews like a PDF: **Zoom in** and **Zoom out** resize it, and it cannot be edited. The menu of an image adds **Open in a new tab**, to see it at full size in the browser. A file that may be malicious, the file of an artifact or a program (`.exe`, `.dll`, `.pif`), is never opened: **Download file** and **Open in a new tab** ask you to choose between **Raw file** and **Encrypted archive**.
:::

Containers add **Content mapping view**, which turns text into structured knowledge: see [Explore analysis](/opencti/use/explore/analysis/#turn-a-report-s-text-into-knowledge).

:::only{opencti=">=7.260811"}
Markdown fields, here and in descriptions, accept images: paste them, drop them or use the image button of the editor. Use PNG, JPEG, GIF or WebP images: SVG images are not supported, and each image must not exceed 5 MB. The platform stores the images with the object when you save it, and refuses the save when the new images add up to more than 20 MB: save after every few large images. There is no limit on their number, but many or large images make saving, exporting and importing the object slower. In **Preview**, click an image to browse all the images of the text.
:::

### Analyses

**Analyses** lists the reports, groupings and cases that contain the object, so you can trace every piece of knowledge back to its sources. **Lines view** lists them; **Graph view** draws the knowledge of all those containers together.

### Sightings

**Sightings** lists the sightings that involve the object: for an observable or an indicator, where it was seen; for an organization, a sector or a location, what was seen there. A sighting usually comes from a security tool such as a SIEM or an EDR. Each one is a **True positive** or a **False positive**, as the analyst who checked it decided. **Create a sighting** records one by hand. See [Explore events](/opencti/use/explore/events/#sightings).

### Entities, Observables and Relationships

Containers list what they hold in three tables, with search and filters: **Entities** for STIX domain objects, **Observables** for STIX cyber observables, **Relationships** for the relationships. A line shows whether the object was added directly or by an [inference rule](/opencti/use/enrich/inferences/).

### Data

**Data** holds the files around the object:

- **Uploaded files**: documents attached to the object, such as the PDF of a report. **Launch an import of this file** extracts its knowledge with an import connector, and requires **Import knowledge**. A file that may be malicious, such as an executable or the file of an artifact, offers two downloads. Take the **Encrypted archive** (password `infected`) rather than the **Raw file**, unless you work in an isolated environment.
- **Exported files**: exports of the object. **Generate an export** starts one, of the object alone or with its first neighbors; **File marking definition levels** sets the marking of the file itself. Downloading an export requires **Download knowledge export**. See [Export knowledge to a file](/opencti/use/share/export/).
- **Analyst workbenches** and **Drafts**: work in progress started from a file, which contains the object by default. **Create a workbench** starts one. See [Review what a file import found](/opencti/use/ingest/workbench/) and [Drafts](/opencti/use/ingest/drafts/).
- **External references files**: files attached to the external references of the object.
- **Pictures Management**: the pictures of threat actors, intrusion sets, malware, tools, organizations and individuals. **Update a picture** sets its **Description** and whether it shows **In Carousel**, in the **Details** card. The first picture of the carousel, by **Order of Carousel**, also illustrates the card of the object in lists.

Attaching a file requires **Upload knowledge files**.

:::only{opencti=">=7.261119"}
A click on a file of **Uploaded files** or **Exported files** opens it in the **Content** tab, when that tab lists it: a PDF, text, HTML, Markdown, PNG or JPEG file you uploaded, or a PDF export. You read it next to the knowledge of the object, without downloading it. A click on another kind of file does nothing. On every line, **Open in a new tab** shows the file in the browser and **Download this file** saves it. The files of artifacts, notes and observed data are not opened in **Content**, and a program file (`.exe`, `.dll`, `.pif`) or the file of an artifact is never opened in the browser: you choose between **Raw file** and **Encrypted archive**.
:::

:::only{opencti=">=7.261030"}
When the type has a FINTEL template, **Generate an export** can produce a PDF from it: choose the **HTML content files to PDF** connector and keep **Export as fintel** on. **File to export** then reads **Generated file**: OpenCTI fills the template with the knowledge of the object rather than converting one of its files. See [Generate a finished intelligence document](/opencti/use/share/export/#generate-a-finished-intelligence-document).
:::

### History

**History** lists every change: **Entity** for the changes to the object itself, **Relations of the entity** for the relationships created or deleted around it. Click a line to open **Knowledge log details**, which shows each **Field** with what was **Added** and **Removed**. History is rebuilt from the platform event stream, so a change can take a few seconds to show.

:::only{opencti=">=7.261119"}
Each change fits on one line, cut when it is long: hover the line to read the whole message. A chevron at the end of the line marks a change with details to open. In **Knowledge log details**, a long description or content reads as plain text rather than as its formatting code.
:::

## Display an identity as an author

Organizations, individuals and systems are also the authors of knowledge. On their page, **Display as** switches between **Knowledge entity**, what is known about the identity, and **Author**, what it published. [Explore entities](/opencti/use/explore/entities/) explains both views and how to judge a source with them.

## Choose which menus people see

Hiding the types a team does not use shortens the menus. A menu disappears when all its types are hidden.

- For everyone: an administrator sets **Hidden entity types** in :menu[Settings > Parameters]. See [Set the platform parameters](/opencti/configure/parameters/).
- For some people: set hidden entity types on a group or an organization. See [Organize users in groups](/opencti/administer/groups/) and [Separate data between organizations](/opencti/administer/organizations/).

## What's next

- [Explore threats](/opencti/use/explore/threats/) to follow an actor, and [Explore arsenal](/opencti/use/explore/arsenal/) to know its malware and tools.
- [Search for knowledge](/opencti/use/explore/search/) across every menu at once.
- [Explore analysis](/opencti/use/explore/analysis/) to read reports and their knowledge.
- [Work with graphs](/opencti/use/analyze/graphs/) for the graph views of containers and investigations.
- [Data model](/opencti/concepts/data-model/) for the object types and how they relate.
