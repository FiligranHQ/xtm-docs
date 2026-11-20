---
title: "How OpenCTI works: platform, workers, managers and connectors"
description: What OpenCTI is made of, which parts keep state and which scale, and the path a piece of data takes from a connector to the knowledge base and the live stream.
doc_type: explanation
products:
  - opencti
personas: [platform-engineer, evaluator, admin]
owner: docs-platform
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/deployment/overview/
  - opencti:/deployment/advanced/managers/
---

OpenCTI is a stateless platform and its workers, in front of four services that keep the data: Elasticsearch or OpenSearch, RabbitMQ, Redis and S3 storage. The platform serves the interface and the GraphQL API, and runs the managers, its background jobs. Workers write queued data through that API, and connectors bring data in and send it out. This page explains what each part does, which parts you can multiply, and the path a piece of data takes. With it, you can plan a deployment, read its logs and find where a delay comes from. How OpenCTI fits in the XTM stack, next to XTM One and OpenAEV, is in [The XTM stack](/deploy/plan/architecture/).

## What OpenCTI is made of

::diagram{id=opencti-architecture-components}

| Part | What it is | Keeps data | Configured in |
| --- | --- | --- | --- |
| Platform | A Node.js application: the web interface, the GraphQL API and the managers | No | [Basic parameters](/reference/opencti/configuration/#basic-parameters) |
| Workers | Python processes that take messages from RabbitMQ and write them through the platform API | No | Their own environment |
| Connectors | Python processes, one per integration, that import, enrich, analyze, export or stream data | No | Their own environment |
| Elasticsearch or OpenSearch | The knowledge base and its search indices, prefixed `opencti` by default | Yes | [Elasticsearch](/reference/opencti/configuration/#elasticsearch) |
| RabbitMQ | The queues between the platform, its workers and its connectors | Messages in transit | [RabbitMQ](/reference/opencti/configuration/#rabbitmq) |
| Redis | The live streams, the locks between platforms, the sessions and the progress of connector jobs | Yes, the streams | [Redis](/reference/opencti/configuration/#redis) |
| S3 storage (MinIO, or RustFS in xtm-docker) | Files: uploads, imports, exports and support packages, in the bucket `opencti-bucket` by default | Yes | [S3 Bucket](/reference/opencti/configuration/#s3-bucket) |

The platform, the workers and the connectors keep nothing on disk: everything they need survives in the four services. You [back up](/deploy/operate/backup/) those four, and you add or replace platforms and workers at will. The versions each service needs are in [Requirements](/deploy/plan/requirements/). To try the whole stack on one host, follow the [Quick start](/deploy/quick-start/).

## The platform and its GraphQL API

The platform is the center of OpenCTI. It serves the web interface and the [GraphQL API](/develop/apis/graphql/), and every other part goes through that API: the interface, workers, connectors and your own scripts. The API checks permissions, resolves duplicates and writes to Elasticsearch. Then it records each change as an event in a Redis stream. Every query, mutation and subscription of the API, with the capability it requires, is in the [GraphQL schema reference](/reference/opencti/graphql-schema/).

For production, you can run several platforms behind a load balancer, all connected to the same Elasticsearch, Redis, S3 storage and RabbitMQ. They share the work:

- Each API request is served by the platform that receives it. Sessions are kept in Redis by default, so a user can land on any platform.
- Each manager runs on one platform at a time. Every platform starts its managers, and each run first takes the manager's lock in Redis. The platform that gets the lock runs it, and the others skip that run. If that platform stops, another one takes the lock.

How to run platforms as a cluster is in [Scale](/deploy/operate/scale/#run-opencti-as-a-cluster).

## Workers

Workers are standalone Python processes. They take the messages that connectors and the platform put in RabbitMQ and write them through the platform API. Writing is asynchronous: a connector hands its data to a queue and moves on, and the workers write it as fast as the platform and Elasticsearch accept it.

To write faster, start more workers. Each worker sends its writes to a platform, so more workers need more platforms to receive them: count one platform for every three workers. Past a point, the throughput of the Elasticsearch cluster is the limit. xtm-docker starts three workers for one platform. The machines each part needs are in [Sizing](/deploy/plan/sizing/), and how to add workers and platforms is in [Scale](/deploy/operate/scale/).

## Managers

Managers are background components inside the platform. Each one does one job: apply inference rules, record history, lower the score of indicators, delete expired data, run playbooks, send notifications. Some read the live stream and react to every change. Others run on a schedule.

Each manager has an `enabled` key in the configuration, and almost all are on by default. For example, `rule_engine:enabled`, or the environment variable `RULE_ENGINE__ENABLED`, turns the rules engine off on the platform that reads it. Intervals are configuration keys too, such as `expiration_scheduler:interval`, in milliseconds. The [manager reference](/reference/opencti/managers/) lists every manager of your version with how it runs, its default and its keys. The keys are also in the [Engines, Schedules and Managers](/reference/opencti/configuration/#engines-schedules-and-managers) section of the configuration reference. To see which managers a running platform has turned on, open the [managers list](/opencti/configure/parameters/#check-dependencies-and-managers) of the platform parameters.

:::only{opencti=">=7.261117"}
### The ingestion health manager

The ingestion health manager checks, every minute, the connectors, feeds and synchronizers that are running. It looks at the user each one runs as (missing, inactive, not a service account, lacking a capability), at the heartbeat and the errors of connectors, and at the runs of feeds and synchronizers that fail or that their source refuses. Their health shows on their screens. When a health turns degraded, critical or recovered, the manager records an `ingestion_health` event in the activity log. It runs by default: `ingestion_health_manager:enabled` (`INGESTION_HEALTH_MANAGER__ENABLED`) turns it off, and `ingestion_health_manager:interval` sets its period, in milliseconds (60000 by default).
:::

:::note[Turn a manager off on every platform]
In a cluster, a manager keeps running as long as one platform has it on. To stop a manager, turn it off in the configuration of every platform.
:::

:::xtm-one[Registration with XTM One]
The XTM One registration manager runs when `xtm:xtm_one_token` (`XTM__XTM_ONE_TOKEN`) is set. It registers the platform with XTM One at `xtm:xtm_one_url` (`XTM__XTM_ONE_URL`, `https://common.xtm1.filigran.io` by default): 30 seconds after start, then every 5 minutes, so XTM One always knows the platform. How to set it up is in [Connect the modules](/deploy/configure/connect-modules/).
:::

## Connectors

Connectors are separate Python processes that connect to the platform with the token of an OpenCTI user, usually a [service account](/opencti/administer/service-accounts/). A connector registers through the GraphQL API and receives the connection details of RabbitMQ from the platform. It then sends its data to its queue in RabbitMQ, not to the API.

Everything a connector writes is written as its user. That user's groups decide which markings the connector can write, and its max confidence level decides what it can overwrite. [Data processing](/opencti/concepts/data-processing/) explains these checks.

Each connector has one of six types:

| Type | What it does | Examples |
| --- | --- | --- |
| `EXTERNAL_IMPORT` | Pulls data from a remote source, converts it to Structured Threat Information Expression (STIX) 2.1 and sends it to OpenCTI | MITRE ATT&CK, MISP, CVE |
| `INTERNAL_ENRICHMENT` | Listens for new entities or user requests, and adds data from a remote source | Shodan, DomainTools, IPinfo |
| `INTERNAL_IMPORT_FILE` | Extracts data from files uploaded through the interface or the API. See [Import from files](/opencti/use/ingest/files/) | STIX 2.1, PDF, text, HTML |
| `INTERNAL_ANALYSIS` | Reads the text of a container and proposes the entities it mentions, on its **Content** tab | Document import, in analysis mode |
| `INTERNAL_EXPORT_FILE` | Generates an export of one object or a list. See [Export knowledge to a file](/opencti/use/share/export/) | STIX 2.1, CSV, PDF |
| `STREAM` | Reads a live stream of the platform and acts on each event | Splunk, Elastic Security, QRadar |

The available connectors are in the [XTM Hub integrations library](https://hub.filigran.io/cybersecurity-solutions/open-cti-integrations). How they are deployed, by hand or by XTM Composer, is in [How integrations run](/deploy/integrations/how-they-run/). How administrators choose, deploy and follow them in the interface is in [Connectors](/opencti/use/ingest/connectors/).

## The path of a piece of data

::diagram{id=opencti-architecture-data-path}

Take Example CERT's advisory "Velvet Harbor targets hotel booking systems", brought in by an `EXTERNAL_IMPORT` connector:

1. The connector fetches the advisory from its source and converts it to a STIX 2.1 bundle: the report, the intrusion set Velvet Harbor, its malware, its indicators and the relationships between them.
2. The connector splits the bundle into one message per object and sends them to its push queue in RabbitMQ. It goes on with its work; the messages wait until a worker is free.
3. A worker takes a message and writes its object through the platform API.
4. The platform checks the connector's permissions, merges the object with what already exists, and writes it to Elasticsearch. The object now appears in searches.
5. The platform records each change as an event in the live stream, in Redis.
6. Everything that reacts to changes reads that stream: the rules engine and other managers, the synchronizers of other platforms and `STREAM` connectors. Connectors and remote platforms read it through the platform's stream API, not from Redis.

The other services play their part on the side:

- **S3 storage** holds files. A file you upload is stored there, then an `INTERNAL_IMPORT_FILE` connector reads it and sends what it finds through the same path. Exports, once generated, are stored there too.
- **Redis** keeps the live stream to a set length: 2,000,000 events by default (`redis:trimming`, `REDIS__TRIMMING`; 0 keeps every event). A consumer that falls further behind than that misses events. Redis also holds the activity and notification streams, the manager locks, the sessions and the progress of connector jobs.
- **RabbitMQ** also carries work in the other direction. The platform puts enrichment, analysis, import and export requests in the queue of the connector that handles them.

The built-in feeds (CSV, JSON, RSS and TAXII) run inside the platform, not as connectors. They enter the path at step 2: the platform puts their objects in the queue for the workers.

## Find where a delay comes from

When data arrives slower than expected, follow the path:

| What you see | Likely cause | What to do |
| --- | --- | --- |
| A connector's queue keeps growing | Too few workers, or platforms too busy to take their writes | Add workers, then platforms, as [Scale](/deploy/operate/scale/) describes |
| Workers are busy but writes are slow | The throughput of Elasticsearch | Check the search engine's own monitoring and [Sizing](/deploy/plan/sizing/) |
| A stream connector or a remote platform misses events | It fell further behind than the trimmed live stream | Raise `redis:trimming`, or find why the consumer is slow |
| A connector has not run for a while | The connector stopped, or its user lost a right | Its status in [Connectors](/opencti/use/ingest/connectors/#follow-what-connectors-import), then its logs |

:::only{opencti=">=7.261117"}
Before reading logs, look at the health the [ingestion health manager](#the-ingestion-health-manager) gives each connector, feed and synchronizer: it names what fails, such as a user that is no longer active or a missing heartbeat.
:::

The messages waiting in each connector's queue show in [Connectors](/opencti/use/ingest/connectors/#follow-what-connectors-import). The [observability metrics](/deploy/operate/metrics/) of the platform show each step, and the [metrics reference](/reference/opencti/metrics/) lists them.

## What's next

- [The XTM stack](/deploy/plan/architecture/): OpenCTI next to XTM One, OpenAEV and XTM Composer, and the services they share.
- [Sizing](/deploy/plan/sizing/): how many platforms and workers, and how large each service.
- [Scale](/deploy/operate/scale/): add workers, run platforms as a cluster, cluster the services.
- [How integrations run](/deploy/integrations/how-they-run/): deploy and manage connectors.
- [Data processing](/opencti/concepts/data-processing/): what the platform does to each object on the way in.
- [OpenCTI configuration reference](/reference/opencti/configuration/): every key, including those of the managers.
