---
title: Map CSV files to knowledge
description: Build an OpenCTI CSV mapper that turns the rows of a partner's CSV file into linked entities and relationships, test it on a sample, and use it for file imports and CSV feeds.
sidebar_position: 1
doc_type: how-to
products:
  - opencti
personas: [cti-lead, cti-analyst, admin]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/administration/csv-mappers/
---

A CSV mapper turns a spreadsheet your team receives, such as a partner's list of malicious addresses, into knowledge: entities, the relationships between them, their dates and their markings, without anyone typing them in. You describe the layout of the file once; then analysts pick the mapper when they [import a CSV file](/opencti/use/ingest/files/), and [Import a CSV file from a URL](/opencti/use/ingest/csv-feed/) apply it to a remote file on a schedule. Build one mapper per CSV layout you receive.

## Before you start

- Your role has the **Manage data mappers** capability. Analysts who only use the mapper do not need it.
- You have a sample of the file, no more than 100 lines and 1 MB, to test the mapper.
- You know what each row says: which entities, which relationships between them. A mapper cannot invent knowledge that the file does not hold, but it can add fixed values, such as a type or a marking, to every row.

## How a mapper reads a file

A mapper is a list of **representations**. Each representation describes one object that every row produces: an entity, such as an IPv4 address, or a relationship between two entities of the same row. For each attribute of the object, you give the column that holds it, by its letter as in a spreadsheet: A, B, C.

For example, a file whose rows hold an IP address in column A, a malware name in column B and a first-seen date in column C needs three representations: an **IPv4 address** reading its value from A, a **Malware** reading its name from B, and a relationship from the malware to the address, reading its start time from C.

