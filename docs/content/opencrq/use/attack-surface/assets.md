---
title: Build the attack surface from your assets
description: Declare the supporting assets an attacker can reach, set their exposure and criticality, and link them to the primary assets they carry, so OpenCRQ prices attacks on your real estate.
sidebar_position: 1
doc_type: how-to
products: [opencrq]
personas: [risk-manager, admin]
owner: docs-opencrq
last_reviewed: 2026-11-04
---

OpenCRQ prices an attack on the estate it can reach: the applications, servers, workstations, network devices and accounts of your organization, which it calls **supporting assets**. This page builds that estate: you declare or import your supporting assets, set the two facts the price reads on each one, its exposure and its criticality, and link each asset to the primary assets it carries. If you are new to primary and supporting assets, read [Risk scopes, primary assets and supporting assets](/opencrq/concepts/risk-scopes-and-assets/) first.

## Before you start

- You are a member of the tenant. Every member works on its assets: no capability is needed.
- The risk scopes and their primary assets exist. See [Risk scopes, primary assets and supporting assets](/opencrq/concepts/risk-scopes-and-assets/).
- To import assets rather than declare them, the platform administrator gives the [OpenAEV](/opencrq/use/sources/openaev/) or [Microsoft Defender](/opencrq/use/sources/microsoft-defender/) connector its credentials in the deployment's configuration. OpenAEV then comes up connected; Microsoft Defender waits for an administrator with **Manage Security** to configure it in :menu[Integrations].

## What the price reads on an asset

