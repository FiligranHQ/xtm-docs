---
title: Nested objects
description: Why the author, markings, labels and external references of an OpenCTI object are shared objects of their own, what that changes for your analysis, and how STIX bundles convert.
sidebar_position: 3
doc_type: explanation
products:
  - opencti
personas: [cti-analyst, developer]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/nested/
---

In OpenCTI, the author, the markings, the labels and the external references of an object are objects of their own, linked to it. Example CERT is one Organization that authors hundreds of reports, and `hotel-sector` is one label carried by every object of the sector. That is why you can open a label or an author and list everything that carries it, why renaming a label renames it everywhere, and why an object never holds two TLP markings. This page explains that model and its consequences for your work; developers find at the end how it maps to Structured Threat Information Expression (STIX) 2.1 bundles.

## What it is good for

| Goal | Example |
| --- | --- |
| Pivot on context | From the label `hotel-sector`, list every report, malware and indicator of the sector |
| Follow a source | From the Organization Example CERT, list everything it authored, and filter any list on **Author** |
| Keep one copy of shared context | One TLP:AMBER marking and one Example CERT, referred to by thousands of objects |
| Count on context | Chart the indicators of each author, or the reports of each label, on a dashboard |
| Exchange plain STIX | Import and export standard STIX 2.1 without thinking about the conversion |

## How OpenCTI stores the context of an object

::diagram{id=opencti-nested-objects-refs}

In a STIX bundle, an object carries part of its context inside itself: a reference to its author, references to its markings, its labels, its external references. OpenCTI turns each of them into a separate object and a link to it. You see these links as fields of the object, in its **Basic information** card and in its **External references** card:

| Context | Object in OpenCTI | Shown on the object as |
| --- | --- | --- |
| Author | An identity, such as an Organization, an Individual or a System | **Author** |
| Markings | A marking definition, such as TLP:AMBER | **Marking** |
| Labels | A label | **Labels** |
| External references | An external reference: a source name, a URL, an external ID | **External references** |
| Kill chain phases, of attack patterns, malware, tools, indicators, infrastructures and relationships | A kill chain phase | **Kill chain phases** |
| Content of a report or a case | Any object | The content tabs of the container, see [Containers](/opencti/concepts/containers/) |
| Organizations an object is shared with | An Organization | **Organizations sharing** |

Observables have their own links, stored the same way: the address a domain name resolves to, the source and destination of network traffic, the parent directory of a file, and others.

::screenshot{id=opencti-nested-objects-basic-information alt="The Basic information card of Velvet Harbor shows its marking TLP:AMBER, its author Example CERT and its labels hotel-sector and phishing, each a link to an object of its own."}

## What it changes for your work

