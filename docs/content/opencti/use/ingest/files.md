---
title: Import from files
description: Upload a STIX bundle, a MISP event, a document or a CSV file to OpenCTI, choose the connector that reads it, and review the result in a draft before it reaches the knowledge base.
sidebar_position: 10
doc_type: how-to
products:
  - opencti
personas: [cti-analyst]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/import-files/
---

A vendor publishes a PDF report on Velvet Harbor, a partner CERT emails you a STIX bundle, the SOC hands you a list of addresses: you want what they say in your knowledge base, linked to what you already know, without retyping it. This page shows how to turn such a file into knowledge: you upload it, an import connector reads it, and you review what it found in a draft or a workbench before anything reaches the knowledge base. For the other ways data gets in, see [How intelligence gets into OpenCTI](/opencti/use/ingest/).

## Before you start

- Your role has the **Upload knowledge files** capability, to upload a file, and the **Import knowledge** capability, to have a connector read it. A role that grants **Import knowledge** only inside drafts can still import, but always into a draft.
- An import connector for your file type runs on the platform. Your administrator deploys it from the [connector catalog](/opencti/use/ingest/connectors/).
- For a CSV file, an administrator has created a [CSV mapper](/opencti/configure/mappers/csv/) that matches its columns.

## Which connector reads which file

| File | Connector | What it creates |
| --- | --- | --- |
| STIX 2.1 bundle (JSON or XML) | `ImportFileStix` | Exactly the objects of the bundle |
| MISP event (JSON) | `ImportFileMISP` | The event, its attributes and its objects, as STIX objects |
| PDF, text, HTML or Markdown document | `ImportDocument` | The entities it recognizes from your knowledge base, and the observables it finds, such as IP addresses and domains |
| YARA rules | `ImportFileYARA` | One Indicator per rule |
| CSV file | **[FILE] CSV Mapper import**, built into the platform | What the CSV mapper you choose describes |

