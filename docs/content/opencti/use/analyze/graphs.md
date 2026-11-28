---
title: Work with graphs
description: Read and act on the knowledge graphs of OpenCTI containers, investigations and entities, with focus, layouts, filters, selection and image export.
sidebar_position: 1
doc_type: how-to
products:
  - opencti
personas: [cti-analyst]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/graphs/
---

OpenCTI draws knowledge as a graph wherever connections matter more than lists: who uses what, who targets whom, and how sure the sources are. This page shows you how to read a graph, find what you need in it and act on its elements, whichever surface you open it from.

## Where you find graphs

| Surface | What it draws |
| --- | --- |
| **Knowledge** tab of a container (report, grouping, case), **Graph** view | The entities and relationships the container holds |
| **Knowledge** tab of a container, **Correlation view** | The observables and indicators of the container, linked to the other containers that share them |
| An investigation, in :menu[Investigations] | The entities you add and expand, see [Pivot and investigate](/opencti/use/analyze/pivot/) |
| **Analyses** tab of an entity, graph view | The containers the entity appears in, with the objects they share |
| An inferred relationship | A fixed drawing of the rule that produced it, see [How OpenCTI infers new relationships](/opencti/use/enrich/inferences/) |

All of them draw nodes, links and states the same way. The inferred relationship drawing is read-only: it has no toolbar, legend or menu.

## Before you start

- Your role has the **Access knowledge** capability.
- To add or remove elements, it also needs **Create / Update knowledge**. To export an image, **Can use web interface export functions (PDF, PNG, etc.)**.

:::only{opencti=">=7.261127"}

- In an investigation, the tools that change it (add, expand, link and remove entities) show only when you have the **can edit** access level on the investigation, or more, and your role has **Create / Update investigations** or **Create / Update dashboards**. Without them, you read, filter and arrange the graph, but the positions you drag are not saved. **Edit the selected item** needs **Create / Update knowledge** on every graph.

:::

::::only{opencti=">=7.261030"}

## Read the graph

Each entity is a node with the color and icon of its type and its name below. Above a node, up to three **badges** give its state, the most severe first:

| Badge | Meaning |
| --- | --- |
| Colored dot | The markings of the entity, in the color of its first marking, with a count when there are several |
| Gauge with a number | A confidence below 50 |
| Wand | The entity was created by the rules engine |

An entity you cannot see because of its markings or an organization restriction has a dashed outline and the name **Restricted**. A relationship that is itself the source or target of another relationship, such as a sighting of a relationship, is drawn as a smaller node.

Links end with an arrow on their target:

| Line | Meaning |
| --- | --- |
| Plain | A relationship stored in the knowledge base |
| Dashed, warning color | An inferred relationship |
| Dotted | A relationship with a confidence below 50 |

Details appear as you zoom in: icons, names, badges and relationship names. Hover an element, or select it, to keep it and its direct neighbors at full strength while the rest fades. After a moment, a hover card previews its type, dates, author, confidence, markings and relationship counts.

::screenshot{id=opencti-graphs-knowledge alt="The knowledge graph of a report: the intrusion set at the center, its targeted sector and used malware around it, the legend at the bottom left and the toolbar under the graph."}

The counters on the left of the toolbar give the size of the graph and what to check first: the entities and relationships drawn, the **restricted** entities you have no access to, and the entities that **need attention** because they carry a warning or an error badge, such as a low confidence. Click a counter to select what it counts. The **Badges** section of the legend works the same way: it lists the badges present in the graph, and a click selects the entities that carry one.

## Find what you need

:::steps

1. Type in the search field of the toolbar to select the matching entities. In the **Analyses** tab of an entity, the search field above the graph filters the containers instead.
2. Narrow the drawing with **Filter by type**, the marking and author filters, or **Time range**. Click a type in the legend to fade or restore it. **Clear all filters** brings everything back. In the **Correlation view**, choose between **Show all correlated entities** and **Show only correlated observables and indicators**.
3. Pick a layout in the toolbar. The default **Force-directed layout** lets related entities gather and saves the positions you drag. **Hierarchical layout (top to bottom)** and **Hierarchical layout (left to right)** follow the direction of relationships. **Layered layout (by entity category)** reads an attack from left to right: threats, arsenal, techniques, observables, victims, locations, then containers. **Radial layout around the selection** puts one entity at the center and the others on rings by distance.
4. Select what you want to look at: click with `Shift`, use **Box selection** or **Lasso selection**, then **Add neighbours to selection** to grow it.
5. To see how two entities connect, select both and click **Highlight shortest path between the two selected nodes**. The counters say how many paths there are and how many hops they take.

:::

To tidy a crowded graph, click **Group by type** next to a type in the legend: all its entities fold into one group node, and links to them are drawn once, with a count. **Hide** removes an entity from the drawing only; it stays in the container, and **Show the hidden entities** brings it back. Hidden entities are remembered in your browser for each graph.

:::example[Trace how Harbor Stealer reaches hotels]
You brief the SOC on the report "Velvet Harbor targets hotel booking systems" and want to show how the stealer is tied to the hospitality sector, and how sure the sources are.

