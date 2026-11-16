---
title: Deduplication
description: How OpenCTI recognizes an object it already has, why two objects become one or stay apart, and how confidence decides what an incoming duplicate can change.
sidebar_position: 6
doc_type: explanation
products:
  - opencti
personas: [cti-analyst, admin, developer]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/deduplication/
---

OpenCTI never stores the same object twice. When you, a connector or a feed create an object, the platform computes its identifiers from a few properties, looks for an object that already has one of them, and updates that object instead of creating a new one. This is called an upsert. This page explains which properties identify each type, why two objects became one or stayed apart, and what an incoming duplicate may change. With it, you can predict whether two sources will meet on the same Intrusion Set, and why a description did or did not change after an import.

## What deduplication is good for

| Goal | Example |
| --- | --- |
| Consolidate several sources on one object | Three feeds report Velvet Loader: one Malware, with the aliases, labels and references of all three |
| Recognize an object under another name | A report names "Harbor Mantis", an alias of Velvet Harbor: the report links to the existing Intrusion Set |
| Re-import without creating noise | A connector sends the same indicators every hour: they are updated, not duplicated |
| Protect trusted knowledge | A low-confidence feed cannot overwrite the description your analysts wrote |

## How the platform recognizes an object

::diagram{id=opencti-deduplication-flow}

Every object has several identifiers, and a match on any of them is a match:

- **The standard ID**, shown as **Standard STIX ID**. The platform computes it from the ID contributing properties of the type, listed below. The same properties always give the same ID, on every OpenCTI platform.
- **Other STIX IDs**, shown as **Other STIX IDs**: the IDs the object had in the sources it came from, and its former standard IDs.
- **One ID per alias**, for types with aliases: the ID the object would have if the alias were its name.
- **One ID per hash**, for files, artifacts and X.509 certificates.

Names are compared without case and without spaces around them: "Velvet Harbor" and "velvet harbor " give the same ID. Aliases, MITRE IDs and label values are compared the same way.

Then the platform acts on what it found:

| Found | What happens |
| --- | --- |
| Nothing | The object is created |
| One object | That object is updated with the incoming data, within the limits of confidence |
| Several objects | Observables that share hashes are merged into one, then updated. For other types, the platform updates the object that has the computed standard ID, or the one that already carries the incoming STIX ID. When the incoming STIX ID belongs to a second object, it merges the two first, keeping the one with the higher confidence |
| An object you cannot see | The creation fails with "Restricted entity already exists". When request access is set up, you can ask for access to it. See [Let users request access](/opencti/administer/request-access/) |

An automatic merge needs the same confidence as a manual one: the user or connector must have a max confidence level at least equal to the confidence of every object merged. Otherwise, the platform does not merge and updates one of the objects.

## Why two objects did not become one

Deduplication only recognizes an identifier, never a resemblance. Two objects stay apart when no name, alias, hash or STIX ID of one matches the other:

