---
title: Pivot and investigate
description: Start an OpenCTI investigation from a few entities, expand it along their relationships, enrich them, roll back an expansion, and turn the findings into a report or a case.
sidebar_position: 2
doc_type: how-to
products:
  - opencti
personas: [soc-analyst, cti-analyst]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/pivoting/
---

Start from the domain or the IP address of an alert and see, in a few double-clicks, which infrastructure, malware and threat it connects to. An investigation is a private graph workspace for this: you add a few entities, expand them along their relationships, roll back what leads nowhere, and move what you found into a case or a report. This page walks you through a full investigation; [Work with graphs](/opencti/use/analyze/graphs/) covers the graph controls themselves.

## Before you start

- Your role has the **Access investigations** capability to open investigations, **Create / Update investigations** to create and change them, and **Delete investigations** to delete them.
- An investigation is visible only to its creator until they share it. To share it, open its menu and choose **Manage access restriction**, as for [dashboards](/opencti/use/dashboards/share/).
- Creating a relationship inside an investigation writes it to the knowledge base, so it also needs **Create / Update knowledge**.

## Start an investigation

:::steps

1. Open :menu[Investigations] and click **Create Investigation**. Give it a name and, optionally, a description.
2. Click **Add an entity to this investigation** and pick the entities to start from, such as a suspicious domain name and the intrusion set it was reported with.

   :::only{opencti=">=7.261117"}
   A new investigation opens on an empty graph that says **Nothing to draw yet**. Its **Add entities** button opens the same **Add entities** panel as **Add an entity to this investigation** in the toolbar.

   ::screenshot{id=opencti-pivot-empty alt="A new investigation with nothing drawn yet: the card in the middle of the graph says to add entities and expand them, with the Read the documentation and Add entities buttons."}
   :::

   ::screenshot{id=opencti-pivot-workspace alt="An investigation graph with a domain name and an intrusion set, each with a counter of relationships not drawn yet, and the side panel showing the details of the selected entity."}

:::

You can also start from the graph of a container. Select entities, right-click one of them and choose **Start an investigation**: the investigation opens with the selected entities. The **Start an investigation** button in the page header starts one with every entity of the container instead.

## Expand along relationships

A counter on a node says how many of its relationships are not drawn yet. The number is an estimate. No counter means there is nothing to expand.

:::steps

1. Select one or more entities, then open the expansion dialog: click **Expand** in the toolbar, double-click an entity, or right-click it and choose **Expand this entity**.
2. Choose the target types and relationship types to bring in, for example **Malware** through **uses** relationships, then click **Expand**. Leave **All types of target** and **All types of relationship** to bring in everything.

   ::screenshot{id=opencti-pivot-expand alt="The expansion dialog listing target entity types and relationship types with their counts, and the Expand button."}

   :::only{opencti=">=7.261119"}
   **Reset filters**, on by default, clears the filters of the graph toolbar once the expansion is done, so that no filter hides what it brought in. Turn it off to keep your filters. The **Enrichment Results** dialog of an enrichment has the same switch.
   :::

3. Repeat on the new entities that look relevant. Click an entity to read its details in the side panel without leaving the graph.

:::

### Roll back an expansion

An expansion can bring in more than you need. Click the roll back button in the toolbar to return the graph to the state before the last expansion. The dialog **Revert to Pre-Expansion State** shows the date of that expansion.

:::warning
Rolling back also undoes every entity you added or removed since that expansion. The history lives in your browser tab and keeps the last 20 operations, so close the tab and it is gone.
:::

::::only{opencti=">=7.261109"}
## Enrich from the investigation

An enrichment connector, such as VirusTotal, adds what an outside source knows about an entity. Run it from the investigation, then choose which of its results join the graph: the rest stays in the knowledge base without cluttering your graph.

Launching an enrichment needs edit access to the investigation, **Create / Update investigations**, **Ask for knowledge enrichment** and **Create / Update knowledge**. The review buttons of a run, and **Review all**, show only when your role also has **Connectors API usage: register, ping, export push ...** or **Manage connector state**: if you do not see them, ask your administrator.

:::steps

