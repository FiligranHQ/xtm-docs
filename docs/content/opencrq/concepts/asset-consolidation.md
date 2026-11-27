---
title: How OpenCRQ merges an asset several sources report
description: How OpenCRQ makes one supporting asset of what Microsoft Defender, OpenAEV and people report, which value wins, what you pin by hand, and when two reports are the same asset.
doc_type: explanation
products:
  - opencrq
personas: [risk-manager, admin]
owner: docs-opencrq
last_reviewed: 2026-11-26
no_screenshot: "The Sources card of an asset two integrations report needs Microsoft Defender, which the capture stack does not run: the diagram shows how the reports become one asset."
---

When Microsoft Defender and OpenAEV both report the booking database server, OpenCRQ shows one supporting asset, not two, and prices attacks on it once. Each integration keeps its own record of the asset, a value you set by hand outranks them all, and one rule per field picks what the asset shows. This page explains how OpenCRQ recognizes the same asset in two reports, which value wins, what **Set by user** and **Resync with connectors** do, and where the name of an asset nobody named comes from. To import the assets, see [Import assets from Microsoft Defender](/opencrq/use/sources/microsoft-defender/) and [Import OpenAEV results](/opencrq/use/sources/openaev/); to correct one, [Build the attack surface from your assets](/opencrq/use/attack-surface/assets/#correct-an-imported-asset).

::diagram{id=opencrq-asset-consolidation}

## One asset, one record per source

Every integration that reports an asset keeps a record of it: the id its tool gives the asset, and what it reports, such as the name, category, exposure, criticality, description, first and last sightings, and the properties of its kind (host name and addresses for a machine, account names for an identity). A sync updates only the records of its own integration. After each sync, OpenCRQ computes the asset's values again from all its records, with the rules of [Which value wins](#which-value-wins). What the screens, the findings and the price read is that result.

On the asset's **Overview**, the **Sources** card lists one row per integration reporting the asset. Its **About this source** hint gives the id the integration knows the asset by, its first and last sightings, and what it reports. A row **Set by hand** stands for the properties a person typed. An asset created by hand that no integration reports reads "This asset was created by hand."

A record stays when its integration stops reporting the asset: the asset keeps what that source last said, and nothing removes it because one source lost sight of it. Microsoft Defender alone removes some records of its own accounts, as [its page](/opencrq/use/sources/microsoft-defender/#what-the-connector-imports) says.

## Which value wins

A value someone pinned by hand wins over every rule (see [Values you set by hand](#values-you-set-by-hand)). Otherwise, each field follows its own rule, and the **What it means and how it is computed** button beside the field names the rule and what each source reports.

| Field | The value shown |
| --- | --- |
| **Name**, **Description**, **URL**, **Person** | The value of the source active most recently that reports one. A source's activity is the later of its last change and its last sighting of the asset, so a source that stopped seeing the asset loses to one still seeing it. On a tie, the higher-ranked source wins |
| **Category** and **Subcategory** | Both from one source, so the pair never disagrees: the highest-ranked source, Microsoft Defender, then OpenAEV, then any other; on a tie, the one active most recently. A subcategory that does not belong to the category shown is dropped, even one set by hand |
| **Exposure** | The most severe any source reports: **Internet Facing**, then **Internal**, then **Unknown** |
| **Criticality** | The highest band any source reports, from **Very High** down to **Low**, **Unknown** counting least, and at least the band the device's user gives it (below) |
| **First seen**, **Last seen** | The earliest first sighting and the latest last sighting of every source |

The **Owner** never comes from a source: only a person, or a push through OpenCRQ's API, sets it. The tenant setting that prices an **Unknown** exposure or criticality applies after these rules: see [What the price reads on an asset](/opencrq/use/attack-surface/assets/#what-the-price-reads-on-an-asset).

### The criticality a device takes from its user

A device takes a floor from the account that uses it. When Microsoft Entra ID or Intune, through the [Microsoft Defender connector](/opencrq/use/sources/microsoft-defender/), name an account as the primary user of a device, a **Very High** account makes the device at least **High**, and a **High** account at least **Medium**. An account of **Medium** or below adds nothing. With several users, the highest floor wins. The account's criticality counts as the account shows it, set by hand included, and a criticality pinned on the device outranks the floor. OpenCRQ applies it again whenever a sync or an edit changes an account's criticality or a device's primary user. The **What it means and how it is computed** button of the device's **Criticality** then names the account and the band it gives.

## Values you set by hand

When you change a field with **Edit**, OpenCRQ pins your value: it keeps it, synchronization after synchronization, whatever the sources report, and the field carries the **Set by user** mark. You can pin the **Name**, **Description**, **Exposure**, **Category**, **Subcategory**, **Criticality**, **URL** and **Person**. Except for the **Subcategory**, the field's **What it means and how it is computed** button says who set it and when. Emptying a text field, such as the **Description**, removes its pin: the sources' value takes over again at their next synchronization.

An asset you create by hand has every value you give it pinned the same way, except an **Exposure** or a **Criticality** left on **Unknown**: when an integration later recognizes the asset, it fills those two.

The properties you type in the cards of the **Overview** (**Network**, **Machine**, **Cloud**, **Identity** or **AI target connection**, depending on the category) are one more record, the **Set by hand** row of **Sources**, ranked above every integration, field by field. A property you type wins and carries **Set by user**; a property you leave empty shows what the integrations report. The integrations' lists, such as **IP addresses** and **MAC addresses**, are joined together, but a list you type replaces them rather than adding to them, so an address you removed does not come back. Typed properties never set the name, category, exposure or criticality. They can give the asset its [shown name](#the-name-of-an-asset-nobody-named), and they are what lets an integration [recognize an asset you created](#an-asset-you-created-by-hand).

**Resync with connectors**, beside **Edit**, hands the asset back to its integrations. It shows only when a value is pinned or a property typed, and an integration still reports the asset. Its confirmation names the fields edited by hand; **Resync** removes every pin and every typed property at once, and the asset takes the values its integrations report. It cannot be undone. Tags, groups, the owner and the links to primary assets stay as they are.

## How OpenCRQ recognizes one asset in two reports

OpenCRQ decides which asset a record belongs to once, when a source reports an asset for the first time. The record then stays on that asset: a later change of name or address does not move it, and two assets never merge afterwards. There is no similarity score and no list of possible duplicates: what no rule joins stays apart.

The rules apply within one family of categories, read from the category each source reports, never across families: a laptop is never compared to a SaaS application.

| Family | Categories | Joined on, the strongest first |
| --- | --- | --- |
| Machine | **Host**, **Container / Workload**, **Cloud resource**, **Mobile device**, **Network device**, **IoT / OT device** | 1. The Microsoft Defender device ID, which an OpenAEV agent deployed by OpenAEV's Microsoft Defender executor also carries.<br>2. The hardware UUID, then the Microsoft Entra device ID, then the cloud VM ID, then the cloud resource ID.<br>3. A shared MAC address.<br>4. The same fully qualified host name and the same platform, with a shared IP or MAC address |
| Identity | **Identity** | The Microsoft Entra object ID, or a security identifier (SID), cloud and on-premises counting as one key |
| AI target | **AI target** | The same endpoint URL and the same model, both present |
| Application | **Web application**, **SaaS application**, **Source code** | Never joined |
| Security platform | **Security platform** | Never joined: a detection tool never becomes a target by sharing a host name |
| Unclassified | **Generic asset** | Never joined |

For the three families that join, the last resort is the external reference, the identifier your own system gave the asset in OpenAEV. Every rule but the first refuses a join that a stronger identifier contradicts:

- A hardware, directory or cloud identifier never joins two machines whose Microsoft Defender device IDs differ. A MAC address or a host name never joins two machines whose device IDs, or whose identifiers of the same kind, differ.
- A shared MAC address also needs the host names to agree when both sides have one, compared on their first label: `bkdb01` and `bkdb01.grand-meridian.example` agree. A docking station or a recycled network card does not merge two machines.
- A host name alone, an IP address alone, or a short host name without its domain never joins two machines.
- Two accounts whose object IDs differ, or whose SIDs of the same kind differ, are never joined. The user principal name (UPN) is never a key: a mailbox handed from a leaver to a joiner keeps its UPN.
- The external reference never joins two assets that an identifier of their family tells apart.

In this version, the identity rules join the accounts Microsoft Defender sees twice, such as an account first met in a sign-in event, known by its SID, and its directory record. OpenAEV reports no object ID or SID for its identities, so an identity it reports is never merged with one Defender reports. AI targets come from OpenAEV alone.

### An asset you created by hand

An integration recognizes an asset you created only through the properties you typed. You cannot type a device ID, a hardware or cloud identifier, an object ID or a SID: those come from the tools that assign them.

- A machine joins through its **MAC addresses**, or through its **Hostname** with a domain, its **Platform** and a shared address.
- An AI target joins through its **Endpoint URL** and its **Model**.
- An identity, and an asset with no typed property, is never recognized: the integration's report becomes an asset of its own.

A **Hostname** you typed that differs from the one the integration reports blocks the MAC address rule, and the integration's report becomes another asset. When the join happens, the integration's record lands on your asset, your pinned values stay, and the integration fills the rest.

## The name of an asset nobody named

When no source reports a name and nobody typed one, OpenCRQ builds a name from the asset's properties, typed ones first. It shows in grey italics, and its hint says the name was deduced from the asset's properties. The lists sort and search on it, it is never pinned, and it follows the properties when they change.

- A machine shows its host name. Without one, its vendor and model, then one address: the seen IP, else the lowest IPv4 address, else an IPv6 address, else a MAC address, such as `Cisco C9300-48P · 10.20.0.1`. The vendor is left out when the model starts with it.
- An identity shows its UPN, else `domain\account` or the account name alone, else its object ID, else its SID.
- An AI target shows its model, else its endpoint URL.
- A **Generic asset** that an integration could not classify is named from the first properties it carries, those of a machine, an identity, then an AI target.

An asset with none of these reads **Unnamed asset**, in grey.

## Tags and groups

A tag is one tag of the tenant, whichever source reports it. When an integration such as OpenAEV reports a tag whose name already exists, it joins that tag: a `booking-platform` tag a person made and the `booking-platform` tag of OpenAEV are one. Each source keeps its own tags on an asset: a person's edit replaces only the person's tags, a sync only its integration's, and the asset shows each tag once.

When an integration renames a tag in its own tool, OpenCRQ follows:

- The integration is the tag's only source: the tag is renamed.
- Other sources still report the old name: a new tag is made for the integration, and the old one stays for the others.
- The new name is already another tag: the integration moves to it, and the old tag is deleted if no other source reports it.

A tag a person renamed or merged in :menu[Settings > Taxonomies > Tags] stays as the person left it: an integration's rename never undoes it. A tag takes its colour from the source that began reporting it last, among those that give one. See [Manage the tags of your assets](/opencrq/configure/risk-taxonomies/#manage-the-tags-of-your-assets).

Groups follow the same principle: each source keeps its own memberships, and a sync replaces only those of its integration. See [Find the assets of a group](/opencrq/use/attack-surface/assets/#find-the-assets-of-a-group).

:::example[The booking database server of Grand Meridian Hotels]
In January 2026, Grand Meridian's risk manager creates the Booking database server by hand: **Category** **Host**, **Subcategory** **Server**, **Criticality** **Very High**, and **Exposure** left on **Unknown**. She also types the **Hostname** `bkdb01.grand-meridian.example` and the **MAC addresses** `00:15:5D:4A:21:07`. Every value but the exposure is pinned.

The administrator then connects Microsoft Defender. Its first run reports the device `bkdb01.grand-meridian.example`, with the same MAC address, **Internal**, and **High** from the asset value set in Defender. The MAC address is shared and the host names agree, so Defender's record joins her asset. Next, OpenAEV reports its endpoint `BKDB01`, **Internal** and **Medium**, with the description "PostgreSQL primary of the booking platform" and the tag `booking-platform`. Its agent was deployed by OpenAEV's Microsoft Defender executor and carries Defender's device ID: it joins the same asset.

The asset reads:

- **Name** Booking database server, **Category** **Host**, **Subcategory** **Server** and **Criticality** **Very High**, each with **Set by user**.
- **Exposure** **Internal**, the most severe the two sources report, since nobody pinned it.
- **Description** from OpenAEV, the only source that reports one.
- The `booking-platform` tag once, shared by the person's tag and OpenAEV's.
- In **Sources**, three rows: Microsoft Defender, OpenAEV and **Set by hand**.

**Resync with connectors** would remove the pins and the typed properties: the name would become one of the names the integrations report, and the **Criticality** **High**, the highest of Defender's **High** and OpenAEV's **Medium**.
:::

## What's next

- [How OpenCRQ's objects connect, from a risk scope to its price](/opencrq/concepts/data-model/): where a supporting asset sits among the other objects.
- [Build the attack surface from your assets](/opencrq/use/attack-surface/assets/), to correct an imported asset or hand it back.
- [Import assets from Microsoft Defender](/opencrq/use/sources/microsoft-defender/) and [Import OpenAEV results](/opencrq/use/sources/openaev/).
- [Risk scopes, primary assets and supporting assets](/opencrq/concepts/risk-scopes-and-assets/), what the price reads on these assets.
- [Manage the tags of your assets](/opencrq/configure/risk-taxonomies/#manage-the-tags-of-your-assets).