- **A spelling that differs beyond case and spaces.** "Velvet Harbour" is not "Velvet Harbor".
- **An alias that is not declared.** "Velvet Harbor Group" is a different name until someone adds it as an alias of Velvet Harbor.
- **A different type.** A Threat Actor Group and an Intrusion Set named Velvet Harbor are two objects, as are a group and an individual Threat Actor of the same name.
- **A property that identifies the type.** Two reports with the same name but different publication dates are two reports.
- **Aliases restricted by your administrator.** Only the aliases that match the rules of the type count as identifiers (see [Names and aliases](#names-and-aliases)).

::screenshot{id=opencti-deduplication-three-intrusion-sets alt="Searching intrusion sets for velvet shows three cards, Velvet Harbor, Velvet Harbor Group and Velvet Harbour, each with its own aliases under Known as."}

When you know two objects are the same, [merge them](/opencti/use/enrich/merge/): you need the **Merge knowledge** capability. The merged names become aliases of the kept object, so the next import of any of those spellings lands on it.

:::example[Three sources, three Velvet Harbors]
Example CERT reports Velvet Harbor, with the aliases FIN99 and Harbor Mantis. Saltmarsh CERT reports "Velvet Harbour", alias "Harbour Mantis"; Bluefin Labs reports "Velvet Harbor Group", alias "VHG". None of these names or aliases is an identifier of Example CERT's object, so your platform shows three Intrusion Sets.

You merge the two others into Velvet Harbor: their names become aliases of Velvet Harbor, and their relationships and reports move to it. From then on, a Saltmarsh CERT report that names "Velvet Harbour" links to Velvet Harbor directly.
:::

## Properties that identify entities

| Type | ID contributing properties |
| --- | --- |
| Attack Pattern, Course of Action | MITRE ID when present, otherwise name |
| Campaign, Channel, Event, Infrastructure, Intrusion Set, Language, Malware, Narrative, Tool, Vulnerability | Name |
| Threat Actor (group or individual) | Name and type: a group and an individual can share a name |
| Incident | Name and original creation date |
| Data Component, Data Source | Name |
| Indicator | Pattern |
| Organization, Individual, Sector, System, Security Platform | Name and identity class |
| Region, Country, City | Name and location type |
| Administrative Area | Name |
| Position | Latitude and longitude when present, otherwise name |
| Report | Name and publication date |
| Grouping | Name, context and original creation date |
| Note | Content and original creation date |
| Opinion | Opinion and original creation date |
| Observed Data | The objects it contains |
| Incident response, Request for information, Request for takedown, Feedback, Task | Name and original creation date |
| Malware Analysis | Product, result name and submission date |
| Label | Value |
| Marking definition | Definition type and definition |
| Kill chain phase | Kill chain name and phase name |
| External reference | URL, otherwise source name and external ID |

:::only{opencti=">=7.260811"}
The abstract of a Note also contributes to its ID, with its content and creation date.
:::

### Names and aliases

For Attack Patterns, Campaigns, Channels, Courses of Action, Events, Incidents, Infrastructures, Intrusion Sets, Languages, Malware, Narratives, Threat Actors, Tools, Vulnerabilities, identities (except Security Platforms) and locations, each alias is an identifier too. As a result:

- Creating a Malware named like the alias of an existing Malware updates the existing one.
- The names and aliases of a type form one set of unique values. Adding an alias that is already the name or an alias of another object of the same type fails with "This update will produce a duplicate". To bring the two together, [merge them](/opencti/use/enrich/merge/).

For administrators: generic aliases, such as "APT" or a product name, can make unrelated objects meet. The `aliases_identifier_rules` key (`ALIASES_IDENTIFIER_RULES`) restricts which aliases count as identifiers, with a list of regular expressions per type in lower case, such as `intrusion-set`. Without a rule for a type, every alias counts. See [Functional customization](/reference/opencti/configuration/#functional-customization) in the configuration reference.

## Properties that identify observables

Observables follow the ID contributing properties of the [STIX 2.1 specification](https://docs.oasis-open.org/cti/stix/v2.1/stix-v2.1.html), extended to the observable types that OpenCTI adds.

| Type | ID contributing properties |
| --- | --- |
| Domain name, Email address, Hostname, IPv4 address, IPv6 address, MAC address, URL, User agent, Text, Phone number, Cryptocurrency wallet, and the other single-value types | Value |
| File, Artifact, X.509 certificate | The first hash found, in this order: MD5, SHA-1, SHA-256, SHA-512, SHA3-256, SHA3-512, SSDEEP, SDHASH, TLSH, LZJD. Without a hash: name for a file, URL for an artifact, serial number or subject for a certificate |
| Autonomous system | Number |
| Directory | Path |
| Email message | Sender, subject and body |
| Mutex | Name |
| Network traffic | Start, end, source, destination, source and destination ports, protocols |
| Process | Command line, and the process ID (PID) when a command line is given |
| Software | Name (without case), CPE, SWID, vendor and version |
| User account | Account type, user ID and login |
| Windows registry key | Key and values |
| Bank account | IBAN |
| Payment card | Card number |
| Media content | URL |
| SSH key | SHA-256 fingerprint |
| Persona | Name and type |

Every hash of a file is an identifier. A file known by its MD5 and a file known by its SHA-256 become one object as soon as an import carries both hashes: the platform merges them. If they disagree on a hash of the same algorithm, it refuses the import with a "Hashes collision" error.

## How relationships are matched

A relationship is identified by its type, its source, its target, its start time and its stop time. To absorb small differences of dates between sources, the platform matches an incoming relationship with an existing one when:

- the type, the source and the target are the same, and
- the start time and the stop time, when the incoming relationship has them, are within 30 days of the existing ones.

An incoming relationship without start and stop times matches any relationship with the same type, source and target. A relationship also matches by its STIX ID, like any object. Sightings work the same way, with their first seen and last seen dates. Relationships created by [inference rules](/opencti/use/enrich/inferences/) exist once per source, type and target.

For administrators, three configuration keys change the matching, in [Functional customization](/reference/opencti/configuration/#functional-customization):

| Key | Variable | Default | Effect |
| --- | --- | --- | --- |
| `relations_deduplication:past_days` | `RELATIONS_DEDUPLICATION__PAST_DAYS` | 30 | How many days before the incoming dates still match |
| `relations_deduplication:next_days` | `RELATIONS_DEDUPLICATION__NEXT_DAYS` | 30 | How many days after the incoming dates still match |
| `relations_deduplication:created_by_based` | `RELATIONS_DEDUPLICATION__CREATED_BY_BASED` | false | When true, the date window only matches relationships of the same author: two sources keep two relationships |

A wider window merges more periods into one relationship; a narrower one keeps distinct periods of activity apart but multiplies relationships.

## What an incoming duplicate can change

When the platform updates an existing object, the [confidence levels](/opencti/concepts/reliability-confidence/) decide what changes. The confidence of the incoming data is its own confidence, capped by the max confidence level of the user or connector that sends it. The platform compares it with the confidence of the existing object. Observables have no confidence, so for them the incoming data always counts as equal or higher.

| Data | Incoming confidence equal or higher | Incoming confidence lower |
| --- | --- | --- |
| Single values, such as the description | Replaced by the incoming value, if it is not empty | Filled only if empty |
| Lists, such as aliases | Incoming values added | Filled only if empty |
| Labels and external references | Added | Added |
| Markings and other references | Added. An object keeps one marking per type, the highest: TLP:AMBER replaces TLP:GREEN, never the reverse | Filled only if empty |
| Author | Replaced only if the incoming confidence is strictly higher | Filled only if empty |
| Confidence | Set to the incoming confidence | Kept |
| Score, validity and revocation of an indicator | Updated, see [Indicator lifecycle](/opencti/use/analyze/indicators-lifecycle/) | Kept |

Three more rules complete the picture:

- **Nothing is removed.** An upsert adds and replaces; it never empties a field or removes a value. Only a synchronizer set to **Use perfect synchronization** replaces data exactly, removals included. See [Live streams](/opencti/use/ingest/streams/).
- **Identifiers and creators always accumulate.** Whatever the confidence, the source's STIX ID goes into **Other STIX IDs**, and the user or connector joins the **Creators**.
- **Dates only widen.** When the confidence allows the change, the original creation date keeps the oldest value, a relationship keeps the earliest start time and the latest stop time, and a sighting keeps the earliest first seen and the latest last seen. See [Meaning of dates](/opencti/concepts/dates/#how-a-new-import-moves-the-dates), which also explains the dates that do not widen.

:::only{opencti="<7.261030"}
In this version, one exception to "nothing is removed": a reference that holds a single value, such as the author, can be emptied by an incoming duplicate that comes without it. For the author, that takes a strictly higher confidence; for other single references, such as the parent directory of a file, an equal one.
:::

Sharing with organizations follows its own rule: an incoming duplicate only adds organizations when the sender has the **Restrict organization access** capability and the platform runs the Enterprise Edition.

:::example[A community feed meets Velvet Harbor]
Example CERT's analysts rated Velvet Harbor 75 and wrote its description. A community feed, whose account has a max confidence level of 40, sends Velvet Harbor with another description, the label `hospitality`, the alias "VH Crew" and the author "Community Feed".

The description, the aliases and the author stay as your analysts set them, because 40 is lower than 75. The label is added, because labels are always added. The feed's STIX ID goes into **Other STIX IDs**, and its account joins the **Creators**, so you can see that the feed also reported Velvet Harbor.
:::

## Standard STIX ID and the source's ID

This section is for developers who correlate OpenCTI data with a source or write connectors.

The **Standard STIX ID** is the ID OpenCTI computes, not always the ID the source used. A TAXII feed may call an object `malware--1234…`, while OpenCTI computes another ID from its name. That is what lets two sources that know the same malware under different IDs meet on one object. The computation is a UUID version 5 over the canonical JSON of the ID contributing properties, as STIX 2.1 recommends: Velvet Harbor is always `intrusion-set--4e78bb64-59c9-51e3-b1b9-ac8133e1c8e0`.

The source's ID is not lost: it goes into **Other STIX IDs**, and the platform finds the object by it on the next import. To correlate OpenCTI data with a source, look at **Other STIX IDs** and at the external references, not only at the standard ID.

When you edit a property that contributes to the ID, such as a name, the platform computes a new standard ID. If another object already has it, the change fails with "This update will produce a duplicate". For observables, the platform merges the two instead. The [errors reference](/reference/opencti/errors/) lists these errors with the others.

## What's next

- [Merge entities](/opencti/use/enrich/merge/): bring two objects together by hand.
- [Reliability and confidence](/opencti/concepts/reliability-confidence/): how confidence levels are set and computed.
- [Meaning of dates](/opencti/concepts/dates/): which dates widen on an import, and which identify objects.
- [Data processing](/opencti/concepts/data-processing/): where deduplication happens on the way in.
- [OpenCTI configuration reference](/reference/opencti/configuration/#functional-customization): the deduplication keys.
