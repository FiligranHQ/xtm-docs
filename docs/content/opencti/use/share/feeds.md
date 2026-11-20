---
title: Share knowledge with your security tools
description: Publish a filtered part of the OpenCTI knowledge base to a SIEM, an EDR, a firewall or another platform, as a live stream, a CSV feed or a TAXII collection, and keep them in sync.
sidebar_position: 1
doc_type: how-to
products:
  - opencti
personas: [soc-manager, admin, platform-engineer]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/feeds/
---

OpenCTI publishes part of its knowledge base itself, with no connector to deploy on its side: as a live stream of events, as a CSV file at a URL, or as a Trusted Automated Exchange of Intelligence Information (TAXII) 2.1 collection. This page helps you choose the right one for each detection tool, filter what it receives, size it, and check that every consumer stays in sync. You find the three in :menu[Data > Data sharing].

::diagram{id=opencti-share-feeds}

## Before you start

- To create and change feeds, your role has the **Manage data sharing** capability.
- To make a feed public, your role also has the **Manage credentials** capability.
- Each consumer that authenticates uses the API token of a user who has the **Access data sharing** capability. Give each tool its own user: what it receives follows that user's markings and organization, and its connections show under its name.

## Choose a kind of feed

| | Live stream | CSV feed | TAXII collection |
| --- | --- | --- | --- |
| Typical consumer | The stream connector of a SIEM, an EDR or a SOAR, another OpenCTI platform | A firewall, a proxy, a script | Any TAXII 2.1 client, such as an EDR or a sharing platform |
| What it receives | STIX 2.1 events, as server-sent events, within seconds of each change | A CSV file with the columns you choose, computed at each download | STIX 2.1 objects, page by page |
| Removals | A `delete` event when an object is deleted or stops matching the filters | An object leaves the file at the next download | Not signaled: the object is no longer returned |
| Related objects | Sent along: author, markings, relationships | Columns can read first-degree neighbors | Not resolved: the client fetches them |
| Volume per request | Continuous, from a start point the consumer chooses | 5,000 objects at most | 500 objects per page |
| URL | `/stream/<id>` | `/feeds/<id>` | `/taxii2/root/collections/<id>/objects/` |

A live stream keeps a tool in step with the knowledge base, removals included, which suits a SIEM or an EDR that alerts on or blocks indicators. A CSV feed suits a device that replaces its whole list at each download. A TAXII collection suits tools that only speak TAXII. Between two OpenCTI platforms, use a live stream: the receiving platform consumes it as described in [Import from another OpenCTI platform](/opencti/use/ingest/streams/).

## What every feed shares

**Filters** decide what a feed publishes: entity types, labels, markings, authors, scores, and any other filter of the platform. Leave them empty to publish everything the reader can see. The [filter key reference](/reference/opencti/filter-keys/) lists every key and operator.

**Access** decides who can read it:

- **A restricted feed** is read with a user's API token, and that user only gets what their markings and organizations let them see. In **Accessible for**, choose the users, groups or organizations allowed to read the feed. Leave it empty for every user with the **Access data sharing** capability.
- **A public feed** is readable by anyone with its URL, without authentication.

::::only{opencti=">=7.260811"}
When you make a feed public, choose a user in **Share data corresponding to permissions associated with this user**. The feed publishes only what this user can see: their markings, their organization and the authorized members of each object. For example, with a user cleared up to TLP:GREEN, a report marked TLP:RED never appears in the feed.

If that user is deleted, OpenCTI makes every public feed that used it private, and stops the live streams among them.

Feeds made public in an earlier version may have no user set: they publish everything that matches their filters. Edit them to set a user.
::::

::::only{opencti="<7.260811"}
:::warning
A public feed publishes everything that matches its filters, whatever the markings, because it reads the data with the platform's system rights. Filter public feeds on markings, such as TLP:CLEAR, and check what they return.
:::
::::

## Create a live stream

