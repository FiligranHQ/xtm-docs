---
title: Map JSON APIs to knowledge
description: Build an OpenCTI JSON mapper that turns the response of a partner's API into linked entities and relationships with JSON paths and identifiers, then use it in a JSON feed.
sidebar_position: 2
doc_type: how-to
products:
  - opencti
personas: [cti-lead, developer, admin]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/administration/json-mappers/
---

A JSON mapper turns what a partner's API returns, such as the threats it tracks and the addresses they use, into knowledge in OpenCTI: entities, the relationships between them, their dates and markings. You describe the layout of the response once: where the items are, which attribute each field fills, and how the items relate to each other. Then a [JSON feed](/opencti/use/ingest/json-feed/) calls the API on a schedule and applies the mapper to each response. Build one mapper per JSON layout. If you have not built a [CSV mapper](/opencti/configure/mappers/csv/) yet, skim it first: representations, reference attributes and default values work the same way.

## Before you start

- Your role has the **Manage data mappers** capability.
- You have a sample of the API response, no more than 50 lines and 1 MB, to test the mapper. Ask the provider for one, or save one from their documentation.

## The JSON paths you need

A JSON path says where a value sits in the document. Four forms cover most feeds:

| Path | Reads |
| --- | --- |
| `$` | The whole document, or the current item |
| `$.name` | The field `name` |
| `$.threats[*]` | Every element of the array `threats` |
| `$.threats[*].addresses[*].ip` | The field `ip` of every address of every threat |

OpenCTI evaluates the paths with the [JSONPath-Plus](https://github.com/JSONPath-Plus/JSONPath) library, so filters such as `$.threats[?(@.score > 50)]` work too.

## How a mapper reads JSON

A JSON document is a tree, while a mapper is a flat list of **representations**, one per kind of object. Three paths connect the two:

1. The **Entity path mapping** of a representation selects the items it turns into objects, for example `$.threats[*]` for every element of a `threats` array.
2. Each attribute has a **JSON Path** that reads its value inside one item: `$` is the item itself.
3. The **Identifier** of a representation is a path to a value that names each item, such as `$.name`. Other representations use it to find that item when they refer to it: a relationship, or an attribute such as the author. Items that share an identifier make a single object: the first one read is kept.

::diagram{id=opencti-json-mapper-paths}

The identifier is what CSV mappers do not need: in a CSV file, everything on a row belongs together; in JSON, a relationship must say which items it links.

## An example

:::example[Read the threats and addresses of a partner API]
The partner API `https://api.example.com/v1/threats` returns the threats it tracks, with the IP addresses each one uses:

```json
{
  "threats": [
    {
      "name": "Velvet Harbor",
      "first_seen": "2025-03-04T00:00:00Z",
      "addresses": [
        { "ip": "203.0.113.31" },
        { "ip": "198.51.100.44" }
      ]
    }
  ]
}
```

To create an Intrusion Set, two IPv4 addresses and a relationship between each address and the intrusion set, the mapper `Threats and addresses` has three representations:

| Representation | Entity path mapping | Identifier | Attributes |
| --- | --- | --- | --- |
| Intrusion Set | `$.threats[*]` | `$.name` | name: `$.name`; first seen: `$.first_seen` |
| IPv4 address | `$.threats[*].addresses[*]` | `$.ip` | value: `$.ip` |
| Relationship **related to** | `$.threats[*]` | | `from`: the IPv4 address representation, identifier `$.addresses[*].ip`; `to`: the Intrusion Set representation, identifier `$.name` |

The relationship reads each threat. Its `from` path returns the two IP addresses of the threat, which match the identifiers of two IPv4 objects; its `to` path returns the name of the threat. The test finds three entities, Velvet Harbor, 203.0.113.31 and 198.51.100.44, and two relationships: each address related to Velvet Harbor.
:::

An identifier can combine several paths separated by commas, when one value is not enough to tell items apart.

If you leave the identifier of a reference empty, the reference points to every object of that representation. That suits a single author for the whole document, for example.

## Create a mapper

::::steps

1. Open :menu[Data > Processing > JSON Mappers] and click **Create a JSON mapper**.
2. Give the mapper a **Name**. It is the only field required to save.
3. Next to **Representations for entity**, click the add button, expand the new representation and pick its **Entity type**.
4. Fill the **JSON Path** of **Entity path mapping**, and the **JSON Path** of **Identifier** if other representations refer to this one.
5. Fill the **JSON Path** of each attribute. Mandatory attributes are marked with an asterisk.
6. For an attribute that points to another representation, such as the author, pick it in **Representation entity**. Then fill the **JSON Path** that, read from the current item, returns the identifiers of the items to link.

   :::only{opencti=">=7.260811"}
   Each representation you pick gets its own path, under it, labelled **Identifier** followed by the name of the representation: until you pick one, the attribute shows **Select a representation first**.
   :::

   :::only{opencti="<7.260811"}
   The attribute has one **JSON Path** for its identifier, whatever the representations you pick.
   :::

7. Next to **Representations for relationship**, add each relationship: its type, its entity path, and the representations and identifiers at each end in `from` and `to`.
8. Click **Create**.

::::

::screenshot{id=opencti-json-mapper-form alt="The JSON mapper edition panel with an IPv4 address representation, its entity path mapping, its identifier path and the JSON path of its value."}

## Read dates and lists, fill missing values

The settings button next to the path of an attribute opens **Attribute mapping configuration**, with the same options as in CSV mappers: a **Date pattern** when dates are not in ISO 8601, in [Luxon](https://moment.github.io/luxon/#/parsing?id=table-of-tokens) tokens such as `dd/MM/yyyy`; a **List separator** when one string holds several values; and a **Default value** for when the field is missing. When a path returns an array, a multiple attribute such as the labels takes every value, without a separator. Markings take a policy: **Use default marking definitions of the user** or **Let the user choose marking definitions**. See [Map CSV files to knowledge](/opencti/configure/mappers/csv/#fill-missing-values).

## Check and test the mapper

The list shows the **Validity** of each mapper: a mapper is valid when every representation has all its mandatory attributes mapped. Only valid mappers can be used.

:::steps

1. In the creation or edition panel, click **Test**. The **Testing JSON mapper** dialog opens.
2. Select your sample file and click **Test**.
3. Read the number of **Entities** and **Relationships** under **Objects found**, and the STIX 2.1 bundle produced.

:::

If a relationship is missing from the result, check that its identifier paths return exactly the values the identifiers of the other representations return. A reference that can hold only one object, such as the author, fails when its path matches several.

:::warning
The test checks the mapping only, not the values or the settings of the entity types, so an object of the test result can still be refused when the feed runs. It also blocks the platform while it runs: keep the sample small.
:::

## Use the mapper

Pick the mapper when you create a [JSON feed](/opencti/use/ingest/json-feed/). The feed calls the API on a schedule and applies the mapper to each response.

The menu of each mapper in the list offers **Update**, **Duplicate**, **Delete** and **Export**. When the provider changes its API, **Duplicate** the mapper first: the **Duplicate a JSON mapper** panel opens prefilled, so you can adapt and test the copy while the feed keeps running on the original. Load an exported mapper on another platform with **Import a JSON mapper**.

## What's next

- [Import from a JSON API](/opencti/use/ingest/json-feed/), which run the mapper.
- [Map CSV files to knowledge](/opencti/configure/mappers/csv/), for CSV files.
- [Customize entity types](/opencti/configure/entities/) for the platform-wide default values.
