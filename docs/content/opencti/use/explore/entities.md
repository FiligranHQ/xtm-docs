---
title: Explore entities
description: See which threats target a sector, an organization, a person, a system or an event in the OpenCTI Entities menu, what was seen there, and how far to trust a source.
sidebar_position: 11
doc_type: how-to
products:
  - opencti
personas: [cti-analyst]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/exploring-entities/
---

The **Entities** menu answers "who is targeted, and what was seen there": the sectors, organizations, people, systems and real-world events that threats target, with the sightings recorded at each of them. It also holds the security tools that defend you, and the organizations and people that author your knowledge. This page shows how to read a victim's targeting, check what was seen at it, and judge a source. The tabs and panels every entity page shares are described in [Explore knowledge](/opencti/use/explore/#read-an-entity-page); this page covers what is specific to these types.

## Before you start

- Your role has the **Access knowledge** capability to read these pages.
- To link a victim to a threat or record a sighting, your role also needs **Create / Update knowledge**; to delete, **Delete knowledge**. See [Create knowledge manually](/opencti/use/ingest/manual/).

## What the Entities menu holds

| Entry | What it represents | What it answers in your analysis |
| --- | --- | --- |
| **Sectors** | An area of activity, such as energy or health, with its subsectors | Which threats go after an industry |
| **Events** | Something that happens in the real world and attracts attacks: a summit, an election, a sports competition | Which threats exploit an occasion |
| **Organizations** | A company, a government body, a CSIRT, a vendor | Which threats target a named victim, and how reliable a source is |
| **Security platforms** | One of your security tools: an EDR, an XDR, a SIEM, a SOAR, an NDR or an ISPM | Which techniques your defenses are expected to cover |
| **Systems** | A technology, such as a platform or a software product | Which threats target a technology, and which vulnerabilities it has |
| **Individuals** | A real person who matters to your analysis, such as a targeted executive | Which threats target a person, and how reliable a person is as a source |

The **Sectors** list is a tree: subsectors appear under their parent, and the search box is the only way to narrow it.

## See which threats target a victim

:::steps

