---
title: Data model
description: How OpenCTI represents what you know about a threat, as entities, observables and relationships that carry their own source and confidence, and where you find each type.
sidebar_position: 2
doc_type: explanation
products:
  - opencti
personas: [cti-analyst, evaluator, developer]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/data-model/
---

OpenCTI stores what you know about threats as a graph. Velvet Harbor is an Intrusion Set, 203.0.113.10 is an IPv4 address, and "Velvet Harbor uses Velvet Loader" is a relationship between two objects. Each statement is an object of its own, with its author, its confidence, its markings and its dates. This page explains which object holds each thing you know, the rules of the graph and what they change for your analysis, and where each type lives in the interface.

## What you know, as OpenCTI stores it

| What you know | How OpenCTI stores it | In the demo data |
| --- | --- | --- |
| Who is behind an attack, and what they run | Entities: Threat Actor Group, Intrusion Set, Campaign | Harbor Syndicate, Velvet Harbor, Operation Night Desk |
| What they use | Entities: Malware, Tool, Attack Pattern, Vulnerability, Channel | Velvet Loader, Spearphishing Attachment, CVE-2024-3400 |
| Who and where they target | Entities: Sector, Organization, Individual, Country, Region | Hospitality, Grand Meridian Hotels, France |
| A technical fact seen on a network or a system | Observables: IPv4 address, Domain name, URL, File and about 30 more | 203.0.113.10, booking-velvetharbor.example |
| A detection rule to deploy | Indicator: a pattern, a score and a validity period | The indicator 203.0.113.10, based on the IP address |
| A link between two things | Relationship, typed and directed, with its own source and confidence | Velvet Harbor `uses` Harbor Stealer, at confidence 30 |
| Something seen somewhere | Sighting, with a count and first and last seen dates | The indicator 203.0.113.10 seen in a hotel network |
| What one source said, kept together | Container: Report, Grouping, Note, Opinion, Observed Data, case | The report "Velvet Harbor targets hotel booking systems" |
| How a piece of knowledge is classified | Meta objects: markings, labels, external references, kill chain phases | TLP:AMBER, the label `hotel-sector` |

## How the graph is built

::diagram{id=opencti-data-model-graph}

Four rules explain most of the model, and most of what surprises analysts:

1. **Every link is an object.** A relationship has its own author, confidence, markings and dates, and its own page. "Velvet Harbor uses Harbor Stealer" is rated 30 because a single source says so, while "Velvet Harbor uses Velvet Loader" is rated 75. A report can contain a relationship, and a note or an opinion can be about one.
2. **Not every pair of types can be linked.** OpenCTI only accepts the relationship types that make sense between two types: an Intrusion Set `uses` a Malware, an Indicator `indicates` an Intrusion Set. The creation forms only offer the types allowed between the two objects you picked. The [data model reference](/reference/opencti/data-model/) gives the relationships each type can start.
3. **One fact, one object.** OpenCTI recognizes an object it already has, so two reports that name Velvet Harbor point to the same Intrusion Set, and two sources that say "Velvet Harbor uses Velvet Loader" share one relationship. Each report still lists it, so you see every source of a fact. [Deduplication](/opencti/concepts/deduplication/) gives the properties that identify each type.
4. **Author, markings and labels are links too.** In STIX, an object names its author, markings and labels inside its own attributes. OpenCTI stores each of these as a relationship, so you can pivot on them: from Example CERT to everything it authored. See [Nested objects](/opencti/concepts/nested-objects/).

The page of a relationship shows what makes it an object: its description, its confidence, its author, its markings and the containers that state it.

::screenshot{id=opencti-explore-relationship-page alt="The page of the relationship Velvet Harbor uses Harbor Stealer: its description says a single source links them, its confidence level is low, and the report it comes from is listed under Latest containers about the object."}

### Observables and indicators are separate objects

An observable records a fact: the IP address 203.0.113.10 exists and was seen. An indicator records a detection: the pattern `[ipv4-addr:value = '203.0.113.10']`, with a score, a validity period and the threat it `indicates`. The indicator is `based-on` the observable.

The split matters for your work. You can keep an observable as context, without asking anyone to block it. Only an indicator ages: its score drops over time and the indicator is revoked when the score reaches the revoke score of its decay rule, while the observable stays. See [How indicators age and expire](/opencti/use/analyze/indicators-lifecycle/).

### Threat Actor is split in two

OpenCTI divides the STIX Threat Actor into **Threat Actor Group**, such as Harbor Syndicate, and **Threat Actor Individual**, a person with a date of birth, a job title and links such as `citizen-of`, `employed-by` or `reports-to`. Both are exported as STIX `threat-actor`, so other tools see a standard object.

## Model what you know

A few analytical tasks, and the objects they take:

