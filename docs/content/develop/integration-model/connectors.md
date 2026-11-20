---
title: Connectors
description: The six connector types of OpenCTI, what each receives and returns, which user and confidence level its data gets, and how connectors are deployed and scaled.
doc_type: explanation
products:
  - opencti
personas: [developer, platform-engineer]
owner: docs-develop
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/deployment/connectors/
---

A connector is a process that runs next to OpenCTI and exchanges STIX 2.1 data with it. OpenCTI knows six connector types: each one is started differently, receives different input, and writes as a different user. This page explains the types as the connector helper of pycti (`OpenCTIConnectorHelper`) and the platform implement them in OpenCTI 7.261030, so you can choose the type of the connector you build and know what its code will receive. To run an existing connector, read [How integrations run](/deploy/integrations/how-they-run/) instead.

::diagram{id=develop-integration-model-connectors}

## The six types

`CONNECTOR_TYPE` sets the type. The connector's own configuration fixes it: a deployment never changes it.

| Type | Started by | Receives | Returns | `CONNECTOR_SCOPE` | Example |
| --- | --- | --- | --- | --- | --- |
| `EXTERNAL_IMPORT` | Its own schedule | Nothing: it fetches from its source | STIX bundles | The types it imports, informative | MITRE ATT&CK, MISP, a vendor feed |
| `INTERNAL_ENRICHMENT` | A user, a playbook, or the platform on each new entity in scope | One entity and the objects around it | A STIX bundle that adds to the entity | Entity types, such as `IPv4-Addr,Domain-Name` | IP reputation, passive DNS |
| `INTERNAL_IMPORT_FILE` | A file uploaded to the platform | The file and its context | A STIX bundle, written directly or kept in a workbench or a draft | MIME types, such as `application/pdf` | Import document, import STIX file |
| `INTERNAL_ANALYSIS` | A user who asks for an analysis of a container | The fields or the file to analyze | An analysis file attached to the entity | MIME types | Import document |
| `INTERNAL_EXPORT_FILE` | A user who asks for an export | The export request: format, scope, filters | A file in the platform storage | MIME types it produces, such as `text/csv` | Export STIX, CSV, TXT |
| `STREAM` | Its own start, then each event | The events of a live stream | Whatever it does in the target system; it can also send bundles back | Informative | A SIEM or EDR synchronization |

The first and the last types run on their own; the others wait for the platform. In the code, the difference is one call:

- A self-triggered connector calls `helper.schedule_iso(message_callback, duration_period)`: the helper runs the callback now, then every `CONNECTOR_DURATION_PERIOD`. A stream connector calls `helper.listen_stream(message_callback)`.
- A platform-triggered connector calls `helper.listen(message_callback)`: the helper calls `message_callback(data)` for each job and reports the string it returns as the result of the job.

## What the callback receives

The platform posts a job as a message with two parts: `internal`, which the helper consumes (the work to report on, the user who asked, the draft), and `event`, which becomes the `data` dictionary of your callback. The keys below come from the platform code of 7.261030.

::::tabs
:::tab[Enrichment]
```python
{
    "event_type": "INTERNAL_ENRICHMENT",
    "entity_id": "indicator--5bc47565-1eed-5022-9d7d-2892cceae172",  # standard id
    "entity_type": "Indicator",
    "stix_entity": {...},        # the entity, in STIX 2.1
    "stix_objects": [...],       # the entity and the objects around it, in STIX 2.1
    "enrichment_entity": {...},  # the entity as the OpenCTI API returns it, with its files
}
```

`CONNECTOR_ENRICHMENT_RESOLUTION` decides who builds `stix_objects`: with `none`, the default of pycti, the platform sends the entity alone and the helper reads the bundle through the API; with `stix_bundle`, the platform sends the bundle with the job. The callback receives both keys either way.

When a playbook runs the connector, `stix_objects` holds the bundle of the playbook, and the bundle you send goes back to the playbook instead of the workers. Pass `playbook_compatible=True` to `OpenCTIConnectorHelper` to offer the connector in playbooks.
:::
:::tab[File import]
```python
{
    "file_id": "import/Report/f1e2.../velvet-harbor-iocs.pdf",
    "file_mime": "application/pdf",
    "file_markings": [...],      # marking ids of the file
    "file_fetch": "/storage/get/import/Report/f1e2.../velvet-harbor-iocs.pdf",
    "entity_id": "...",          # the entity the file was uploaded on, if any
    "validation_mode": "workbench",  # or "draft"
    "bypass_validation": False,  # True: write directly, without a workbench or a draft
    "force_validation": False,
}
```