1. Select the entities to enrich, all of the same type, then click **Enrichment** in the toolbar, or right-click the selection and choose **Enrichment**. Only the connectors that accept that type are listed. A selection mixing types, or a type no connector accepts, shows **No connector available for the selected entities.**
2. In **Start an enrichment**, pick one or more connectors and click **Enrich**. The enrichment starts at once, as a background task. Picking a connector in the **Enrichment** entry of the right-click menu starts it the same way. A selection of more than 1,000 entities first opens **Launch a background task**: click **Launch** to confirm.
3. Follow the runs in the **Enrichment** drawer: **In progress**, then **To review** when the results are ready, then **Done**. The **Enrichment** button of the toolbar shows how many enrichments are in progress or have results waiting.
4. In **To review**, choose for each run:
   - **Review results**: the **Enrichment Results** dialog lists only the data the enrichment created or updated. Pick what to bring in and click **Expand**.
   - **Add all to graph**: everything the run found joins the graph, after you confirm.
   - **Mark as reviewed**: nothing joins the graph, and the run moves to **Done**.

:::

::screenshot{id=opencti-pivot-enrichment-drawer alt="The Enrichment drawer of the investigation Velvet Harbor pivot: the Start an enrichment card offers VirusTotal for the selection, and the To review tab holds VirusTotal's run on booking-velvetharbor.example with its Review results, Add all to graph and Mark as reviewed buttons."}

**Review all** opens the results of every run waiting for review. A run whose enrichment found nothing new says **Nothing to add to the graph**. **Refresh the knowledge using this connector**, beside a connector in the drawer, runs it again on the same entity.

:::example[Enrich the look-alike booking domain with VirusTotal]
Your investigation `Velvet Harbor pivot` holds the domain `booking-velvetharbor.example` and the intrusion set **Velvet Harbor**. Before you search the proxy logs, you want what VirusTotal knows of the domain, without opening every result one by one.

1. Click the domain in the graph, then **Enrichment** in the toolbar. In **Start an enrichment**, pick **VirusTotal**, the only connector that accepts a domain here, click **Enrich**.
2. Once the run is in **To review**, the toolbar button says **1 enrichment has results waiting**. Click **Add all to graph** beside VirusTotal's run, then **Add**.

The graph gains a second address the domain resolves to, `203.0.113.82`, beside `203.0.113.10`. It also gains the domain's indicator, now scored from Google Threat Intelligence's verdict, and the malware Harbor Stealer, the campaign Operation Night Desk and a report that VirusTotal ties to the domain. VirusTotal's two notes on the domain, its external references and VirusTotal itself, their author, join too. The run moves to **Done**, and the new address goes into your log search with the old one.
:::

::screenshot{id=opencti-pivot-enrichment-added alt="The graph of the investigation Velvet Harbor pivot after Add all to graph, in its hierarchical layout: booking-velvetharbor.example now links to 203.0.113.82 and 203.0.113.10, Harbor Stealer, Velvet Harbor and Operation Night Desk, under the domain's indicator, VirusTotal's report and notes, and VirusTotal's external references."}
::::

::::only{opencti=">=7.261119"}
## See the techniques on the ATT&CK matrix

Once the graph holds attack patterns, the **Matrix view** places them on the MITRE ATT&CK matrix, and its heat map shows which techniques come back most often among the entities you investigate. Use it to tell your detection team which techniques to hunt for first.

The **Graph view** and **Matrix view** buttons sit in the page header, left of **Update**. They show only when your role has **Access knowledge**: without it, the investigation stays a graph.

:::steps

1. Click **Matrix view**. The matrix opens on the `mitre-attack` kill chain when the platform has it; **Kill chain :** picks another one. The techniques of the investigation are highlighted, labeled with their ATT&CK ID, such as `T1059.001 - PowerShell`.
2. Click **Display only used techniques** to hide the techniques that are not in the investigation. Click it again, now **Display the whole matrix**, to bring them back.
3. Click **Displays heat map of attack pattern use**. Each technique is colored by the number of entities of the investigation that use it, from yellow to red, under the legend **Technique usage (low → high)**. A technique whose sub-techniques are used, but not itself, gets a colored border only.
4. Hover a technique to read its **Linked entities**, the entities of the investigation that count for it.
5. To see what your defenses should catch, pick your security platforms in **Compare with my security posture**, as on a threat's matrix (see [Match threats with your security platforms](/opencti/use/explore/entities/#match-threats-with-your-security-platforms)).

:::

::screenshot{id=opencti-pivot-matrix alt="The Matrix view of the investigation Velvet Harbor techniques, with only used techniques shown and the heat map on: PowerShell and Web Protocols in red, used by two entities each, the other techniques in yellow, under the legend Technique usage from 1 to 2."}