Supporting assets belong to the tenant, not to a risk scope. A scope measured over the **Whole estate** walks every asset of the tenant; a scope measured over **Mapped assets only** walks the assets mapped into it. A scope that [declares its estate](/opencrq/concepts/risk-scopes-and-assets/#the-declared-estate) also counts the assets nobody listed, up to its declaration. The attack run reads three facts on each asset:

| Field | Values | What it changes |
| --- | --- | --- |
| **Category** and **Subcategory** | **Host**, **Web application**, **Network device**, **Identity**, **IoT / OT device**, **Cloud resource**, **SaaS application**, **Mobile device**, **Container / Workload**, **AI target**, **Security platform**, **Source code**, **Generic asset**; then a subcategory such as **Server**, **Workstation**, **Website**, **VPN gateway**, **User account**, **Point of sale**, **EDR**, **SIEM** or **Code repository** | The attack techniques that apply to the asset. Its **TTP** tab lists them as **Category TTPs**. A **Security platform**, a detection or prevention tool such as an EDR or a SIEM, stays in the inventory but has no technique and carries no loss: the price never counts it as a target |
| **Exposure** | **Internet Facing**, **Internal**, **Unknown** (the default) | Where an attack gets in. Exploiting a public service needs an internet-facing asset; phishing, drive-by compromise and stolen credentials do not look at exposure. Past the door, each step weighs an asset by its exposure, 1 internet-facing and 0.4 internal, mixed with its criticality. It also sets how reachable the findings on the asset are |
| **Criticality** | **Very High**, **High**, **Medium**, **Low**, **Unknown** (the default) | How much an asset weighs once the attacker is inside and moves toward the target: 1 for **Very High**, 0.9, 0.7 and 0.4 below, mixed with its exposure, and counting more the deeper the step. It measures the blast radius of the asset, not its business value, which the primary assets carry. It also raises the score of the findings on the asset |

An asset left on **Unknown** is priced as the tenant setting says: **Internal** exposure and **Low** criticality by default, under :menu[Settings > Engine > CRQ Indicators] (**Unknown exposure** and **Unknown criticality**). An unknown left on a server that faces the internet prices it as internal: set the fields you know.

## Declare a supporting asset

:::steps
1. Go to :menu[Assets]. The card above the list counts the assets of each category, the largest first, and the list shows each asset with its **Exposure**, **Criticality**, **Category TTPs** and **Findings**. On a short screen, the card shows its first three categories, and **Show all categories** opens the rest. Inside a risk scope, the card also sets the scope's [declared estate](/opencrq/concepts/risk-scopes-and-assets/#the-declared-estate) beside what is measured, and **Estimate my estate** (**Review my estimate** once the scope has one) leads to the risk scope to answer it.

   ::screenshot{id=opencrq-assets-list alt="The Assets list of Grand Meridian Hotels: six assets, from the internet-facing booking web application and VPN gateway to the internal database server, workstations, night.audit account and Spanish payment terminals, each with its exposure and criticality."}

2. Select **Create**. The **Create a new supporting asset** panel opens.
3. Enter the **Name** and, optionally, an **Owner** and a **Description**.
4. Choose the **Exposure**, the **Category** (**Host** by default) and the **Criticality**. Each category in the list says what it covers.
5. Choose a **Subcategory**, described in the list too, then fill the fields the category adds: a host takes a **Hostname**, a **Seen IP**, its **IP addresses**, a **Platform** and an **Operating system**; a web application takes its **URL**; an identity takes its **User principal name** and **Account name**.
6. Add **Your tags** if you sort your estate by tag, then select **Create**.

   ::screenshot{id=opencrq-assets-create alt="The Create a new supporting asset panel, with Exposure on Unknown, Category on Host and Criticality on Unknown, then the Subcategory and the first host fields, Hostname and Seen IP."}
:::

The asset opens on its **Overview**. Every value you set by hand carries the **Set by user** mark. When you create the asset from a risk scope's **Assets** page and that scope is measured over **Mapped assets only**, the asset is mapped into it too.

## Correct an imported asset

An integration sets the fields of the assets it imports, and updates them at each synchronization. To correct one, open the asset and select **Edit**: the value you set wins over the integration's and carries the **Set by user** mark, synchronization after synchronization. Each field's **What it means and how it is computed** button tells which rule or source the value comes from.

To hand the asset back to its integrations, select **Resync with connectors** beside **Edit**, then **Resync**. The button shows only when a value was set by hand and an integration still observes the asset. Every field value you set by hand goes back to what the integrations report, and the next synchronization can update it again. Tags you added by hand stay.

## Find the assets of a group

An asset group is a named set of assets, such as the servers of the head office. OpenCRQ does not create groups from its own screens: they come from your other tools.

- **The OpenAEV connector** mirrors the asset groups of OpenAEV when OpenAEV serves its snapshot export: once a day, as it reads OpenAEV's inventory, with their static members and the members a dynamic group's filter finds at that moment. A member OpenCRQ did not import is left out. A group deleted in OpenAEV is deleted in OpenCRQ once every OpenAEV tenant has answered that read, so an OpenAEV outage never empties your groups. See [Read the snapshot export of OpenAEV](/opencrq/use/sources/openaev/#read-the-snapshot-export-of-openaev).
- **XTM One, or a script**, puts an asset in groups by name when it writes the asset through OpenCRQ's API, and creates a group that does not exist yet.

Each source keeps its own memberships: a sync replaces only the members its connector added, never those XTM One or a person added.

:::steps
1. Go to :menu[Assets]. The **Groups** column shows the groups of each asset on one line, with **+N** for the rest.
2. Open the **Group** filter, shown beside **Source** and **Category** once the tenant has groups, and choose a group: each one shows its number of assets. The filter then reads the group's name, such as **Group: Hotel front desks**. **All groups** shows the whole list again.

   ::screenshot{id=opencrq-assets-groups alt="The Assets list of Grand Meridian Hotels filtered on the Hotel front desks group: the front-desk workstations, the night.audit account and the Spanish payment terminals, each with Hotel front desks in its Groups column."}
:::

The group filter combines with the search and the other filters: **Source** keeps the assets one integration observed, or, with **Set by user**, those declared by hand; **Category** keeps one category, or, under **Match**, every category **Except** that one. The count of active filters shows after them, with **Clear all** to lift every filter and the search at once. When nothing matches, the list reads **No results**, with **Clear filters**. The filters stay in the page's address, so you can share the filtered list.

:::example[Review the front desks before the awareness campaign]
Grand Meridian's risk manager prepares the front desks' training budget. In :menu[Assets], she opens the **Group** filter and chooses **Hotel front desks (3)**. The list shows three assets: the Front-desk workstations, the night.audit account and the Front-desk payment terminals of Spain. Two of them have a **Criticality** of **High**: those are the assets she names in the training request.
:::

## Link the asset to what it supports

A primary asset is lost through the supporting assets that carry it. The links say which ones, and with which role.

:::steps
1. Open the asset and its **Business dependencies** tab. The graph shows the primary assets the asset supports, the assets it depends on and those it serves, with counters of the critical, exposed and unowned assets in the chain.
2. Select **Manage**. The **Manage dependencies of supporting asset** panel opens on **Depends on**, beside **Primary assets** and **Provides to**, each tab with its count of links.
3. Open **Primary assets** and pick one or more roles for each primary asset the asset serves, such as **Stores**, **Processes**, **Reads / renders** or **Performs / executes**. Removing the last role unlinks the primary asset.
4. In **Depends on**, check the assets this one runs on; in **Provides to**, the assets that run on it. Search by name, or filter by **Category** and **Source**: linked assets come first.
5. Select the save button, which counts your changes, such as **Save 3 changes**.

   ::screenshot{id=opencrq-assets-dependencies alt="The Business dependencies graph of booking.grand-meridian.example, Very High and Internet Facing with 45 category TTPs, counting 2 supporting assets, 2 critical and 1 exposed: the web application performs the online booking platform and processes the guest records, depends on the booking database server, and faces the internet."}
:::

Each role passes some security criteria of the primary asset to the supporting asset; [Links and their roles](/opencrq/concepts/risk-scopes-and-assets/#links-and-their-roles) gives the table. A scenario reads the links two ways. At initial access, the run tells apart the assets linked to the scenario's primary assets from the rest of the estate. And the **Targets** tab of the scenario ranks the supporting assets by the share of its loss each one carries, a share halved at each dependency step.

## See the asset in OpenCTI and OpenAEV

An asset with a **Hostname** is the same machine in OpenCTI, as a System, and in OpenAEV, as an asset. The asset's header always shows two emblems beside **Edit**, OpenCTI then OpenAEV. An asset imported from one of them is found there from its import. Otherwise, OpenCRQ looks for its hostname with your own rights: the module answers as your account there, the one with your email address, so a machine you cannot see in it is not found. Hover an emblem, or move to it with the keyboard, to read what it stands for:

- In colour, the module holds the machine: the emblem opens it there, in a new tab. Its hint gives the module, the type and name of the machine there, the key facts the module returns about it, and what you find there: "See the reports, relationships and context of this object in OpenCTI." or "See the simulations and findings of this object in OpenAEV."
- Greyed, the module does not hold it: the hint reads **Not found in OpenCTI** or **Not found in OpenAEV**, then "It appears here once OpenCTI holds a machine with the same hostname." An asset with no hostname is found only in the module it came from. While OpenCRQ looks, the hint reads **Looking in OpenAEV…**; when the module does not answer, **OpenAEV could not be reached** and "Try again later."
- Greyed, the tenant is not connected to the module: the hint reads **OpenCTI is not connected** or **OpenAEV is not connected**, with the fields the module would show, as examples. With **Manage Security**, the emblem opens the connector's page in :menu[Integrations]; without it, the hint says "Ask an administrator to connect OpenCTI." See [Connect OpenCTI with a Live Stream](/opencrq/use/sources/opencti/) and [Import OpenAEV results](/opencrq/use/sources/openaev/).

## Read access relations and events

**Access relations** shows the relations that integrations observe between the asset and other assets of the inventory: people do not set them. The tab opens on a graph. Its list view, like the **Relations** card of the **Overview**, groups them by kind, each with the integration that reported it and the date it was last seen.

- Who uses which device: the user accounts that are the primary users of a device, as Microsoft Entra ID and Intune report them, under **Assigned devices** and **Assigned to**. The attack run reads them for phishing: a flaw on a device nobody uses opens nothing.
- Where an attack goes next: the attack paths of the OpenAEV simulations, from one asset of the inventory to the next, under **Leads to** and **Reached from**, and as **attack path** in the graph's legend. A step reads **OpenAEV, proven by a test** when an execution got through, and **OpenAEV, inferred** when OpenAEV drew it and no execution has proved it yet. A step that OpenAEV prevented is left out. The [OpenAEV connector](/opencrq/use/sources/openaev/) never deletes a step OpenAEV stops showing, so a purged simulation or an outage erases nothing: the date says how fresh it is. It imports attack paths only when OpenAEV serves its [snapshot export](/opencrq/use/sources/openaev/#read-the-snapshot-export-of-openaev). The price does not read attack paths.

Without such an integration, the tab says that no access relation is known, and the **Overview** has no **Relations** card.

Two other tabs say how the asset's findings moved and which controls cover it:

- **Events** is the history of the asset's **OpenCRQ Posture**, the score out of 100 its findings give it: 75 and above is **Strong**, then **Moderate**, **Weak** and **Critical** below 25. Each line says what moved it, such as a new worst finding or a finding resolved. An asset without findings has no posture, and the tab reads **No history yet.** See [Review the findings on your assets](/opencrq/use/attack-surface/findings/).
- **Protected by** lists the controls that cover the asset's category, with the posture each one has in your risk scopes. See [Set how strong your controls are and read how they held](/opencrq/use/defenses/controls/).

:::xtm-one[XTM One follows the attack paths from an asset]
XTM One reads through OpenCRQ how far an attack can go from an asset, the blast radius of a CTEM case: the assets the OpenAEV attack paths reach from it, step after step up to five steps, each step proven or inferred with the simulation that showed it, and the primary assets that depend on the assets reached. A weakness on an asset nothing depends on then still counts for the primary assets it opens the way to. A step OpenAEV has not reported for 30 days no longer counts. XTM One reads as the person it works for: an asset that person cannot read in OpenCRQ stops the path there and is never named.
:::

:::example[The booking web application of Grand Meridian France]
Grand Meridian Hotels declares the estate behind its booking platform by hand, from :menu[Assets] with **Create**:

- **Name** booking.grand-meridian.example, **Exposure** **Internet Facing**, **Category** **Web application**, **Subcategory** **Website**, **Criticality** **Very High**, and its URL https://booking.grand-meridian.example.
- **Name** Booking database server, **Exposure** **Internal**, **Category** **Host**, **Subcategory** **Server**, **Criticality** **Very High**.
- **Name** Head-office VPN gateway, **Exposure** **Internet Facing**, **Category** **Network device**, **Subcategory** **VPN gateway**, **Criticality** **High**.

Then, on the web application's **Business dependencies** tab, **Manage**: in **Primary assets**, **Performs / executes** for the Online booking platform and **Processes** for the Guest records; in **Depends on**, the Booking database server.

The list now reads 6 assets, two of them internet-facing: the web application and the VPN gateway are where an attack exploiting a public service gets in. The web application carries 45 **Category TTPs**, the server 537: a host is open to far more techniques than a website.
:::

## Check that it works

- The asset shows in :menu[Assets] with the exposure and criticality you set, and the **Set by user** mark on each value you set.
- Its **Business dependencies** graph shows the primary assets it supports.
- OpenCRQ prices the tenant's scenarios again when what they read changes, and once a day: the **CRQ** tab of a scenario dates its latest run.

## What's next

- [Review the findings on your assets](/opencrq/use/attack-surface/findings/), the flaws an integration found on them.
- [Set how strong your controls are and read how they held](/opencrq/use/defenses/controls/) on the estate.
- [Build and price a risk scenario](/opencrq/use/quantify/risk-scenarios/) on the primary assets they carry.
- [Import assets from Microsoft Defender](/opencrq/use/sources/microsoft-defender/) and [Import OpenAEV results](/opencrq/use/sources/openaev/).