A live stream sends an event each time an object that matches its filters is created, updated or deleted. When an object enters the stream, for example because it receives the label you filter on, the stream also sends the objects it depends on, such as its author, its markings and, unless the consumer asks not to, its relationships, even if they do not match the filters. The access rules of the reader still apply to these objects.

:::steps

1. Open :menu[Data > Data sharing > Live streams] and click **Create Live Stream**.
2. Enter a **Name** and a **Description**.
3. Set who can read it in **Accessible for**, or turn on **Public stream**. See [What every feed shares](#what-every-feed-shares).
4. Add filters for the objects to publish.
5. Optionally, filter events by their origin.

   ::::only{opencti=">=7.260811"}
   Open **Advanced options** and filter on the user, group or organization that made the change, for example to leave out what another platform's connector wrote. A stream with origin filters cannot replay past events: a consumer that asks for a recovery receives a `no-recover` event and starts from the live events. The dependencies of an object are sent whatever their origin.
   ::::

   ::::only{opencti="<7.260811"}
   Origin filters are not available in this version: filter on the creator of objects instead.
   ::::

6. Click **Create**. The stream shows as **Stopped** in the **Status** column.
7. Open the actions menu (⋮) of the stream and choose **Start**. The status turns to **Started**, and consumers can connect. A stopped stream refuses connections.

:::

::screenshot{id=opencti-share-feeds-streams-list alt="The Live streams list shows each stream with its Stream ID and a copy button, whether it is public, its Started or Stopped status, the health of its consumers and its filters."}

:::example[Stream hospitality indicators to the SIEM in real time]
The SOC wants its SIEM to alert on every indicator the CTI team ties to the hospitality sector as soon as it is published, and to stop alerting on those that lose their relevance. The SIEM runs the stream connector of its vendor from the [connector catalog](/opencti/use/ingest/connectors/), which reads a live stream. In **Create a stream**:

- **Name**: `Hospitality indicators for the SIEM`
- **Description**: `Indicators labelled hotel-sector with a score of 60 or more, read by the SIEM stream connector.`
- **Public stream**: off. **Accessible for**: `SIEM stream reader`, the user of the connector, member of a group whose markings stop at TLP:AMBER.
- Filters:
  - **Entity type**, operator **Equals**, value **Indicator**
  - **Label**, operator **Equals**, value `hotel-sector`
  - **Score**, operator **Greater than/ Equals**, value `60`

Start the stream, then give the connector the platform URL, `https://opencti.example.com`, the stream ID copied from the **Stream ID** column, and the token of `SIEM stream reader`. Each event the SIEM receives carries one STIX object. Here is a creation, abridged (most STIX properties and the `origin` field are left out):

```text
id: 1768471200000-0
event: create
data: {"version":"4","message":"creates a Indicator `checkin-velvet.example`","data":{"type":"indicator","spec_version":"2.1","id":"indicator--3f6a1c52-8e47-4b0d-9a2e-6c1d7b5f0e93","name":"checkin-velvet.example","pattern":"[domain-name:value = 'checkin-velvet.example']","pattern_type":"stix","valid_from":"2025-12-22T00:00:00.000Z","extensions":{"extension-definition--ea279b3e-5c71-4632-ac08-831c66a786ba":{"score":85,"main_observable_type":"Domain-Name"}}}}
```

When decay lowers the score of `203.0.113.31` below 60, the stream sends a `delete` event for it, and the connector removes it from the SIEM. Indicators marked TLP:RED never reach the SIEM, because `SIEM stream reader` cannot see them.
:::

### What a consumer receives

The stream sends `create`, `update`, `delete` and `merge` events. An update that makes an object match the filters arrives as a `create`, and an update that makes it stop matching arrives as a `delete`, so the consumer's copy follows the filters. Each event has an ID built from its date, such as `1768471200000-0`. Between events, the platform sends a `heartbeat` every 5 seconds, so that proxies do not close the connection.

On one connection, the stream sends each dependency once. It remembers the objects it already sent, by default the last 5,000 for 2 hours, and does not send them again when another object refers to them (see [Limits](#limits)). The consumer must therefore keep the authors, markings and other dependencies it receives: one it drops comes back only when the stream forgets it, or when the consumer reconnects, since a new connection starts with an empty memory.

A consumer tunes what it receives with parameters of the stream URL, or headers of the same name:

| Parameter | Effect |
| --- | --- |
| `from` (or the `Last-Event-ID` header) | Starts after this event ID or date. A consumer that reconnects with the last ID it processed misses nothing |
| `recover` | With `from`, first sends the current state of the objects that changed between `from` and this date, then the live events |
| `listen-delete=false` | Ignores deletions |
| `no-dependencies=true` | Skips related objects |
| `with-inferences=true` | Adds the knowledge inferred by the rules engine |

See [Live streams API](/develop/apis/streams/) for the protocol, and [HTTP endpoints](/reference/opencti/http-endpoints/) for how the stream authenticates.

### Keep consumers in sync

The platform keeps its recent events in one Redis stream, shared by all live streams: by default the last 2,000,000 events (see [Limits](#limits)). A consumer that stays disconnected or slow until its last event leaves this window can no longer resume where it stopped: it has to reconnect with `recover` to catch up from the database.

The **Consumers** column of :menu[Data > Data sharing > Live streams] shows how many consumers are connected, and the worst health among them:

- **Healthy**: no consumer is falling behind.
- **Degraded**: a consumer may lose events within a day.
- **At risk**: a consumer may lose events within an hour.

Click the row of a stream to open **Stream consumers**, which refreshes every 5 seconds. Each consumer shows under the email of its user, with **Connected since**, its own health (**Healthy**, or the time left before it falls out of the retained events), and:

| Metric | What it measures |
| --- | --- |
| **Stream rate** | Events per second the whole platform writes to its stream |
| **Processing rate** | Events per second this consumer reads from the stream, before filtering |
| **Resolution rate** | Dependencies per second the stream loads and sends to this consumer |
| **Delivery rate** | Events per second sent to this consumer, after filtering |
| **Last event ID** | The date of the last event the consumer read. The info icon shows the ID itself |
| **Time lag** | How far behind the newest event the consumer is. **None** when it is caught up |

A **Processing rate** that stays below the **Stream rate** means the consumer falls behind: its **Time lag** grows until it is at risk. Check the consumer's side first, such as a slow SIEM ingestion or a network limit, then the load of the platform.

To change a stream, choose **Update** in its actions menu: the **Update a live stream** panel has the same fields as the creation. Filter changes apply to connected consumers from the next event. Objects that now match are not sent until they change, so a consumer that needs them at once reconnects with `recover`.

:::only{opencti=">=7.261119"}
A connected consumer also follows the rights of its user. Before each batch of events, the stream reads the user again: a change of its groups, organizations or markings applies from the next events, and the stream forgets the dependencies it already sent, so that they are checked again against the new rights. The stream ends the connection, and the consumer has to reconnect, when:

- its user is deleted or merged into another one;
- the account of its user is no longer active, for example locked or past its expiration date (for a restricted stream);
- you turn **Public stream** on or off, or choose another user in **Share data corresponding to permissions associated with this user**.
:::

To cut a consumer off at once, stop or delete its stream.

To check what a stream sends, choose **Open stream** in its actions menu: the browser shows its events.

## Create a CSV feed

A CSV feed answers each request with the objects that match its filters and were created, or updated, during the last minutes you choose. Its content is computed at each request: a consumer that downloads it every hour, with a rolling time of 60 minutes, gets each new object once.

:::steps

1. Open :menu[Data > Data sharing > CSV feeds] and click **Create Feed**.

   ::screenshot{id=opencti-share-feeds-csv-create alt="The Create a feed panel with the Name, Description, Separator and Rolling time (in minutes) fields, set to 60 minutes by default."}

2. Enter a **Name**, a **Description** and the **Separator** of the columns.
3. Set who can read it in **Accessible for**, or turn on **Public feed**. See [What every feed shares](#what-every-feed-shares).
4. Set the **Rolling time (in minutes)**, and in **Base attribute** choose whether it counts from the **Creation date** or the **Update date** of objects.
5. Choose the **Entity types** to publish, at least one, and add filters.
6. Turn on **Include headers in the feed** to put the column names on the first line.
7. Add the columns. For each column, enter a **Column name**, then, for each entity type, the **Attribute** that fills it.
8. Click **Create**.

:::

:::example[Feed the firewall with hospitality-sector IPv4 indicators]
The network team wants its firewall to block the IP addresses that the CTI team tags for the hospitality sector. The firewall downloads a list of addresses every hour, one per line, and replaces its previous list each time. In **Create a feed**:

- **Name**: `Hospitality IPv4 blocklist`
- **Description**: `Active IPv4 indicators of the hotel sector, for the perimeter firewall.`
- **Public feed**: off. **Accessible for**: the user account the firewall uses, for example `Firewall feed reader`.
- **Separator**: `,`
- **Rolling time (in minutes)**: `43200`, and **Base attribute**: **Update date**. Every indicator created or updated in the last 30 days comes back at each download, so the full list is rebuilt every hour.
- **Entity types**: **Indicator**
- Filters:
  - **Main observable type**, operator **Equals**, value **IPv4 address**
  - **Label**, operator **Equals**, value `hotel-sector`
  - **Score**, operator **Greater than/ Equals**, value `50`
  - **Revoked**, operator **Equals**, value `false`
- **Include headers in the feed**: off, since the firewall expects addresses only.
- One column: **Column name** `ip`, with the **Attribute** `name` for **Indicator**, since these indicators are named after their address.

After **Create**, copy the URL from the row of the feed and give it to the firewall, with the API token of `Firewall feed reader`:

```bash
curl -H "Authorization: Bearer ${OPENCTI_FEED_TOKEN}" \
  https://opencti.example.com/feeds/5f1c2a7e-3b9d-4e8a-a6c4-2d7f9b0e1c35
```

Each line of the answer is one address, such as `203.0.113.31`. Indicators that decay below a score of 50 or are revoked drop out of the list at the next download. For a SIEM lookup table, add a second column `score` filled with `x_opencti_score`, and turn the headers on.
:::

To fill a column from a related object, click **Direct** next to the entity type to switch it to **Relationship**, then choose the **Relationship type**, the **Target type** and the **Attribute** of the target. For example, an indicator feed can have a "Malware" column filled with the names of the malware the indicator `indicates`. When several objects match, **Multi-match** decides: **All (list)** joins them with the **List separator**, which must differ from the CSV separator, and **First match** keeps one.

A CSV feed returns at most 5,000 objects, the most recent first, and leaves out the rest without warning. Keep the rolling time short enough for the volume your filters match, or raise the limit (see [Limits](#limits)).

To start a feed from another one, choose **Duplicate** in its actions menu: the **Duplicate a feed** form opens with the same settings and the name followed by "- copy".

## Create a TAXII collection

:::steps

1. Open :menu[Data > Data sharing > TAXII collections] and click **Create TAXII Collection**.
2. Enter a **Name** and a **Description**.
3. Set who can read it in **Accessible for**, or turn on **Public collection**. See [What every feed shares](#what-every-feed-shares).
4. Turn on **Include inferences** to publish knowledge created by the rules engine, and **Copy OpenCTI scores to confidence level for indicators** for clients that read the confidence of indicators.
5. Add filters for the objects to publish.
6. Click **Create**.

:::

Clients read the collection through the TAXII 2.1 API of the platform, with an API token as bearer or with basic authentication. Objects come oldest change first, 500 per page at most: the answer says `"more": true` and gives a `next` value to ask for the following page. A client that polls asks for objects `added_after` the date of the last one it received. OpenCTI compares this date with the last update of each object, so a changed object comes back. A client can also narrow a request with `match[type]` or `match[id]`.

:::example[Give the EDR the hospitality indicators it can block]
The EDR reads TAXII 2.1 collections and blocks indicators whose confidence is 60 or more. In **Create a TAXII collection**:

- **Name**: `Hospitality indicators for the EDR`
- **Accessible for**: `EDR TAXII reader`
- **Copy OpenCTI scores to confidence level for indicators**: on, so the EDR reads the OpenCTI score as the confidence of each indicator.
- Filters: **Entity type**, operator **Equals**, value **Indicator**, and **Label**, operator **Equals**, value `hotel-sector`.

Every 15 minutes, the EDR polls `https://opencti.example.com/taxii2/root/collections/0b6e2f94-1c3d-4a7e-8f52-9d4c6a1b3e70/objects/?added_after=2026-01-14T00:00:00.000Z` with the token of `EDR TAXII reader`, and follows `next` while `more` is true. The filters do not test the score on purpose: a TAXII client is not told when an object leaves a collection. An indicator that decays must still come back, with its lower confidence, for the EDR to stop blocking it.
:::

## Limits

These limits are configuration keys of the platform, set by a platform engineer. Each has an environment variable: its name in upper case, with `__` between levels.

| Limit | Key | Environment variable | Default |
| --- | --- | --- | --- |
| Objects in one answer of a CSV feed | `data_sharing:max_csv_feed_result` | `DATA_SHARING__MAX_CSV_FEED_RESULT` | `5000` |
| Objects in one page of a TAXII collection. A client's `limit` cannot go above it | `app:data_sharing:taxii:max_pagination_result` | `APP__DATA_SHARING__TAXII__MAX_PAGINATION_RESULT` | `500` |
| Events the platform keeps for live streams to resume from | `redis:trimming` | `REDIS__TRIMMING` | `2000000` |
| Time between two heartbeats of a live stream, in milliseconds | `app:live_stream:heartbeat_period` | `APP__LIVE_STREAM__HEARTBEAT_PERIOD` | `5000` |
| Objects a live stream connection remembers as already sent, so as not to send them again as dependencies | `app:live_stream:cache_max_size` | `APP__LIVE_STREAM__CACHE_MAX_SIZE` | `5000` |
| Time, in hours, a live stream connection remembers an object it sent | `app:live_stream:cache_max_time` | `APP__LIVE_STREAM__CACHE_MAX_TIME` | `2` |
| Size of a request body the platform accepts, such as objects pushed to a [TAXII push](/opencti/use/ingest/taxii-push/) collection | `app:max_payload_body_size` | `APP__MAX_PAYLOAD_BODY_SIZE` | `50mb` |

:::only{opencti=">=7.261119"}
When `app:http:secure_by_default` (`APP__HTTP__SECURE_BY_DEFAULT`) is `true`, the size of a request body defaults to `10mb` instead of `50mb`: set `app:max_payload_body_size` if your TAXII push clients send larger bundles. The TAXII API root, `/taxii2/root/`, gives this limit in bytes as its `max_content_length`, such as `52428800` for `50mb`.
:::

Raise a result limit only after you test what your platform serves in one answer: a larger page costs memory on the platform and on Elasticsearch. Keeping more stream events costs Redis memory. The configuration reference describes each key with its theme: [Technical customization](/reference/opencti/configuration/#technical-customization), [Functional customization](/reference/opencti/configuration/#functional-customization), [Redis](/reference/opencti/configuration/#redis) and [Network and security](/reference/opencti/configuration/#network-and-security).

## Check that it works

Copy the URL of the feed from its row and open it with a user who should read it: the browser shows the stream's events, the CSV lines or the TAXII objects. For a public feed, open the URL in a private window. If the result is empty, check the filters and the rights of the user whose access applies. Once a tool reads a live stream, its connection shows in the **Consumers** column.

## What's next

- [Indicator decay](/opencti/configure/decay/), to set how scores drop, and so when indicators leave your feeds.
- [Import from another OpenCTI platform](/opencti/use/ingest/streams/), to consume a live stream on another platform.
- [Export knowledge to a file](/opencti/use/share/export/), to send a file once rather than a feed.
- [HTTP endpoints](/reference/opencti/http-endpoints/): the exact paths of feeds, TAXII collections and streams, and how each authenticates.
