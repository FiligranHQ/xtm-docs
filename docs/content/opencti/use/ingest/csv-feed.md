---
title: Import a CSV file from a URL
description: Import a CSV file served at a URL on a schedule, mapping its columns to entities and relationships with a CSV mapper, and test the result before you create the feed.
sidebar_position: 6
doc_type: how-to
products:
  - opencti
personas: [cti-lead, cti-analyst, admin]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/import/csv-feed/
---

A CSV feed downloads a CSV file from a URL on a schedule and turns its lines into entities and relationships, following a CSV mapper. Use it for the indicator lists and blocklists that many sources publish as CSV: the team gets them in the knowledge base, linked and kept up to date, without anyone copying lines by hand. For how imported data reaches the knowledge base, read [How intelligence gets into OpenCTI](/opencti/use/ingest/).

## Before you start

- Your role has the **Manage ingestion** capability. To create or change a CSV mapper, it also needs **Manage data mappers**.
- You have the URL of the file and the credentials it requires.
- You know how the file maps to entities. Either a [CSV mapper](/opencti/configure/mappers/csv/) exists for it, or you write one inline in the feed.
- To let the form create a service account, an administrator has set **Default group for Service accounts** in :menu[Settings > Security > Policies].

## How a CSV feed runs

At each run, the feed downloads the whole file. If the file has not changed since the last run, nothing is imported. Otherwise every line goes through the mapper again: OpenCTI recognizes the objects it already has, so a line imported before updates its object instead of creating a duplicate.

A CSV feed runs at most every 5 minutes, whatever its schedule. Its default **Schedule period** is 1 hour.

:::only{opencti=">=7.261127"}
A run sends the objects to the workers in bundles of up to 1,000 objects by default. If you **Stop** the feed during a run, the run stops before its next bundle: the bundles already sent are imported, the rest of the file is not. The feed does not record the file as imported, so when you start it again, its next run imports the whole file, even if the file has not changed.
:::

## Create the feed

Open the creation form:

::::only{opencti=">=7.260811"}
Open :menu[Integrations], select the **Available** tab, and click **Create** on the **CSV Feed** card.
::::

::::only{opencti="<7.260811"}
Open :menu[Data > Ingestion > CSV Feeds] and click the create button.
::::

::screenshot{id=opencti-ingest-csv-feed-create alt="The Create a CSV Feed form has two tabs, Overview and Inline csv mapper; Overview holds the name, the CSV URL and the service account, and Verify and Create stay disabled until the form is complete."}

Then fill it in:

:::steps

1. Enter a **Name** and a **Description**. With an automatic service account, the name also names the account.
2. Choose a **Schedule period**.
3. Enter the **CSV URL**.
4. Keep **Automatically create a service account** on and set its confidence level, or turn it off and choose an existing account.
5. Choose the mapper:
   - to use a mapper of the platform, turn on **Existing csv mappers** and choose it in **CSV Mappers**;
   - to write one for this feed only, leave the switch off and fill in the **Inline csv mapper** tab, as described in [Map CSV files to knowledge](/opencti/configure/mappers/csv/).
6. If the mapper lets the user choose markings, choose them in **Marking definition levels**. If it uses the default markings of the user, the markings of the account that writes the data apply.
7. Choose the **Authentication type** and enter the credentials: **Basic user / password**, **Bearer token** or **Client certificate**, or **None**.
8. Keep **Verify SSL certificate** on, unless the server uses a certificate your platform cannot check.
9. Click **Verify**, then **Test** in the dialog. The test reads the first 10 lines of the file and shows the entities and relationships it would create, in STIX.
10. If the result is what you expect, close the dialog and click **Create**. **Create** stays disabled until a test finds at least one entity.
11. **Start** the feed.

:::

:::example[Import the ransomware servers listed by a research lab]
Bluefin Labs, a fictional threat research lab, updates every hour a public CSV file of the servers that ransomware families use, with the date each one was first seen:

```csv
# Bluefin Labs ransomware servers - updated hourly
ip,malware,first_seen
203.0.113.58,Velvet Loader,08.01.2026
198.51.100.44,Harbor Stealer,12.01.2026
192.0.2.140,HarborGate,14.01.2026
```

The platform already has the CSV mapper `IP and malware feed` for this layout:

| Mapper setting | Value |
| --- | --- |
| **My CSV file contains headers** | On, so the `ip,malware,first_seen` line is skipped |
| **CSV separator** | **Comma** |
| **Char to escape line** | `#`, so the comment line is skipped |
| IPv4 address | Value in column A |
| Malware | Name in column B, with the default values `ransomware` for **Malware types** and true for **Is family** |
| Relationship `related-to` | From the IPv4 address to the malware, start time in column C with the **Date pattern** `DD.MM.YYYY` |

Create the feed with:

- **Name**: `Bluefin Labs ransomware servers`
- **Schedule period**: **1 hour**, the update rhythm of the file
- **CSV URL**: `https://feeds.bluefin-labs.example/ransomware-servers.csv`
- **Automatically create a service account**: on, with a confidence level of 40. The lab is new to the team, so what it sends never overwrites what your analysts or better-known sources wrote
- **Existing csv mappers**: on, and **CSV Mappers**: `IP and malware feed`
- **Authentication type**: **None**, as the file is public

**Test** finds six entities, three addresses and three malware, and three relationships, such as 203.0.113.58 related to Velvet Loader since 8 January 2026. When Bluefin Labs adds a line, the next run imports the file again: the existing addresses and malware are recognized, and only the new line creates objects.
:::

## Change a running feed

To try a new configuration without stopping the feed that works, open the actions menu (⋮) of the feed, on the feed's own page from version 7.260811, and choose **Duplicate**. The creation form opens filled with the current settings: change what you need, run the test again, create the copy, then stop the original once the copy works.

An exported CSV feed carries its mapper inline. When you import it on another platform, the mapper appears in the **Inline csv mapper** tab.

## Check that it works

::::only{opencti=">=7.261030"}
Open the feed from the **Deployed** tab of :menu[Integrations]. After the first run, **Overview** shows the last run and the hash of the file imported, **Works** shows the import with its errors, and **Logs** shows each fetch with the number of objects sent.
::::

::::only{opencti=">=7.260811 <7.261030"}
Open the feed from the **Deployed** tab of :menu[Integrations]. After the first run, its page shows the last run and the hash of the file imported, then the works of the feed with their errors.
::::

::::only{opencti="<7.260811"}
The feed imports through a connector named after it: follow its progress in :menu[Data > Ingestion > Connectors].
::::

A line the mapper cannot read appears as an error of the work, with the reason. Fix the mapper, then **Reset state** to import the whole file again.

:::only{opencti=">=7.261127"}
A CSV URL on your internal network, such as a private address, may be refused: when your administrator blocks internal destinations, each fetch fails and **Logs** shows an error that names the host. Ask them to add the host to the allow list, as described in [Tune feeds and streams](/opencti/use/ingest/feed-options/#internal-destinations).
:::

## What's next

- [Map CSV files to knowledge](/opencti/configure/mappers/csv/), to map columns, split multiple values and set defaults.
- [Manage a feed](/opencti/use/ingest/#manage-a-feed): reset, export or share the feed on the XTM Hub.
- [Tune feeds and streams](/opencti/use/ingest/feed-options/#csv-feeds), for the minimum interval and the bundle size.