A file the browser cannot type is recognized by its extension: `.yar` and `.yara` files are read as YARA rules. Your administrator can map more extensions in the [technical customization settings](/reference/opencti/configuration/#technical-customization).

`ImportDocument` does not read CSV files: use a CSV mapper for them.

The built-in CSV import is on by default. If **[FILE] CSV Mapper import** is not offered, or if your CSV imports always wait for review, your administrator set it that way in the platform configuration:

| Setting | Effect | Default |
| --- | --- | --- |
| `import_csv_built_in_connector:enabled` (`IMPORT_CSV_BUILT_IN_CONNECTOR__ENABLED`) | Turns the built-in CSV import on or off. When it is off, **[FILE] CSV Mapper import** is inactive and reads no file | `true` |
| `import_csv_built_in_connector:validate_before_import` (`IMPORT_CSV_BUILT_IN_CONNECTOR__VALIDATE_BEFORE_IMPORT`) | Makes every CSV import wait for review. A direct import then lands in a draft, or in a workbench when `app:validation_mode` is `workbench` | `false` |
| `import_csv_built_in_connector:bulk_creation_size` (`IMPORT_CSV_BUILT_IN_CONNECTOR__BULK_CREATION_SIZE`) | Number of lines read and converted at a time. A lower number uses less memory on large files. An error in a batch is reported once, for its first line and the lines that follow | `5000` |

The other settings are in the [engines, schedules and managers settings](/reference/opencti/configuration/#engines-schedules-and-managers).

:::warning
A connector that reads documents can guess wrong: a word taken for a malware name, an entity of the wrong type, a value marked as unknown. Review its result before you approve it.
:::

## Where you can upload a file

| From | The file is linked to |
| --- | --- |
| **Import data** in the top bar | Nothing, unless you choose an **Associated entity** |
| :menu[Data > Import], tab **Global files**, button **Import data** | Nothing, unless you choose an **Associated entity** |
| The **Data** tab of an entity, **Import data** next to **Uploaded files** | That entity |

The associated entity changes what the import creates:

- **A container**, such as a report, a grouping or a case: every object found in the file is added to the container when you approve the draft.
- **Any other entity**: only observables are linked to it, with a `related-to` relationship.

To keep a file as an attachment of a report or a case without reading it, upload it in the **Content** tab of the container instead.

## Import a file step by step

The step-by-step import lets you choose the connector and puts the result in a draft for review.

::diagram{id=opencti-ingest-files-flow}

:::steps

1. Click **Import data** in the top bar. The **Import data** window opens on **Import mode**.

   ::screenshot{id=opencti-ingest-files-import-modes alt="The Import data window offers three import modes: Direct/Automatic Import, Step-by-Step Import and Import using a Form."}

2. Choose **Step-by-Step Import**, then click **Next**.
3. Drop your files in the window or click **Browse files**. To import text you copied, such as the body of an email, click **Copy/paste mode**, paste the text and click **Create file**.
4. For each file, choose the connectors in **Connectors**. Only the connectors that accept the file type are listed. For a CSV file, choose **[FILE] CSV Mapper import**, then the CSV mapper in **Configuration**. Click **Next**.
5. In **Import options**, set the **File marking definition levels**, which restrict who can see the file itself, and an **Associated entity** if the data belongs to one.
6. Keep **Draft** as the **Validation mode**, and give the draft a name. Choose **Workbench** only to use an [analyst workbench](/opencti/use/ingest/workbench/); it is offered when you import a single file.
7. Click **Import**. When the upload ends, click **Navigate to draft** to open the draft the connector fills.

:::

With **Workbench**, the window offers **Navigate to entity** when you chose an **Associated entity**, or **Navigate to import**, which opens **Global files**.

The connector works in the background: the draft fills in over a few seconds or minutes. Review it, then click **Approve draft** to write its content to the knowledge base. See [Prepare changes in a draft](/opencti/use/ingest/drafts/).

:::example[Add a vendor advisory to the Velvet Harbor report]
Harbor Security published a PDF advisory, `harbor-security-velvet-harbor-advisory.pdf`, with new details on the campaign your team follows in the report "Velvet Harbor targets hotel booking systems". You want what it names in that report.

1. Click **Import data**, choose **Step-by-Step Import**, then **Next**.
2. Drop `harbor-security-velvet-harbor-advisory.pdf`, choose `ImportDocument` in **Connectors**, then **Next**.
3. **File marking definition levels**: `TLP:CLEAR`, as the advisory is public.
4. **Associated entity**: `Velvet Harbor targets hotel booking systems`, so everything found joins the report.
5. **Validation mode**: **Draft**, **Draft name**: `Harbor Security advisory on Velvet Harbor`.
6. Click **Import**, then **Navigate to draft**.

The draft lists what the connector recognized: the intrusion set Velvet Harbor and the malware Velvet Loader, already in your knowledge base, the domain `booking-velvetharbor.example` and the IPv4 address `203.0.113.10`. It also took `harbor-security.example`, the address of the vendor's website in the page footer, for a domain of the attacker: delete it from the draft, then approve. The report now contains the domain and the address.
:::

## Import a file directly

**Direct/Automatic Import** sends the file to every connector set to run automatically on its type, with nothing to choose. Use it for files in a standard format, such as STIX 2.1 or MISP. Each connector's own configuration decides whether its result waits in a draft or goes straight to the knowledge base. When imports must go through a draft, for example because your role only imports into drafts, this mode is not offered. Where the platform's own automatic imports wait for review is a platform setting, `app:validation_mode` (`APP__VALIDATION_MODE`): `draft` by default, or `workbench`.

The third mode, **Import using a Form**, does not read a file: it opens one of your [forms](/opencti/use/ingest/forms/). It is offered from the top bar and from **Global files**, not from an entity.

:::only{opencti=">=7.261109"}
**Import using a Form** shows only to the people who can fill in forms: with **Create / Update knowledge** or **Create knowledge via Form Intake**, outside or inside drafts, or with **Import knowledge** inside drafts. **Import knowledge** alone, outside drafts, no longer offers it. Someone who can fill in forms but not import files goes straight to the forms when they open **Import data**, with no mode to choose.
:::

## Run an import again

Every uploaded file stays in the list where you uploaded it: **Global files**, or **Uploaded files** in the entity's **Data** tab.

In **Global files**, click **Show the imports**, the icon at the end of the file's line. **File imports** opens and lists the most recent runs of connectors on this file:

- each run shows its status, or **Error** when it failed: hover the status to read the message;
- **Launch an import**, at the top, runs another connector or another CSV mapper on the file, for example after a failed attempt;
- **Navigate to draft** opens the draft a run filled;
- **Delete this work** removes a run from the list. What it imported stays.

The actions menu (⋮) of the file offers **Download** and **Delete**. In an entity's **Data** tab, the icons of a file launch an import of it, download it or delete it, and its runs are listed under the file.

::::only{opencti=">=7.260811"}
:::xtm-one[Read documents with an AI agent]
When XTM One is connected to your platform, an import connector can hand each file to an XTM One AI agent, which extracts its entities, observables and relationships. Such a connector asks you to **Select agent** in the step-by-step import; in direct mode it uses the first agent XTM One offers for the task. If no agent can do the task, the connector shows **No agent available**. The result lands in a draft, so a person reviews what the agent found. See [XTM One in OpenCTI](/xtm-one/use/in-opencti/).
:::
::::

## Check that it works

- The draft lists what the connector created, by type, in its tabs. An empty draft after a few minutes means the connector found nothing or failed: open **Show the imports** on the file to read the error.
- After you approve the draft, the objects appear in the knowledge base and, with an associated container, in its **Entities** and **Observables** tabs.

## What's next

- [Prepare changes in a draft](/opencti/use/ingest/drafts/), to review and approve the result.
- [Map CSV files to knowledge](/opencti/configure/mappers/csv/), to import your own CSV formats.
- [Import with connectors](/opencti/use/ingest/connectors/), to deploy the import connectors.
- [Reliability and confidence](/opencti/concepts/reliability-confidence/), for the confidence level imported data receives.