1. **Context objects are shared.** A label or an external reference exists once. Changing it, for example correcting the URL of an Example CERT advisory, changes it on every object that refers to it.
2. **Context is deduplicated like any object.** Two imports of the same external reference, with the same URL, give one object; two feeds that send the label `Hotel-Sector` and `hotel-sector` give one label. See [Deduplication](/opencti/concepts/deduplication/).
3. **An object holds one marking per type.** When an update adds TLP:AMBER to an object marked TLP:GREEN, the object becomes TLP:AMBER; a later TLP:GREEN does not lower it. Markings are what restrict access, see [Markings](/opencti/administer/markings/).
4. **An object has one author.** An incoming duplicate replaces the author only with a strictly higher confidence, so the author of a report stays stable when several feeds relay it. See [What an incoming duplicate can change](/opencti/concepts/deduplication/#what-an-incoming-duplicate-can-change).
5. **These links are not knowledge.** They do not appear among the relationships of the **Knowledge** tab. To say that Velvet Harbor targets the hospitality sector, create a "targets" relationship to the Sector: a label `hotel-sector` helps you find objects, but states nothing about the threat.

:::example[Find everything Example CERT said about the hotel sector]
You want every object Example CERT published on the hospitality sector, to brief a new member hotel chain. In :menu[Data > Entities], filter on **Author** is Example CERT and **Label** is `hotel-sector`: the reports and the Intrusion Set Velvet Harbor come up together, whatever their type, because they all link to the same Example CERT and the same label. If Example CERT renames its advisory site, one change to each external reference updates every object that cites it.
:::

## For developers: nested references in STIX bundles

STIX holds this context in two forms. **Nested references** point to other objects by their identifier, in properties ending in `_ref` or `_refs`. **Nested objects** are embedded whole, such as an external reference or a kill chain phase. Labels are plain strings. An incoming Intrusion Set:

```json
{
  "type": "intrusion-set",
  "spec_version": "2.1",
  "id": "intrusion-set--4e78bb64-59c9-51e3-b1b9-ac8133e1c8e0",
  "created": "2025-10-22T09:00:00.000Z",
  "modified": "2025-10-22T09:00:00.000Z",
  "name": "Velvet Harbor",
  "aliases": ["FIN99", "Harbor Mantis"],
  "created_by_ref": "identity--ac6f7b3a-7aa5-558d-a1bd-4a4f52840f9e",
  "object_marking_refs": ["marking-definition--f88d31f6-486f-44da-b317-01333bde0b82"],
  "labels": ["hotel-sector", "phishing"],
  "external_references": [
    {"source_name": "Example CERT", "url": "https://cert.example.com/advisories/velvet-harbor"}
  ]
}
```

`created_by_ref` points to the Organization Example CERT and `object_marking_refs` to TLP:AMBER. The platform stores each reference as a reference relationship, with these types and these input fields in the GraphQL API:

| In a bundle | Relationship type | GraphQL input field |
| --- | --- | --- |
| `created_by_ref` | `created-by` | `createdBy` |
| `object_marking_refs` | `object-marking` | `objectMarking` |
| `labels` | `object-label` | `objectLabel` |
| `external_references` | `external-reference` | `externalReferences` |
| `kill_chain_phases` | `kill-chain-phase` | `killChainPhases` |
| `object_refs` | `object` | `objects` |
| `granted_refs`, in the OpenCTI extension | `granted` | `objectOrganization` |

**On import**, the platform resolves each reference to an existing object or to an object of the same bundle. A relationship whose source or target is missing waits and is sent again; another missing reference delays the object for a few attempts, then the platform writes it without that reference. See [The path of a bundle](/opencti/concepts/data-processing/#the-path-of-an-object).

**On export**, the platform writes the references back as standard STIX properties. What STIX 2.1 does not allow on a cyber observable, such as an author, labels or a score, goes into the OpenCTI extensions of the object. The OpenCTI extension `extension-definition--ea279b3e-5c71-4632-ac08-831c66a786ba` also carries the internal ID and the platform dates. Velvet Harbor's command and control address, trimmed:

```json
{
  "type": "ipv4-addr",
  "spec_version": "2.1",
  "id": "ipv4-addr--570130c0-7daf-5d1e-992f-58b53b447312",
  "value": "203.0.113.10",
  "extensions": {
    "extension-definition--ea279b3e-5c71-4632-ac08-831c66a786ba": {
      "extension_type": "property-extension",
      "id": "8a3d108f-908c-4833-8ff4-4d6fc996ce39",
      "type": "IPv4-Addr",
      "created_at": "2026-01-12T09:15:00.000Z",
      "updated_at": "2026-01-12T09:15:00.000Z"
    },
    "extension-definition--f93e2c80-4231-4f9a-af8b-95c9bd566a82": {
      "extension_type": "property-extension",
      "labels": ["c2"],
      "description": "Command and control server of Velvet Harbor.",
      "score": 80,
      "created_by_ref": "identity--ac6f7b3a-7aa5-558d-a1bd-4a4f52840f9e"
    }
  }
}
```

A reference that holds one value, such as `created-by`, accepts one relationship per object.

:::only{opencti=">=7.261030"}
Adding a `created-by` relationship through `relationAdd` on an object that already has an author replaces the author.
:::

:::only{opencti="<7.261030"}
Adding a `created-by` relationship through `relationAdd` on an object that already has an author fails with "Cant add another relation on single ref": change the author with `fieldPatch` on `createdBy` instead.
:::

## What's next

- [Containers](/opencti/concepts/containers/): how `object_refs` turns a report into a container of knowledge.
- [Deduplication](/opencti/concepts/deduplication/): how references are matched and what an incoming duplicate may add.
- [Markings](/opencti/administer/markings/): how marking definitions restrict access.
- [Export](/opencti/use/share/export/): export objects and lists as STIX 2.1.
- [Data model](/opencti/concepts/data-model/): the families of objects and relationships.
