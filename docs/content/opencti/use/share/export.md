---
title: Export knowledge to a file
description: Export an OpenCTI entity, a list or part of a container to a STIX, CSV, text or PDF file, choose what markings it may contain, and download it from the platform.
sidebar_position: 2
doc_type: how-to
products:
  - opencti
personas: [cti-analyst]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/export/
---

Export turns what you know into a file you can hand over: a report as STIX 2.1 for a partner CERT's tools, a spreadsheet of indicators for the SOC, or a finished advisory in PDF. For each file, you decide the highest marking its content may carry, so that it holds nothing your sharing agreement forbids, and the marking of the file itself, so that only the right colleagues can download it from the platform. To publish data continuously instead, see [Share knowledge with your security tools](/opencti/use/share/feeds/).

## Before you start

- Your role has the **Generate knowledge export** capability, to create exports, and the **Download knowledge export** capability, to download them.
- An export connector for the format you want runs on the platform, such as `ExportFileStix` for STIX 2.1, `ExportFileCsv` for CSV or `ExportFileTxt` for text. Your administrator deploys them from the [connector catalog](/opencti/use/ingest/connectors/). The formats offered are those of the running connectors.
- You are not in a draft: exports are not available in drafts.

## Choose what the file contains

| Setting | What it does |
| --- | --- |
| **Export type** | **Simple export (just the entity)** exports the entity alone. **Full export (entity and first neighbours)** adds every entity directly related to it |
| **Content max marking definition levels** | Leaves out what is marked higher. Choose one level per marking type: with TLP:GREEN, objects marked TLP:AMBER or TLP:RED are left out. For a marking type you leave empty, such as PAP, your own maximum shareable marking of that type applies, and objects of a type you may not share at all are always left out. You can only choose up to your own maximum shareable markings |
| **File marking definition levels** | Marks the file itself: only users who have these markings can see and download it |

See [Markings](/opencti/administer/markings/) for maximum shareable markings.

## Export an entity

:::steps

1. Open the entity and click the export button at the top of the page, **Generate an export**.

   ::screenshot{id=opencti-share-export-format alt="The Generate an export window opens on the Format step, with one card per available format, such as PDF and HTML."}

2. Choose the format.
3. Choose the **Connector** that produces the file, and the **Export type**.
4. Set the **Content max marking definition levels** and the **File marking definition levels**.
5. Click **Create**.

:::

The export runs in the background. When it ends, the file appears in **Exported files**, in the **Data** tab of the entity. Click it to download it.

:::example[Send the Velvet Harbor report to a partner CERT as STIX]
Saltmarsh CERT asks for the report **Velvet Harbor targets hotel booking systems** to load it into its own tools, and your sharing agreement with them stops at TLP:GREEN. Open the report and click **Generate an export**:

- Format: the STIX 2.1 format, `application/vnd.oasis.stix+json`.
- **Connector**: the STIX export connector, such as `ExportFileStix`.
- **Export type**: **Full export (entity and first neighbours)**, so the file carries the objects of the report with it.
- **Content max marking definition levels**: **TLP:GREEN**.
- **File marking definition levels**: **TLP:GREEN**.

The file holds the report, its sector, countries, techniques, indicators and malware such as Velvet Loader. The intrusion set Velvet Harbor and the malware Harbor Stealer, both marked TLP:AMBER, are left out, so check that the file still makes sense before you send it.
:::

## Export a list

:::steps

1. Open a list, such as :menu[Analyses > Reports], and filter it. To export some rows only, select them.
2. Click the export button of the list. The **Exports list** panel opens with the earlier exports of this list.
3. Click **Generate an export**.
4. Choose the **Export format** and the markings.

   ::::only{opencti=">=7.260811"}
   For CSV, **Choose column to export**: **All attributes**, or **Current view** for the columns shown in the list.
   ::::

5. Click **Create**. The file appears in the **Exports list** when it is ready; click it to download it.

:::

:::example[Give the detection team the hotel-sector indicators as CSV]
The SOC wants a spreadsheet of the indicators the CTI team tagged for the hospitality sector. In :menu[Observations > Indicators], add **Label**, operator **Equals**, value `hotel-sector`, and keep in the list only the columns the SOC needs. Click the export button, then **Generate an export**, and choose:

- **Export format**: **CSV**
- **Content max marking definition levels**: **TLP:AMBER**, the highest level the SOC may receive.

::::only{opencti=">=7.260811"}
- **Choose column to export**: **Current view**, so the file has the columns of your view only.
::::

The CSV file lists the hotel-sector indicators, such as `reservations-velvet.example`, `203.0.113.31` and `checkin-velvet.example`.
:::

To export part of a container, such as the attack patterns and indicators of a report for a detection team, open its **Entities** tab, select the objects, and export them the same way.

## Generate a finished intelligence document

In reports, groupings and cases, an Enterprise Edition platform can fill a fintel template with the container's data and produce an HTML or PDF document. Your administrator sets the templates per entity type in [entity type customization](/opencti/configure/entities/), and their look in [Brand your PDF reports](/opencti/configure/fintel-designs/).

:::steps

1. Open the container and click **Generate an export**. You can also start from its **Content** tab.
2. Choose **HTML** or **PDF**.
3. Choose **Generate FINTEL from template** as the **Connector**, then the **Template**.

   ::::only{opencti=">=7.261030"}
   For a PDF, you can also choose **HTML content files to PDF** with **Export as fintel** turned on, then the **Template**, the **Fintel design** and the **Page options**: **Include cover page**, **Include back page** and **Remove empty sections**.
   ::::

4. Enter the **Export file name**, the file markings and the **Content max marking definition levels**: a template can list the container's objects, such as its indicators, and this setting leaves out those marked higher.
5. Click **Create**.

:::

An HTML document opens in the **Content** tab of the container, where you can edit it. To turn an edited HTML file into a PDF, export it again with the **HTML content files to PDF** connector and choose it in **File to export**. Both built-in connectors need the **Upload knowledge files** capability. From the **Content** tab, you can then [send the document by email](/opencti/use/share/dissemination-lists/).

:::xtm-one[Write a report with AI]
On reports, incident response cases and requests for information, the format step also offers **Ask AI (multiple formats supported)** when AI is available on your platform. It writes a text about the container in the tone, language and number of paragraphs you choose, as HTML, Markdown, plain text or JSON, and saves it as a new file or in the main content of the container. See [XTM One in OpenCTI](/xtm-one/use/in-opencti/).
:::

## Check that it works

The new file shows in **Exported files** or in the **Exports list**, with a progress indicator while the connector works. If it never appears, check in the list of connectors that the export connector runs, and read the error of its last work. See [Import with connectors](/opencti/use/ingest/connectors/) for where connectors are listed.

## What's next

- [Send intelligence by email](/opencti/use/share/dissemination-lists/), to email a document to recipients outside the platform.
- [Share knowledge with your security tools](/opencti/use/share/feeds/), to share data continuously.
- [Brand your PDF reports](/opencti/configure/fintel-designs/), for the look of PDF documents.