Read the file with `helper.api.fetch_opencti_file(helper.opencti_url + data["file_fetch"])`.
:::
:::tab[Analysis]
```python
{
    "event_type": "INTERNAL_ANALYSIS",
    "entity_id": "report--...",
    "entity_type": "Report",
    "content_type": "FIELDS",    # or "FILE"
    "content_source": "content_mapping",
    "content_fields": "description content",  # with FIELDS
    "file_id": "...", "file_mime": "...", "file_fetch": "...",  # with FILE
    "analysis_name": "FIELDS_analysis_content_mapping.analysis",
}
```
:::
:::tab[Export]
```python
{
    "event_type": "INTERNAL_EXPORT_FILE",
    "export_scope": "single",    # "single", "query" or "selection"
    "export_type": "simple",     # "simple" or "full"
    "format": "text/csv",
    "file_name": "2026-01-15T10:00:00.000Z_TLP:GREEN_(Export CSV)_Report-....csv",
    "entity_id": "...", "entity_name": "...", "entity_type": "Report",
    "file_markings": [...],
    "main_filter": {...}, "access_filter": {...},
    "list_params": {...},        # with "query" and "selection"
    "selected_ids": [...],       # with "selection"
}
```
:::
:::tab[Stream]
The callback receives each server-sent event as a message object: `msg.event` is the event type (`create`, `update`, `delete`, `merge`), `msg.data` a JSON string with the STIX object in `data`, and `msg.id` the position in the stream. After each event, the helper saves `msg.id` in the connector's state, so a restart resumes where it stopped.
:::
::::

## As which user, with which confidence

A connector registers with the token in `OPENCTI_TOKEN`, and the platform records that token's user as the connector's user. Everything else follows from it:

| Type | Writes or reads as |
| --- | --- |
| `EXTERNAL_IMPORT`, `STREAM` | The connector's user |
| `INTERNAL_ENRICHMENT`, `INTERNAL_ANALYSIS` | The connector's user: the platform sends the job without the user who asked |
| `INTERNAL_IMPORT_FILE` | The user who uploaded the file: the helper sends the bundle on their behalf |
| `INTERNAL_EXPORT_FILE` | The user who asked for the export: the helper reads through `helper.api_impersonate`, with that user's markings, so an export never leaks what they cannot see |

OpenCTI honors a request on behalf of another user only from a user that bypasses capabilities, and silently ignores it otherwise. An export connector whose user does not bypass capabilities therefore reads with its own rights, and its exports can hold what the requester may not see. Run export connectors with a user of a group whose role bypasses capabilities, such as an Administrator role. The other types run with a user of the `Connectors` group: for file imports, the workers write on behalf of the uploader themselves.

The worker writes each object with the confidence level of the user it writes as:

- A new object gets the confidence of the bundle, capped at the user's maximum confidence level. An object without `confidence` gets the user's maximum.
- An existing object takes the new values only when the new confidence is at least the one it has. With a lower confidence, the worker still fills the fields that are empty and adds the labels and external references, but changes nothing else.

So the confidence of the connector's user is the ceiling of everything it imports. With `CONNECTOR_AUTO_CREATE_SERVICE_ACCOUNT=true`, the connector creates its own user at start: `[C] <CONNECTOR_NAME>`, in the `Connectors` group, with the maximum confidence level `CONNECTOR_AUTO_CREATE_SERVICE_ACCOUNT_CONFIDENCE_LEVEL` (50 by default). XTM Composer does the same from the catalog: its form proposes the `max_confidence_level` of the [manifest](/develop/build/manifest/).

## The settings that depend on the type