Some attributes are required by the data model, and the form marks them with an asterisk: a **Malware** needs **Is family**, which a feed rarely gives. When the file has no column for such an attribute, give it a default value (see [Fill missing values](#fill-missing-values)); otherwise the mapper stays invalid.

Attributes that point to another object, such as the author, the markings or the labels, also come from representations: you add a representation of that object, then pick it in the attribute.

| Attribute | Representation to add |
| --- | --- |
| Author | Individual, Organization or System |
| Markings | Marking definition |
| Labels | Label |
| External references | External reference |
| Kill chain phases | Kill chain phase |

## Create a mapper

:::steps

1. Open :menu[Data > Processing > CSV Mappers] and click **Create a CSV mapper**.
2. Give the mapper a **Name** that says whose file it reads. It is the only field required to save, so you can build the mapper in several sessions.
3. Describe the file:
   - turn on **My CSV file contains headers** if the first line holds column names: the mapper skips it, and any comment lines before it;
   - pick the **CSV separator**: **Comma**, the default, **Semicolon** or **Pipe**;
   - in **Char to escape line**, give the character that starts lines to skip, such as `#` for comments.
4. Next to **Representations for entity**, click the add button, expand the new representation and pick its **Entity type**.
5. For each attribute, pick the **Column index** that holds it. Mandatory attributes are marked with an asterisk.
6. Repeat for every entity of the row. Add the reference representations, such as an organization for the author, then pick them in the attribute with **Representation entity**.
7. Next to **Representations for relationship**, add each relationship, pick its type, then pick the representations at each end in `from` and `to`.
8. Click **Create**.

:::

::screenshot{id=opencti-csv-mapper-form alt="The CSV mapper edition panel with the header switch, the CSV separator choice, an IPv4 address representation reading its value from column A, and a relationship representation linking it to a malware."}

## Map different types from the same columns

When one column holds the type of each row, for example `IPv4-Addr` or `Url` in column A and the value in column B, turn on **Entity dynamic mapping** in each representation. Pick the **Column index** to test, the **Operator** (**Equal** or **Not equal**) and the **Value** to compare with. The representation then only reads the rows that pass the test: an IPv4 representation for rows where A equals `IPv4-Addr`, a URL representation for rows where A equals `Url`, both reading the value from column B.

## Read dates and lists

The settings button next to the column of an attribute opens **Attribute mapping configuration**:

- **Date pattern**, for dates: the mapper reads ISO 8601 dates such as `2026-01-15` by default; give your own format otherwise, in the tokens of the [Luxon](https://moment.github.io/luxon/#/parsing?id=table-of-tokens) library: `dd` for the day, `MM` for the month, `yyyy` for the year, `HH:mm` for the time. `12.01.2026` reads with `dd.MM.yyyy`, `01/12/2026 14:30` with `MM/dd/yyyy HH:mm`. Mind the case: `DD.MM.YYYY`, the example the interface gives, reads nothing. A date the pattern cannot read is left empty, without an error.
- **List separator**, for attributes with several values: the character between values in the cell, such as `|` or `+`.

## Fill missing values

When a cell is empty, or the file has no column for an attribute, the mapper can put a default value instead. There are two levels:

| Where | Applies to |
| --- | --- |
| Default value of the attribute in :menu[Settings > Customization > Entity types] | Every object of that type created on the platform, by forms and by mappers. See [Customize entity types](/opencti/configure/entities/#make-attributes-mandatory-and-set-default-values) |
| **Default value** in **Attribute mapping configuration** of the mapper | This mapper only, and it wins over the platform default |

The panel says whether a platform default exists for the attribute, so you know what you override. A default value set at either level also makes a mandatory attribute count as mapped.

Markings work differently: you choose a policy, not a marking. Decide it with the senders of the file in mind, because it controls who sees what the import creates.

- **Use default marking definitions of the user**: objects get the default markings of the user who imports the file. Suits files that always carry the same sharing level.
- **Let the user choose marking definitions**: the user picks the markings, among those they can see, when they select the mapper. Suits files whose senders set a TLP per delivery.

:::example[Import the Velvet Harbor infrastructure list]
Each month, Example CERT sends its members a CSV file of the addresses Velvet Harbor malware calls, with the date each one was first seen, written the European way. The team lead of a member CERT wants analysts to load it in a minute, under the TLP the sender sets.

```csv
# Velvet Harbor infrastructure shared by Example CERT, January 2026
ip,malware,first_seen
203.0.113.31,Velvet Loader,12.01.2026
198.51.100.44,Harbor Stealer,14.01.2026
203.0.113.77,HarborGate,19.01.2026
```

The mapper that reads it:

- **Name**: `Example CERT infrastructure list`
- **My CSV file contains headers**: on, **CSV separator**: **Comma**, **Char to escape line**: `#`, so the comment line and the header line are both skipped
- Representation for entity **IPv4 address**: **Value** in column **A**
- Representation for entity **Malware**: **Name** in column **B**; **Is family** has no column, so open its **Attribute mapping configuration** and turn on **Default value**: each name in the file is a malware family
- Representation for relationship **communicates with**: `from` the Malware representation, `to` the IPv4 address representation, **Start time** in column **C**, with the **Date pattern** `dd.MM.yyyy` in its **Attribute mapping configuration**
- Markings of each representation: **Let the user choose marking definitions**, so the analyst who imports the file applies the TLP the sender asked for

The list shows the mapper as valid. The test finds six entities, three addresses and three malware families, and three relationships, such as Velvet Loader communicates with 203.0.113.31 since 12 January 2026. Without the date pattern, or with `DD.MM.YYYY`, the relationships have no start time; without the default on **Is family**, the mapper is invalid and cannot be picked.
:::

## Check and test the mapper

The list of mappers shows the **Validity** of each one: a green check when every representation has the attributes the data model requires, from a column or a default value; a red cross otherwise. Only valid mappers can be picked for an import or a feed.

:::steps

1. In the creation or edition panel, click **Test**. The **Testing csv mapper** dialog opens.
2. Select your sample file and click **Test**.
3. Read the result: under **Objects found**, the number of **Entities** and **Relationships**, then the STIX 2.1 bundle the mapper produced.

:::

:::warning
The test only checks the mapping. It does not check the values, such as a valid IP address, or the attributes your platform made mandatory in the entity types, such as a description, so an object of the test result can still be refused at import. The test also blocks the platform while it runs: keep test files under 100 lines and 1 MB.
:::

## Use the mapper

When a user uploads a CSV file in :menu[Data > Import] or in the **Data** tab of an entity, they pick the mapper to apply.

:::only{opencti=">=7.260811"}
By default the result lands in a new [draft](/opencti/use/ingest/drafts/), for review before it reaches the knowledge base. The user can choose a [workbench](/opencti/use/ingest/workbench/) instead by changing the validation mode.
:::

:::only{opencti="<7.260811"}
By default the result lands in a new [workbench](/opencti/use/ingest/workbench/), for review before it reaches the knowledge base.
:::

To read the same layout from a URL on a schedule, pick the mapper in a [CSV feed](/opencti/use/ingest/csv-feed/).

For platform engineers: the built-in connector that runs CSV imports is set by the `import_csv_built_in_connector` keys ([Engines, schedules and managers](/reference/opencti/configuration/#engines-schedules-and-managers)), and `app:csv_ingestion:max_bundle_size` (1000 objects by default) sets the size of the bundles a CSV import sends to the workers ([Technical customization](/reference/opencti/configuration/#technical-customization)).

## Duplicate, export and import mappers

The menu of each mapper in the list offers **Update**, **Duplicate**, **Delete** and **Export**.

- **Duplicate** a mapper before changing a layout that works: the **Duplicate a CSV mapper** panel opens prefilled, with " - copy" after the name. Test the copy on the new file, then switch the feed or tell analysts.
- **Export** a mapper to a JSON file, and load it on another platform with **Import a CSV mapper**. Files exported from OpenCTI versions older than 6.6 are refused.

## What's next

- [Import files](/opencti/use/ingest/files/), where analysts apply a mapper.
- [Import a CSV file from a URL](/opencti/use/ingest/csv-feed/) to read a remote CSV file on a schedule.
- [Map JSON APIs to knowledge](/opencti/configure/mappers/json/), the same idea for JSON.
