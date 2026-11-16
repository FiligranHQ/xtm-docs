---
title: Search inside files
description: Index the text of the files stored in OpenCTI so that users can search inside them, and choose which files are indexed, up to what size.
sidebar_position: 50
doc_type: how-to
products:
  - opencti
edition: ee
personas: [admin, platform-engineer]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/administration/file-indexing/
---

File indexing extracts the text of the files attached to entities, or imported into the platform, and stores it in the search engine. Once it runs, a global search also looks inside PDF reports, spreadsheets and documents, and shows the files that match. This page covers the requirements, the configuration and the controls of the indexing engine.

## Before you start

- Your platform has an Enterprise Edition license.
- Your role has the **Access to file indexing** capability. Starting, pausing or resetting the engine also needs the role option **Allow modification of sensitive configuration**, because file indexing is a protected setting by default, unless the platform turns that protection off: see [sensitive configuration](/opencti/administer/sensitive-configuration/).
- The search engine can extract text from attachments. You need one of:
  - Elasticsearch 8.4 or later;
  - Elasticsearch before 8.4 with the `ingest-attachment` plugin;
  - OpenSearch with the `ingest-attachment` plugin.

  When none is met, :menu[Settings > File indexing] shows a warning instead of the configuration.

## Choose what to index

Indexing takes storage in the search engine. Before you start it, open :menu[Settings > File indexing] and set the **Configuration** card. Each change is saved at once.

| Setting | Default | What it does |
| --- | --- | --- |
| **File types to index** | All | Tick the types to index: PDF, JSON, plain text, Markdown, CSV, HTML, Excel (`.xls`, `.xlsx`), Word (`.docx`) and OpenDocument text (`.odt`) |
| **Max file size (in MB)** | 5 | Files larger than this are skipped |
| **Restrict to specific entity types** | Empty: all types | Index only the files attached to entities of these types |
| **Include files not related to any knowledge (data import)** | Off | Also index the files uploaded in :menu[Data > Import] that are not attached to an entity |

The **Information** card helps you size the choice before you start: **Total files in S3** and **Files volumes in S3** give what your configuration would index, and **S3 volume by file type** splits it, with the **Files count** and **Files size** of each type. Expect the search engine to grow by roughly the size of the text extracted from these files: unticking a heavy type that nobody searches, such as large CSV exports, is the most effective saving.

::screenshot{id=opencti-file-indexing-settings alt="The File indexing screen: the Status, Indexed files and Volume indexed cards at the top, the Control card with Start and Reset, the Information card with S3 volumes, and the Configuration card with the file types ticked and a maximum size of 5 MB."}

## Start indexing

:::steps

1. In the **Control** card, click **Start**. **Status** changes to **Running**.
2. Follow the progress in **Indexed files**, the number of files indexed out of the files that match your configuration, and in **Volume indexed**.

:::

The first run goes through every existing file. After that, the engine checks for new files every minute and indexes them. When the markings or the organizations an entity is shared with change, its indexed files follow, so users only find the files they are allowed to see.

:::example[Search inside the PDF reports of members]
The hotel members send Example CERT their incident write-ups as PDF and Word files, attached to reports. Analysts want a search for `HarborGate` to find the backdoor in those files too, without indexing the large CSV exports of :menu[Data > Import].

The platform runs Elasticsearch 8.19, which extracts attachments without a plugin, and keeps the indexing manager on, as by default. With Docker Compose:

```yaml
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.19.18
  opencti:
    environment:
      FILE_INDEX_MANAGER__ENABLED: "true"
      FILE_INDEX_MANAGER__INTERVAL: 60000
```

In the **Configuration** card of :menu[Settings > File indexing]:

- **File types to index**: only **PDF** and **DOCX**
- **Max file size (in MB)**: `10`, because some write-ups carry screenshots
- **Restrict to specific entity types**: **Report** and **Incident**
- **Include files not related to any knowledge (data import)**: off

Then click **Start**. Once **Indexed files** reaches the total, a search for `HarborGate` lists the report "HarborGate backdoor on booking servers" and the PDF attached to it where the word appears.
:::

## Pause or reset

- **Pause** stops the engine; **Start** resumes it where it stopped. In the **Control** card, **Indexing manager start** is when the engine last started a pass over new files, every minute by default, and **Last indexation** is when that pass ended. If these dates stop moving while **Status** reads **Running**, check that `file_index_manager:enabled` is on and look for `File index manager` errors in the platform logs.
- **Reset** stops the engine and deletes every indexed file from the search engine. Files in storage are not touched. Click **Start** again to rebuild the index from scratch.

If you change the configuration after the first run, files already indexed stay indexed, and files that were excluded are not caught up. Reset, then start, to apply the new configuration to every file.

## Check that it works

Upload a PDF to an entity, wait a minute, and search for a word that only appears in that file. The result list shows the file and the entity it belongs to. See [Search](/opencti/use/explore/search/).

## What's next

- [Search](/opencti/use/explore/search/) to search knowledge and files.
- [Configuration reference](/reference/opencti/configuration/) for `file_index_manager:enabled` and `file_index_manager:interval`, which turn the engine on and set how often it looks for new files.