1. Open :menu[Entities > Sectors], or the entry of the victim you follow, and search its name.
2. Open it, then the **Knowledge** tab. Its **Overview** summarizes what is known about the victim.
3. In the side menu, open **All threats** to list everything that targets it, or one entry, such as **Intrusion sets** or **Campaigns**, to list one kind. The number beside an entry counts its relationships.
4. Switch the list to **Relationships view** to see how sure and how recent each targeting is. **Start time** and **Stop time** say when it happened, as the source states it; **Platform creation date** says when it reached OpenCTI. See [Lists of linked objects](/opencti/use/explore/#lists-of-linked-objects).
5. Click a line to open the relationship and check its sources: see [The page of a relationship](/opencti/use/explore/#the-page-of-a-relationship).

:::

::screenshot{id=opencti-entities-sector-knowledge alt="All threats in the Knowledge tab of the Hospitality sector: the intrusion sets, threat actors, campaigns and malware that target it, with the side menu listing organizations, threats and campaigns with their counts."}

The **Overview** of **Knowledge** is the same for every victim:

- **Total reports**, **Total observables** and **Total relations**, each with its change over the last **30 days**: is the victim getting more attention?
- **Distribution of reports**: the reports that contain the victim, by author. One author alone means one source shapes everything you know about it.
- **Top 10 threats targeting this entity**: the threat actors, intrusion sets, campaigns and malware with a **targets** relationship to the victim, ranked by number of relationships.
- **Sightings distribution** counts the sightings of the object itself, so it stays empty on a victim, where things are seen. What was seen at the victim is in its **Sightings** tab.

The side menu lists the threats that target the victim, then what describes the victim itself:

| Entry | Lists | On |
| --- | --- | --- |
| **All threats**, **Threat actors**, **Intrusion sets**, **Campaigns**, **Incidents**, **Malware**, **Tools** | What targets the victim. For an organization, **Threat actors** also lists the actors it employs | Every type except security platforms |
| **Attack patterns** | Only the attack patterns recorded as targeting the victim itself. The techniques of the threats that target it are on each threat's own **Attack patterns** | Every type |
| **Organizations** | For a sector, the organizations in it; for an organization or a person, the organizations it is part of | Sectors, organizations, individuals |
| **Sectors**, **Individuals**, **Used tools** | The sectors the organization belongs to, its people, the tools it uses | Organizations |
| **Locations** | Where the victim is located | Organizations, individuals, events |
| **Vulnerabilities** | The vulnerabilities the organization or system has: its exposure | Organizations, systems |
| **Observables**, **Related entities** | What a source linked to the victim without a more precise relationship (**related to**) | Every type except security platforms |

:::only{opencti=">=7.260811"}
An organization also lists, under **Infrastructures**, the infrastructures that belong to it.
:::

:::only{opencti="<7.260811"}
On an organization, **Vulnerabilities** lists only the vulnerabilities a source linked to it with **related to**: in this version, an organization cannot have a vulnerability.
:::

:::example[See who targets the hospitality sector]
Your hotel group asks which threats are after its sector. Open :menu[Entities > Sectors], search `Hospitality`, open it, then **Knowledge**.

**Top 10 threats targeting this entity** lists the intrusion sets Velvet Harbor and Ember Tide, the threat actor groups Copper Lantern and Harbor Syndicate, the campaigns Operation Night Desk, Operation Checkout Rush and Operation Low Season, and the malware Velvet Loader and Harbor Stealer. **Distribution of reports** shows a single author, Example CERT: before you brief management, look for a second source. In the side menu, **Organizations (2)** lists Grand Meridian Hotels, a member of the sector, and Example CERT, the CERT that serves it; **Incidents (1)** lists the intrusion at Grand Meridian Hotels. Open Velvet Harbor from **Intrusion sets** to read its techniques.
:::

**AI Insights** on a sector summarizes its activity and the reports about it: see [Read the home dashboard and AI summaries](/opencti/use/explore/insights/).

### Stated and inferred targeting

A targeting is either stated by a source or inferred by the platform. With the rule **Targeting propagation via belonging** turned on, a threat that targets an organization is also shown as targeting the sector the organization is part of. In **Relationships view**, the wand icon of an inferred line names the rule in its tooltip, **Inferred knowledge based on the rule**, and its **Creators** column reads the rule manager. An administrator turns rules on in [Turn inference rules on or off](/opencti/configure/rules/). See [How OpenCTI infers new relationships](/opencti/use/enrich/inferences/) to read them, and [Explore locations](/opencti/use/explore/locations/#why-a-threat-shows-on-a-place-it-never-targeted) for a worked case.

## Check what was seen at a victim

The **Sightings** tab of a sector, an organization, an individual, a system or an event lists what a security tool or a partner saw there. Each line gives:

- **Status**: **True positive**, or **False positive** when the sighting turned out to be legitimate activity.
- **Count**: how many times it was seen, between **First obs.** and **Last obs.**
- **Confidence level**: how much the reporter trusts the sighting.

A name reads **Restricted** when the sighted object carries a marking you cannot see. A wand icon marks a sighting created by a rule. Open a line for the full sighting: see [Explore events](/opencti/use/explore/events/#sightings).

::screenshot{id=opencti-entities-organization-sightings alt="The Sightings tab of Grand Meridian Hotels: three true positive sightings, the indicator booking-velvetharbor.example seen 14 times, an event seen 4 times and the indicator 203.0.113.10 seen 3 times, with their first and last observation dates."}

On Grand Meridian Hotels, the domain `booking-velvetharbor.example` was seen 14 times on the night of January 8, 2026, and `203.0.113.10` 3 times in December 2025. Both indicate Velvet Harbor: the targeting is no longer only reported, it reached the hotel chain. The third line is the event **European Hospitality Summit 2026**, seen 4 times from January 5 to 8: the booking requests that named it reached the front desk. Its **Name** stays empty in the list; open the line to see which event it is.

## Read the details of each type

**Details**, on **Overview**, holds what makes each type useful to your analysis:

- **Organizations**: the **Organization type** says what the organization is to you: **constituent** (someone you serve or protect), **csirt**, **partner**, **vendor** or **other**. **Contact information** says how to reach it, for example to warn a targeted member. **Score** is a value from 0 to 100 that your team or the source assigns; OpenCTI does not compute it.
- **Sectors**: a sector lists its **Subsectors**, and a subsector its **Parent sectors**.
- **Events**: the **Event types**, such as **national-election** or **sport-competition**, then the **Start date** and **End date** of the event. Compare them with the dates of the campaigns that target it.
- **Individuals** and **Systems**: **Contact information** and **Reliability**, for when they are a source.
- **Security platforms**: the **Security platform type**.

:::only{opencti=">=7.261030"}
Events also have a **Score**, from 0 to 100, set by your team or the source.
:::

An organization here is a piece of knowledge, such as a victim or a source. The organizations that segregate users and data on the platform are managed in the settings: see [Separate data between organizations](/opencti/administer/organizations/).

::::only{opencti=">=7.261117"}
## Rank systems by criticality and exposure

A system can describe one of your own assets, such as the booking platform of a hotel chain: how much the business depends on it, whether it is reachable from the internet, and who owns it. When a threat targets several systems, these asset fields tell you which one to defend first. They use the asset categories and the criticality scale of OpenAEV assets.

| Field | Values | What it tells you |
| --- | --- | --- |
| **Criticality** | **Very high**, **High**, **Medium**, **Low** or **Unknown** | How much the business depends on the system |
| **Internet facing** | **Yes**, **No** or **Unknown** | Whether an attacker can reach it from the internet |
| **Environment** | **Production**, **Staging**, **Development**, **Test** or **Other** | Whether it serves real users and data |
| **System category**, **System subcategory** | A category, such as **Host** or **Web application**, then a kind of that category, such as **Server** or **Website** | What kind of asset it is |
| **Platform** | An operating system, such as **Linux** or **Windows**, or a kind of platform, such as **Container** | What it runs on |
| **Asset owner** | Free text | Who to contact about it |

The values of **Criticality** and **Internet facing** are fixed; the other lists are vocabularies an administrator can extend in [Taxonomies](/opencti/configure/taxonomies/). Every field is optional.

:::only{opencti=">=7.261127"}
A system can also be a code asset your teams build and ship: choose the category **Source code**, then **Code repository**, **CI/CD pipeline**, **Package registry**, **Software package** or **Other code asset**. A platform upgraded from an earlier version gets this category and its subcategories too. Renaming a category keeps its subcategories.
:::

To fill them in:

:::steps
1. Open :menu[Entities > Systems], then the system, and click **Update**.
2. Choose the **System category**, then the **System subcategory**: the list keeps the subcategories of the category you chose. Without a category, it lists them all, grouped by category.
3. Set **Internet facing**, **Criticality**, **Environment** and **Platform**, and type the **Asset owner**. Each field is saved when you change it.
:::

The same fields are in **Create a system**.

**Details**, on the system's **Overview**, shows **Criticality**, **Internet facing** and **Environment** first, each in the color of its value, then the **Type** (category and subcategory), the **Platform** and the **Asset owner**. A last line counts the asset fields not set and names them, so you know what to ask the system's owner.

::screenshot{id=opencti-entities-system-details alt="The Details card of the Booking database server: Criticality Very high, Internet facing No and Environment Production, then its type Host, Server, its platform Linux and its asset owner Grand Meridian Hotels IT operations."}

The systems list shows **Criticality** and **Environment** as columns. They cannot be sorted: filter the list on **Criticality** instead. Every asset field can be used as a filter. A filter lists the stored values, such as `VERY_HIGH` for **Very high** or `production` for **Production**.

::screenshot{id=opencti-entities-systems-list alt="The systems list of Grand Meridian Hotels, with the Criticality and Environment columns: the booking database server and the online booking platform Very high, the payment terminals High, the workstations Medium, all in Production."}

:::example[Decide which system of a hotel chain to protect first]
After the intrusion of January 9, 2026, Grand Meridian Hotels asks which of its systems to protect first. Open :menu[Entities > Systems] and add the filter **Criticality**, operator **Equals**, value `VERY_HIGH`. Two systems remain: the **Online booking platform**, **Internet facing** **Yes**, and the **Booking database server**, **Internet facing** **No**, a Linux **Host** of the **Server** kind owned by Grand Meridian Hotels IT operations.

Open the **Booking database server**, then **Knowledge**: Velvet Harbor, Harbor Syndicate and Operation Night Desk target it, and the incident of January 9, 2026 reached it. The booking platform is reachable from the internet, and the database behind it holds the guest records and is already targeted: brief their owners first.
:::
::::

## Judge an organization or a person as a source

Organizations, individuals and systems also author knowledge: a CERT publishes reports, a sensor creates sightings. **Display as**, at the top of their page, switches from **Knowledge entity** to **Author**:

- **Overview** replaces the latest relationships with **Latest containers authored by this entity**;
- **Knowledge** counts and draws the knowledge the source authored;
- **Analyses** lists the reports, groupings and cases it authored.

Read it with the **Reliability** of the source, in **Basic information**: see [Reliability and confidence](/opencti/concepts/reliability-confidence/).

:::example[Judge Example CERT as a source]
Before you rely on the hospitality advisories of Example CERT, open :menu[Entities > Organizations], then **Example CERT**, and set **Display as** to **Author**. **Latest containers authored by this entity** lists its reports on the sector, from **Fake booking requests reach hotel front desks** in October 2025 to **Hospitality threat brief, January 2026**, its monthly brief to the member hotels, and **Ember Tide encrypts hotel booking servers before the summer** in April 2026. **Basic information** gives its **Reliability**, **B - Usually reliable**. A steady flow of reports on one sector from a usually reliable CSIRT is a source worth following.
:::

## Match threats with your security platforms

A security platform describes one of your security tools. In its **Knowledge** tab, **Attack patterns** places on the ATT&CK matrix the techniques it **should cover**: those your team expects it to detect or block. **Display only used techniques** keeps those techniques; the other views are those of any [Attack patterns](/opencti/use/explore/#attack-patterns) entry.

On a threat, in **Knowledge** > **Attack patterns**, **Compare with my security posture** picks one or more security platforms and marks, on the threat's matrix, the techniques they should cover. The techniques of the threat that no platform should cover are your gaps. To measure what your tools really detect, run a simulation in OpenAEV: see [Measure your defenses against a threat](/opencti/use/analyze/security-coverage/).

:::only{opencti=">=7.261030"}
With the rule **Vulnerability propagation via software usage** turned on, a system that uses a software inherits the vulnerabilities of that software in its **Vulnerabilities**.
:::

## What's next

- [Explore locations](/opencti/use/explore/locations/) for the geographic side of victimology.
- [Explore threats](/opencti/use/explore/threats/) for the techniques and arsenal of the threats that target your victims.
- [Follow your priority threats with PIRs](/opencti/use/analyze/pir/) to follow the threats against your own sectors and locations.
- The [data model reference](/reference/opencti/data-model/#entities) for the attributes and relationships of each type of the Entities menu.