| Task | How to model it |
| --- | --- |
| Attribute a campaign | Operation Night Desk `attributed-to` Velvet Harbor, which is `attributed-to` Harbor Syndicate. Each link carries its own confidence, so a weak attribution stays visibly weak |
| Follow how an actor's techniques change | One `uses` relationship per Attack Pattern, with a **Start time** and a **Stop time**: Velvet Harbor used Spearphishing Attachment from March to September 2025, then Valid Accounts from October 2025 |
| Map a disinformation campaign | A Campaign `uses` the Narrative "Hotel chains hide guest data breaches". A Channel `uses` the same Narrative, `publishes` the fake articles as Media content observables and `amplifies` other Channels. A narrative can be `subnarrative-of` a broader one |
| Record a victim's incident | An Incident, such as the unauthorized access to Grand Meridian Hotels' booking database, `attributed-to` Velvet Harbor and `targets` Grand Meridian Hotels |

What the dates of a relationship mean, and which ones the platform sets, is in [Meaning of dates](/opencti/concepts/dates/).

## Where each type lives in the interface

The left menu groups the types by family, in nine menus:

| Menu | Types |
| --- | --- |
| **Analyses** | Reports, groupings, malware analyses, security coverages, notes, external references |
| **Cases** | Incident responses, requests for information, requests for takedown, tasks, feedbacks |
| **Events** | Incidents, sightings, observed data |
| **Observations** | Observables, artifacts, indicators, infrastructures |
| **Threats** | Threat actors (group and individual), intrusion sets, campaigns |
| **Arsenal** | Malwares, channels, tools, vulnerabilities |
| **Techniques** | Attack patterns, narratives, courses of action, data components, data sources |
| **Entities** | Sectors, events, organizations, security platforms, systems, individuals |
| **Locations** | Regions, countries, administrative areas, cities, positions |

Relationships have no menu of their own. You find them on the **Knowledge** tab of the objects they link, and all of them in :menu[Data > Relationships]. How to read any entity page, from who says so to its links and sources, is in [Explore knowledge](/opencti/use/explore/). The [data model reference](/reference/opencti/data-model/) lists every type of your version in the order of the menus.

## What OpenCTI adds to STIX 2.1

The model is Structured Threat Information Expression (STIX) 2.1, so a bundle from a partner imports without conversion and an export is read by any STIX tool. OpenCTI adds what analysts need and the standard lacks:

- **Entities** for cybercrime, disinformation and case management: Channel, Narrative, Event, Language, Administrative Area, Security Platform, Threat Actor Group and Individual, the cases, Task and Feedback.
- **Observables** for fraud and information systems: Cryptocurrency wallet, Bank account, Payment card, Phone number, Credential, Tracking number, Hostname, User agent, Media content, Persona, SSH key, AI prompt and others.
- **Relationship types**, 15 of the 59 OpenCTI knows: `publishes`, `amplifies` and `subnarrative-of` for channels and narratives, `employed-by`, `citizen-of`, `national-of` and `resides-in` for individuals, `cooperates-with` and `part-of` between groups, and others.

These types and attributes are exported as STIX custom objects and custom properties, such as `x_opencti_score`, so another tool still reads the standard part. How to work with the model through the API is in [Integrate and extend](/develop/).

## Attributes of a type

Each type adds its own attributes to the common ones: author, markings, labels, confidence and dates.

:::only{opencti=">=7.261117"}
A System, such as a hotel chain's booking platform, also describes the asset it is: **System category** and **System subcategory**, **Internet facing**, **Criticality**, **Environment**, **Platform** and **Asset owner**. All but **Asset owner**, a free text, take their values from vocabularies, and the values of **Criticality** and **Internet facing** are fixed. These fields follow the asset taxonomy of OpenAEV, so a System maps to an OpenAEV or XTM One asset. You can filter on each of them.
:::

- The [data model reference](/reference/opencti/data-model/) gives, for each type of your version, the attributes it adds, which ones are required and the inherited attributes of each family.
- :menu[Settings > Customization > Entity types] lets administrators make attributes mandatory or give them default values. See [Entity types](/opencti/configure/entities/).
- The [GraphQL schema reference](/reference/opencti/graphql-schema/) documents every type and attribute of the API, for scripts and integrations.
- To read every type of the API, such as `IntrusionSet`, with its fields, open the [GraphQL schema reference](/reference/opencti/graphql-schema/). The **Playground** button of the **API access** card of your profile opens the GraphQL playground to run queries; it needs the **Allow token usage** capability and shows while the playground is on (`app:graphql:playground:enabled`). Its schema documentation stays empty unless an administrator allows introspection (`app:graphql:playground:force_disabled_introspection`, `true` by default, see [Harden](/deploy/operate/harden/#graphql-playground)).

## What's next

- [Containers](/opencti/concepts/containers/): reports, groupings and cases, and how they hold knowledge.
- [Deduplication](/opencti/concepts/deduplication/): how OpenCTI recognizes an object it already has.
- [Reliability and confidence](/opencti/concepts/reliability-confidence/): what the confidence of an object and of a source change.
- [Explore knowledge](/opencti/use/explore/): browse each family of types.
- [Create knowledge manually](/opencti/use/ingest/manual/): create entities and relationships in the interface.
- [Data model reference](/reference/opencti/data-model/): every type, its attributes and its relationships.