1. Open the report in :menu[Analyses > Reports], then its **Knowledge** tab, **Graph** view.
2. In the toolbar, pick **Layered layout (by entity category)**: Velvet Harbor sits on the left with the threats, Velvet Loader and Harbor Stealer in the arsenal, the three attack patterns next, then the hospitality sector with the victims and France and Europe with the locations.
3. Click the Harbor Stealer node, `Shift`-click the Hospitality node, then click **Highlight shortest path between the two selected nodes**. The path goes through Velvet Harbor.
4. Look at the link between Velvet Harbor and Harbor Stealer: it is dotted, because a single source reported it, with a confidence of 30. The link to Velvet Loader is plain.
5. Click **Group by type** next to **Attack Pattern** in the legend to fold the three techniques into one node, then press `Shift` + `E` to export the image for the briefing.

The SOC sees at a glance that the loader is a well-sourced part of the attack on hotels, while the stealer still needs a second source.
:::

::screenshot{id=opencti-graphs-toolbar alt="The graph toolbar with its counters on the left, then the view, layout, selection, creation and filter groups, the search field and the More actions button on the right."}

## Act on an element

Right-click an entity, a relationship or the empty canvas, or press `Shift` + `F10`, to open the menu of what is under the pointer:

- On an entity: **Open in a new tab**, **Select entity and neighbours**, **Pin in place**, **Hide**, **Lay out the graph around it**, **Start an investigation**, and **Expand this entity** in investigations.
- On a selection: **Add neighbours to selection**, **Invert selection**, **Fit the selection**, **Hide**, **Create a relationship** between two selected entities, **Start an investigation** with the selection.
- On the empty canvas: **Select all nodes**, **Select by entity type**, **Clear selection**, **Show the hidden entities**, **Ungroup all**, **Unfix the nodes and re-apply forces**. The last one forgets the positions you dragged and leaves the hierarchical, layered or radial layout for the force-directed one.

The less used actions, and those the width of the window leaves no room for, are in **More actions** at the right end of the toolbar. In 3D mode, which has no right-click menu, the actions of the empty canvas are there too.

The creation tools of the toolbar change the knowledge, not only the drawing: **Add an entity to this container**, **Create a relationship**, **Create a nested relationship**, **Create a sighting**, **Edit the selected item** and **Remove selected items**. You can also draw a relationship by dragging from one entity to another with the right mouse button.

:::tip
Press `?` over a graph to list its keyboard shortcuts. The most used: `F` fits the whole graph, `N` adds neighbors to the selection, `P` highlights the shortest path, `H` hides the selection, `G` shows or folds the legend, `/` puts the cursor in the search field of the graph instead of the global search, `Shift` + `M` opens the graph in full screen, and `Esc` clears the selection or leaves full screen.
:::

## Export an image

Click **Export the whole graph as a high-resolution image** in the toolbar, or press `Shift` + `E`. You get the whole graph, not only the visible area, at print resolution, with a title and a legend. The PDF export and, for an investigation, the STIX export stay in the page header. On the timeline and matrix views of a container, which have no graph toolbar, **Export to image** in the page header exports the visible area.

The 3D mode, in the layout group, draws the same graph in three dimensions with the same filters and selection. Box and lasso selection, the legend and most layouts work in 2D only.

Every entity and relationship drawn is also listed for keyboard and screen reader users: press `Tab` to reach the list, the arrow keys to move in it, and `Enter` to select.

::::

::::only{opencti="<7.261030"}

## Use the graph

The toolbar at the bottom of the graph holds the view and editing tools:

- **Fit graph to canvas**, **Enable 3D mode**, and **Enable vertical tree mode** or **Enable horizontal tree mode** to change how the graph is laid out.
- **Filter entity types**, **Filter marking definitions** and **Display time range selector** to narrow the drawing.
- **Select by entity type** and **Select all nodes** to select, then the buttons on the right to add entities, create a relationship, a nested relationship or a sighting, edit or remove the selection.

Inferred relationships are drawn with dotted lines of a different color. Click an entity to see its details in the side panel, and double-click it in an investigation to expand it. The header of the page exports the graph as an image or a PDF.

:::example[Show only the threats and their victims]
You present the report "Velvet Harbor targets hotel booking systems" to the security leads of a hotel group, who care about who attacks and whom, not about techniques. Open the report, then its **Knowledge** tab, **Graph** view. Click **Filter entity types** and clear **Attack Pattern**, **IPv4 address**, **Domain name** and **Indicator**. The graph keeps Velvet Harbor, its malware, and the sector and countries it targets. Click **Enable horizontal tree mode** to read it from left to right, then export it from the page header.
:::

::::

## What's next

- [Pivot and investigate](/opencti/use/analyze/pivot/) from any entity with the investigation graph.
- [Explore analysis](/opencti/use/explore/analysis/), where the graph and correlation views of containers live.
- [How OpenCTI infers new relationships](/opencti/use/enrich/inferences/), to understand dashed links and the wand badge.