An entity counts for a technique when the investigation also holds the relationship between them: a **uses** relationship in either direction, an indicator that **indicates** the technique, or a container that contains it. Each entity counts once, and the colors scale to the most used technique of the selected kill chain. The heat map stays off until at least one technique has such an entity.

The matrix reads the whole investigation: the filters of the graph toolbar do not apply, and the matrix says **Graph filters don't apply to the matrix** while some are on. An attack pattern that has no phase in the selected kill chain is not drawn, and a line above the matrix counts it. With no attack pattern in the investigation, the matrix says **Add attack patterns to the graph view to visualise them here**: expand a threat or a malware with **Attack pattern** as target type, then come back. Your choices in the matrix toolbar last until you leave the investigation.

In **Matrix view**, **Export to image** and **Export to PDF** in the page header save the matrix, in a file named after the investigation followed by `Tactics matrix`. They require **Can use web interface export functions (PDF, PNG, etc.)**.

:::example[Tell the SOC which Velvet Harbor techniques to hunt first]
Your investigation `Velvet Harbor techniques` holds the intrusion set **Velvet Harbor**, its malware Velvet Loader and Harbor Stealer, and the seven techniques they use. Before the weekly SOC briefing, you want the techniques that come back most.

1. Open the investigation and click **Matrix view**. The `mitre-attack` kill chain shows, with the seven techniques highlighted.
2. Click **Display only used techniques**, then **Displays heat map of attack pattern use**. The legend goes from 1 to 2.
3. Hover `T1059.001 - PowerShell`: **Linked entities (2)** lists Velvet Harbor and Velvet Loader. `T1071.001 - Web Protocols` is used by Velvet Harbor and Harbor Stealer.
4. Click **Export to image** in the page header, then **Dark (with background)**.

The image `Velvet Harbor techniques - Tactics matrix` goes into the briefing: PowerShell and web traffic to the command and control servers come first in the hunt, ahead of the phishing techniques only Velvet Harbor uses.
:::
::::

## Capitalize on the investigation

When the graph tells a story worth keeping, keep it:

- **Add it to a container.** Open the investigation menu, choose **Add to a container**, then pick an existing container or create one: a report, a grouping, an incident response, a request for information or a request for takedown. Every entity and relationship of the investigation goes into the container, and OpenCTI opens the container's knowledge graph.
- **Export it.** The page header exports a PDF of the visible area, or a STIX 2.1 bundle in which all objects of the investigation are wrapped in a report. The graph toolbar exports a high-resolution image.

:::only{opencti=">=7.261030"}
You can also **duplicate it** to try another line of inquiry without losing this one: in its menu, choose **Duplicate**, enter a **New name** in **Duplicate the investigation**, and click **Duplicate**.
:::

:::example[Pivot from a look-alike booking domain]
A member hotel's proxy reports visits to `booking-velvetharbor.example`. Before you search the logs, you want every address and malware family tied to that domain, so that one search covers them all.

1. In :menu[Investigations], click **Create Investigation** and name it `Velvet Harbor pivot`, with the description `Pivot from the look-alike booking domain.`
2. Click **Add an entity to this investigation** and add the domain `booking-velvetharbor.example` and the intrusion set **Velvet Harbor**.
3. Double-click the domain. In the expansion dialog, choose **IPv4 address** as target type and click **Expand**: `203.0.113.10` appears through a **resolves to** relationship.
4. Double-click Velvet Harbor, choose **Malware** and **Infrastructure** as target types and **uses** as relationship type, then click **Expand**. Velvet Loader, Harbor Stealer, HarborGate backdoor and the infrastructure Velvet Harbor C2 appear.
5. Open the investigation menu, choose **Add to a container**, and pick the incident response `Sighting of booking-velvetharbor.example`.

The case now holds the domain, the address it resolves to, the intrusion set and its malware, so you search the proxy and EDR logs for `203.0.113.10` and the three malware families at once.
:::

::screenshot{id=opencti-pivot-add-to-container alt="The Add to container dialog of an investigation, with a container type selector and the choice between an existing container and a new one."}

## What's next

- [Work with graphs](/opencti/use/analyze/graphs/), for layouts, filters, selection and shortcuts.
- [Handle incidents and requests in cases](/opencti/use/analyze/cases/), to act on what the investigation found.
- [Containers](/opencti/concepts/containers/), to choose between a report, a grouping and a case.
