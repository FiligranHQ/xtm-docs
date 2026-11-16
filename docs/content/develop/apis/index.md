---
title: APIs and integration options
sidebar_label: APIs
description: The ways code reaches OpenCTI, from the GraphQL API to live streams, feeds, connectors and MCP, what each one is for, how they authenticate, and which to choose.
doc_type: explanation
products:
  - opencti
  - openaev
personas: [developer, soc-manager, platform-engineer]
owner: docs-platform
last_reviewed: 2026-11-01
sidebar_position: 0
migrated_from:
  - opencti:/deployment/integrations/
---

OpenCTI offers several ways in for code, and they do not do the same job. The GraphQL API reads and writes anything, at the pace of your script. Live streams push every change to a consumer as it happens. CSV feeds and TAXII collections hand a filtered list to tools that poll. Connectors are programs that run beside the platform. This page explains what each way is for, what they share, and how to pick one for an integration, such as sending Velvet Harbor indicators to a SIEM or importing a hotel-sector feed.

::diagram{id=develop-apis}

## Choose a way in

| You want to | Use | Endpoint | Page |
| --- | --- | --- | --- |
| Query or change knowledge from a script, an app or a notebook | GraphQL API | `/graphql` | [Query OpenCTI with GraphQL](/develop/apis/graphql/) |
| Do the same from Python, with helpers per entity type | Python client (`pycti`), on top of the GraphQL API | `/graphql` | [Python client](/develop/build/python-client/) |
| Keep a SIEM, an EDR, a SOAR or another platform in step with the knowledge base, deletions included | Live stream | `/stream/{id}` | [Data streaming](/develop/apis/streams/) |
| Give a device a list it downloads on a schedule | CSV feed | `/feeds/{id}` | [Share knowledge with your security tools](/opencti/use/share/feeds/#create-a-csv-feed) |
| Serve a tool that speaks Trusted Automated Exchange of Intelligence Information (TAXII) 2.1 | TAXII collection | `/taxii2/root/collections/{id}/objects/` | [Share knowledge with your security tools](/opencti/use/share/feeds/#create-a-taxii-collection) |
| Let a partner push STIX objects into OpenCTI | TAXII push | `/taxii2/root/collections/{id}/objects/` (`POST`) | [Let partners push intelligence to you](/opencti/use/ingest/taxii-push/) |
| Import, enrich or export data continuously, as a component of the platform | Connector | Queues and `/graphql` | [Connectors](/develop/integration-model/connectors/) |
| Let an AI assistant or an AI agent query OpenCTI | Model Context Protocol (MCP) server of XTM One | `/mcp/opencti` on XTM One | [MCP servers](/develop/apis/mcp/) |
| Script OpenAEV scenarios, simulations and results | OpenAEV REST API | `/api` | [REST API](/develop/apis/rest/) |

When a ready-made integration exists, start there: the [connector catalog](/opencti/use/ingest/connectors/) lists the connectors of the XTM Hub, among them stream connectors for the main SIEMs and EDRs.

## Pull, listen or push

The ways in differ in who drives the exchange, and so in what the other side has to do.

- **You pull, with the GraphQL API.** Your code asks, the platform answers, page by page. Nothing tells your code that something changed: to follow changes, it polls with a date filter, or it listens to a stream instead. Use it for reads on demand, writes, and anything a stream or a feed does not cover.
- **You listen, with a live stream.** The platform sends each creation, update, deletion and merge within seconds, with the objects each one depends on. Your consumer keeps its position and resumes after a cut. Use it when a copy of the knowledge must stay in step, removals included, such as indicators an EDR blocks.
- **The tool pulls, with a feed.** A CSV feed or a TAXII collection is computed at each request from its filters. The tool learns nothing about removals beyond what disappears from the list. Use it for devices that only download.
- **A partner pushes, with TAXII push.** OpenCTI receives STIX bundles from a client and imports them.

A connector can combine these: a stream connector listens to a live stream and writes to a SIEM; an import connector reads a feed and sends STIX bundles to OpenCTI through its queues.

## What every way shares

**One identity per integration.** Every request runs as a user: its markings, its organizations and its confidence level decide what it reads and what it may write, and its name shows in the history of what it changes. Give each integration its own user: see [Create accounts for integrations](/opencti/administer/service-accounts/) explains, rather than a person's account.

**The same credentials.** The GraphQL API, live streams, CSV feeds and TAXII collections accept:

| Credential | Header | Capability the user needs |
| --- | --- | --- |
| API token | `Authorization: Bearer` followed by the token | **Allow token usage** |
| Login and password | `Authorization: Basic` followed by `login:password` in Base64 | **Allow basic auth usage** |

Both capabilities sit under **API usage with authorization header** in the role editor. A user creates tokens in the **API access** card of their profile, with a duration from 30 days to unlimited: see [Get an API token](/develop/apis/graphql/#get-an-api-token). On top of that, feeds and live streams need **Access data sharing**, and a feed or a live stream made public needs no credential at all.

**The same filters.** List queries, feeds, live streams, triggers and playbooks select objects with the same filter group format. See [Filters](/develop/apis/filters/).

**The same data model.** What goes out is STIX 2.1, with the OpenCTI properties in an extension; what the API returns follows the same types. The [data model reference](/reference/opencti/data-model/) lists the entity types and their attributes.

::::only{opencti=">=7.261030"}
:::xtm-one[XTM One and the APIs]
XTM One exposes OpenCTI to any MCP client, such as Cursor or an AI agent your team builds: the client searches, reads and creates knowledge with the rights of its user, with nothing to code against the GraphQL API. The **XTM One MCP server** card of your OpenCTI profile gives the endpoint URL. See [XTM One in OpenCTI](/xtm-one/use/in-opencti/).
:::
::::

## Limits to plan for

| Limit | Default | Where it is set |
| --- | --- | --- |
| Items in one page of a GraphQL list | 500 when `first` is left out, 5,000 at most | `elasticsearch:default_pagination_result`, `elasticsearch:max_pagination_result` |
| Requests per client, by address and user agent | 10,000 a second, then HTTP `429` | `app:rate_protection:max_requests`, `app:rate_protection:time_window` |
| Events a live stream consumer can resume from | The last 2,000,000 events of the platform | `redis:trimming` |
| Objects in one answer of a CSV feed | 5,000 | `data_sharing:max_csv_feed_result` |
| Objects in one page of a TAXII collection | 500 | `app:data_sharing:taxii:max_pagination_result` |

A platform engineer changes them in the configuration: see the [configuration reference](/reference/opencti/configuration/).

## What's next

- [Query OpenCTI with GraphQL](/develop/apis/graphql/): a token, a first request, queries, mutations, pagination and errors.
- [Data streaming](/develop/apis/streams/): the events of live streams, and how to write a consumer.
- [Filters](/develop/apis/filters/): the filter group format.
- [HTTP endpoints](/reference/opencti/http-endpoints/): every endpoint of the platform and how it authenticates.
