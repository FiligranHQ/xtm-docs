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
- To import assets rather than declare them, an administrator with **Manage Security** connects [OpenAEV](/opencrq/use/sources/openaev/) or [Microsoft Defender](/opencrq/use/sources/microsoft-defender/) in :menu[Administration > Integrations].

## What the price reads on an asset

Supporting assets belong to the tenant, not to a risk scope. A scope measured over the **Whole estate** walks every asset of the tenant; a scope measured over **Mapped assets only** walks the assets mapped into it. A scope that [declares its estate](/opencrq/concepts/risk-scopes-and-assets/#the-declared-estate) also counts the assets nobody listed, up to its declaration. The attack run reads three facts on each asset:

| Field | Values | What it changes |
| --- | --- | --- |
| **Category** and **Subcategory** | **Host**, **Web application**, **Network device**, **Identity**, **IoT / OT device**, **Cloud resource**, **SaaS application**, **Mobile device**, **Container / Workload**, **AI target**, **Security platform**, **Generic asset**; then a subcategory such as **Server**, **Workstation**, **Website**, **VPN gateway**, **User account**, **Point of sale**, **EDR** or **SIEM** | The attack techniques that apply to the asset. Its **TTP** tab lists them as **Category TTPs**. A **Security platform**, a detection or prevention tool such as an EDR or a SIEM, stays in the inventory but has no technique and carries no loss: the price never counts it as a target |
| **Exposure** | **Internet Facing**, **Internal**, **Unknown** (the default) | Where an attack gets in. Exploiting a public service needs an internet-facing asset; phishing goes through workstations and user accounts. At the door, an internet-facing asset weighs 1, an internal one 0.4 |
| **Criticality** | **Very High**, **High**, **Medium**, **Low**, **Unknown** (the default) | How much an asset weighs once the attacker is inside and moves toward the target: 1 for **Very High**, 0.9, 0.7 and 0.4 below. It measures the blast radius of the asset, not its business value, which the primary assets carry. It also raises the score of the findings on the asset |

An asset left on **Unknown** is priced as the tenant setting says: **Internal** exposure and **Low** criticality by default, under :menu[Settings > Engine > CRQ Indicators] (**Unknown exposure** and **Unknown criticality**). An unknown left on a server that faces the internet prices it as internal: set the fields you know.

## Declare a supporting asset

:::steps
1. Go to :menu[Attack surface > Assets]. The card above the list counts the assets of each category, the largest first, and the list shows each asset with its **Exposure**, **Criticality**, **Category TTPs** and **Findings**. On a short screen, the card shows its first three categories, and **Show all categories** opens the rest. Inside a risk scope, the card also sets the scope's [declared estate](/opencrq/concepts/risk-scopes-and-assets/#the-declared-estate) beside what is measured, and **Estimate my estate** (**Review my estimate** once the scope has one) leads to the risk scope to answer it.

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

To hand the asset back to its integrations, open **More actions**, select **Resync with connectors**, then **Resync**: every value you set by hand is cleared and the integrations' values apply again.

## Find the assets of a group

An asset group is a named set of assets, such as the servers of the head office. OpenCRQ does not create groups from its own screens: they come from your other tools.

- **The OpenAEV connector** mirrors the asset groups of OpenAEV at each run, with their static members and the members a dynamic group's filter finds at that moment. A member OpenCRQ did not import is left out. A group deleted in OpenAEV is deleted in OpenCRQ once every OpenAEV tenant has answered the run, so an OpenAEV outage never empties your groups. See [Import OpenAEV results](/opencrq/use/sources/openaev/).
- **XTM One, or a script**, puts an asset in groups by name when it writes the asset through OpenCRQ's API, and creates a group that does not exist yet.

Each source keeps its own memberships: a sync replaces only the members its connector added, never those XTM One or a person added.

:::steps
1. Go to :menu[Attack surface > Assets]. The **Groups** column shows the groups of each asset on one line, with **+N** for the rest.
2. Open the **Group** filter, shown beside **Source** and **Category** once the tenant has groups, and choose a group: each one shows its number of assets. The filter then reads the group's name, such as **Group: Hotel front desks**. **All groups** shows the whole list again.

   ::screenshot{id=opencrq-assets-groups alt="The Assets list of Grand Meridian Hotels filtered on the Hotel front desks group: the front-desk workstations, the night.audit account and the Spanish payment terminals, each with Hotel front desks in its Groups column."}
:::

The group filter combines with the search and the other filters: **Source** keeps the assets one integration observed, or, with **Set by user**, those declared by hand; **Category** keeps one category, or, under **Match**, every category **Except** that one. The count of active filters shows after them, with **Clear all** to lift every filter and the search at once. When nothing matches, the list reads **No results**, with **Clear filters**. The filters stay in the page's address, so you can share the filtered list.

:::example[Review the front desks before the awareness campaign]
Grand Meridian's risk manager prepares the front desks' training budget. In :menu[Attack surface > Assets], she opens the **Group** filter and chooses **Hotel front desks (3)**. The list shows three assets: the Front-desk workstations, the night.audit account and the Front-desk payment terminals of Spain. Two of them have a **Criticality** of **High**: those are the assets she names in the training request.
:::

## Link the asset to what it supports

A primary asset is lost through the supporting assets that carry it. The links say which ones, and with which role.

:::steps
1. Open the asset and its **Business dependencies** tab. The graph shows the primary assets the asset supports, the assets it depends on and those it serves, with counters of the critical, exposed and unowned assets in the chain.
2. Select **Manage**. The **Manage dependencies of supporting asset** panel has three tabs: **Primary assets**, **Depends on** and **Provides to**.
3. In **Primary assets**, pick one or more roles for each primary asset the asset serves, such as **Stores**, **Processes**, **Reads / renders** or **Performs / executes**. Removing the last role unlinks the primary asset.
4. In **Depends on**, check the assets this one runs on; in **Provides to**, the assets that run on it. Search by name, or filter by **Category** and **Source**: linked assets come first.
5. Select **Save**.

   ::screenshot{id=opencrq-assets-dependencies alt="The Business dependencies graph of booking.grand-meridian.example, Very High and Internet Facing with 45 category TTPs, counting 2 supporting assets, 2 critical and 1 exposed: the web application performs the online booking platform and processes the guest records, depends on the booking database server, and faces the internet."}
:::

Each role passes some security criteria of the primary asset to the supporting asset; [Links and their roles](/opencrq/concepts/risk-scopes-and-assets/#links-and-their-roles) gives the table. A scenario reads the links two ways. At initial access, the run tells apart the assets linked to the scenario's primary assets from the rest of the estate. And the **Targets** tab of the scenario ranks the supporting assets by the share of its loss each one carries, a share halved at each dependency step.

## See the asset in OpenCTI and OpenAEV

An asset with a **Hostname** is the same machine in OpenCTI, as a System, and in OpenAEV, as an asset. When the tenant's OpenCTI or OpenAEV connector is set, the asset's page shows an **In OpenCTI** or **In OpenAEV** card above its tabs: the link that opens the machine there. OpenCTI finds it by name; the **In OpenAEV** card adds the key facts OpenAEV returns about it. The card reads **Not found in OpenCTI.** or **Not found in OpenAEV.** when that product does not hold the hostname.

When a connector is not set, the page offers **Connect OpenCTI** or **Connect OpenAEV** beside **Edit**, which opens the connector's settings. Without **Manage Security**, the button is disabled, and its hint says to ask an administrator. An administrator sets the connectors: see [Connect OpenCTI with a Live Stream](/opencrq/use/sources/opencti/) and [Import OpenAEV results](/opencrq/use/sources/openaev/).

## Read access relations and events

- **Access relations** shows who uses which device: the user accounts that are the primary users of a device, as Microsoft Entra ID and Intune report them. Integrations set these relations, not people. The attack run reads them for phishing: a flaw on a device nobody uses opens nothing. Without such an integration, the tab says that no access relation is known.
- **Events** is the history of the asset's **OpenCRQ Posture**, the score out of 100 its findings give it: 75 and above is **Strong**, then **Moderate**, **Weak** and **Critical** below 25. Each line says what moved it, such as a new worst finding or a finding resolved. An asset without findings has no posture, and the tab reads **No history yet.** See [Review the findings on your assets](/opencrq/use/attack-surface/findings/).
- **Protected by** lists the controls that cover the asset's category, with the posture each one has in your risk scopes. See [Set how strong your controls are and read how they held](/opencrq/use/defenses/controls/).

:::example[The booking web application of Grand Meridian France]
Grand Meridian Hotels declares the estate behind its booking platform by hand, from :menu[Attack surface > Assets] with **Create**:

- **Name** booking.grand-meridian.example, **Exposure** **Internet Facing**, **Category** **Web application**, **Subcategory** **Website**, **Criticality** **Very High**, and its URL https://booking.grand-meridian.example.
- **Name** Booking database server, **Exposure** **Internal**, **Category** **Host**, **Subcategory** **Server**, **Criticality** **Very High**.
- **Name** Head-office VPN gateway, **Exposure** **Internet Facing**, **Category** **Network device**, **Subcategory** **VPN gateway**, **Criticality** **High**.

Then, on the web application's **Business dependencies** tab, **Manage**: in **Primary assets**, **Performs / executes** for the Online booking platform and **Processes** for the Guest records; in **Depends on**, the Booking database server.

The list now reads 6 assets, two of them internet-facing: the web application and the VPN gateway are where an attack exploiting a public service gets in. The web application carries 45 **Category TTPs**, the server 537: a host is open to far more techniques than a website.
:::

## Check that it works

- The asset shows in :menu[Attack surface > Assets] with the exposure and criticality you set, and the **Set by user** mark on each value you set.
- Its **Business dependencies** graph shows the primary assets it supports.
- OpenCRQ prices the tenant's scenarios again when what they read changes, and once a day: the **CRQ** tab of a scenario dates its latest run.

## What's next

- [Review the findings on your assets](/opencrq/use/attack-surface/findings/), the flaws an integration found on them.
- [Set how strong your controls are and read how they held](/opencrq/use/defenses/controls/) on the estate.
- [Build and price a risk scenario](/opencrq/use/quantify/risk-scenarios/) on the primary assets they carry.
- [Import assets from Microsoft Defender](/opencrq/use/sources/microsoft-defender/) and [Import OpenAEV results](/opencrq/use/sources/openaev/).