pycti reads these settings for every connector; each one matters to some types only. Their types and defaults are in the [connector settings reference](/reference/opencti/integration-settings/#connectors-connector-settings).

| Setting | Types | What it changes |
| --- | --- | --- |
| `CONNECTOR_AUTO` | Enrichment, file import | `true`: the platform sends every new entity or file in scope, not only what a user asks for |
| `CONNECTOR_AUTO_UPDATE` | Enrichment | `true`: also the entities in scope that are updated |
| `CONNECTOR_ENRICHMENT_RESOLUTION` | Enrichment | `none` or `stix_bundle`: see the **Enrichment** tab above |
| `CONNECTOR_ONLY_CONTEXTUAL` | File import | `true`: only files uploaded on an entity, never files imported on their own |
| `CONNECTOR_VALIDATE_BEFORE_IMPORT` | File import, external import | `true`: what the connector sends waits in a workbench or a draft for a user to validate it |
| `CONNECTOR_DURATION_PERIOD` | External import | Time between two runs, as an ISO 8601 duration (`PT1H`); `0` runs once |
| `CONNECTOR_RUN_AND_TERMINATE` | External import | `true`: run once and exit, for a connector started by cron or a Kubernetes job |
| `CONNECTOR_QUEUE_THRESHOLD` | External import | Size of the connector's queue, in megabytes, above which a scheduled run is skipped until the workers catch up. 500 by default |
| `CONNECTOR_LIVE_STREAM_ID` | Stream | The live stream to follow, from :menu[Data > Data sharing > Live streams]; unset, the raw stream of all events |
| `CONNECTOR_LIVE_STREAM_LISTEN_DELETE` | Stream | `false`: skip deletion events |
| `CONNECTOR_LIVE_STREAM_NO_DEPENDENCIES` | Stream | `false`: also send the objects each event depends on |
| `CONNECTOR_LIVE_STREAM_WITH_INFERENCES` | Stream | `true`: also send what the inference rules created |
| `CONNECTOR_LIVE_STREAM_START_TIMESTAMP` | Stream | First run only: the stream position to start from, in milliseconds |
| `CONNECTOR_LIVE_STREAM_RECOVER_ISO_DATE` | Stream | First run only: replay the existing data up to this date before live events |

## How a connector reaches the platform

By default, a connector sends its bundles to RabbitMQ and receives its jobs from RabbitMQ, at the address the platform returns at registration. When the connector cannot reach RabbitMQ, use HTTP instead:

| Setting | Effect |
| --- | --- |
| `CONNECTOR_QUEUE_PROTOCOL=api` | Sends bundles through the platform API instead of RabbitMQ. A bundle cannot exceed the platform's `app:max_payload_body_size` (`APP__MAX_PAYLOAD_BODY_SIZE`), 50 MB by default. `QUEUE_PROTOCOL` is its former name, read only when `CONNECTOR_QUEUE_PROTOCOL` is unset |
| `CONNECTOR_LISTEN_PROTOCOL=API` | Starts an HTTP server in the connector, and the workers post the jobs to it at `CONNECTOR_LISTEN_PROTOCOL_API_URI` followed by `CONNECTOR_LISTEN_PROTOCOL_API_PATH`. For enrichment, file import, analysis and export connectors |

:::only{opencti=">=7.261119"}
When the platform turns on the secure HTTP defaults (`app:http:secure_by_default` set to `true`) and leaves `app:max_payload_body_size` unset, the limit drops to 10 MB. A connector that sends large bundles over the API then has its larger bundles refused: send smaller bundles, or ask the platform engineer to set `APP__MAX_PAYLOAD_BODY_SIZE` explicitly.
:::

With both, an enrichment connector never connects to RabbitMQ. The `MQ_*` settings override the RabbitMQ address the platform returns; [Run an integration without XTM Composer](/deploy/integrations/manual/) shows them in a deployment.

Three more settings shape the connection to the platform:

- `OPENCTI_SSL_VERIFY=true` checks the platform's TLS certificate, off by default. Turn it on whenever `OPENCTI_URL` is HTTPS.
- `OPENCTI_CUSTOM_HEADERS` adds HTTP headers to every API call, as `name:value` pairs separated by `;`, for a gateway in front of the platform.
- `CONNECTOR_EXPOSE_METRICS=true` exposes Prometheus metrics of the connector on `CONNECTOR_METRICS_PORT` (9095 by default): bundles sent, runs, errors.

## Keep a copy of what the connector sends

A connector can also write each bundle elsewhere, for an audit or to move data by other means:

- `CONNECTOR_SEND_TO_DIRECTORY=true` writes each bundle as a JSON file in `CONNECTOR_SEND_TO_DIRECTORY_PATH`, and deletes the files after `CONNECTOR_SEND_TO_DIRECTORY_RETENTION` days (7; `0` keeps them).
- `CONNECTOR_SEND_TO_S3=true` uploads each bundle to S3, by default to the bucket of the platform, under `CONNECTOR_SEND_TO_S3_FOLDER` (`connectors`), and deletes them after `CONNECTOR_SEND_TO_S3_RETENTION` days (7). `CONNECTOR_SEND_TO_S3_BUCKET` and the `S3_*` settings, such as `S3_ENDPOINT`, `S3_PORT` and `S3_BUCKET_REGION`, replace what the platform returns at registration.
- `CONNECTOR_SEND_TO_QUEUE=false` stops sending bundles to the platform: only the copies remain.

## Deploy and scale

| Type | Instances per `CONNECTOR_ID` | To go faster |
| --- | --- | --- |
| `EXTERNAL_IMPORT` | One. Two copies would run twice and overwrite each other's state | Add workers: the import speed is the speed at which workers empty the queue |
| `INTERNAL_ENRICHMENT`, `INTERNAL_IMPORT_FILE`, `INTERNAL_ANALYSIS`, `INTERNAL_EXPORT_FILE` | One or more. Each process handles one job at a time; copies with the same ID share the `listen_` queue | Run more replicas of the same connector |
| `STREAM` | One. The state holds its position in the stream | One connector per target system or per live stream |

A process takes a job off the queue before it handles it: a job that a crash interrupts is not delivered again, and its work stays in progress. The [Scale](/deploy/operate/scale/) page sizes the workers.

Each connector has two RabbitMQ queues named after its ID, `push_<id>` for what it sends and `listen_<id>` for its jobs, created at its first registration. The **Deployed** tab of :menu[Integrations] shows the messages waiting in them. Removing a connector from the interface deletes its queues and its works.

## What's next

- [Build an OpenCTI connector](/develop/build/connector/), an external import connector step by step.
- [Integration manifest](/develop/build/manifest/), to make it deployable from the catalog.
- [OpenCTI integration settings](/reference/opencti/integration-settings/), every setting with its default.
- [Import with connectors](/opencti/use/ingest/connectors/), what a connector's screens show.
