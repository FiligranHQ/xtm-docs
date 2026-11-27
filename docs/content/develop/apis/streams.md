---
title: Data streaming
description: The live streams and the raw stream of OpenCTI over server-sent events, the events they send, how to resume after a cut, how to write a consumer, and the settings that tune them.
doc_type: reference
products:
  - opencti
personas: [developer, soc-manager, platform-engineer]
owner: docs-platform
last_reviewed: 2026-11-01
sidebar_position: 4
no_screenshot: "This page describes an HTTP protocol and its events: the screens that create and watch live streams are on the Native feeds page."
migrated_from:
  - opencti:/reference/streaming/
---

OpenCTI sends every change of its knowledge base as a stream of events, over server-sent events (SSE): a long HTTP response that the platform keeps writing to. A consumer, such as the stream connector of a SIEM or another OpenCTI instance, stays connected and applies each creation, update, deletion and merge as it happens, and resumes where it stopped after a cut. This page is the reference of the protocol: the endpoints, the events and their fields, the parameters, the rules a live stream follows to keep a consumer consistent, and the settings a platform engineer tunes. To create a live stream and watch its consumers in the interface, see [Share knowledge with your security tools](/opencti/use/share/feeds/#create-a-live-stream). Every event below was received from OpenCTI 7.261030 on the demo data, and trimmed.

::diagram{id=develop-apis-streams}

## Endpoints

Every change the platform makes, through the API, a connector, a playbook or the interface, adds one event to a Redis stream that all the endpoints read.

| Endpoint | What it sends | Who can connect |
| --- | --- | --- |
| `/stream/{id}` | A **live stream** created in :menu[Data > Data sharing > Live streams]: the events that match its filters, completed with the objects they depend on, as the user is allowed to see them | A user with the **Access data sharing** capability, among the members the stream allows; anyone when the stream is public |
| `/stream/live` | The same processing as a live stream, with no filter: every change, with its dependencies | A user with the **Bypass all capabilities** capability |
| `/stream` | The **raw stream**: every event as the platform stored it, with no filter, no dependency and no recovery. For replication and backup tools | A user with the **Bypass all capabilities** capability |

A consumer authenticates like any API client, with `Authorization: Bearer` followed by an API token, and its user also needs **Allow token usage**. See [Query OpenCTI with GraphQL](/develop/apis/graphql/#get-an-api-token) to get a token. Give each consumer its own user: what it receives follows the markings and organizations of that user, and its connections show under its name in **Stream consumers**.

## Connect

```bash
curl -N "https://opencti.example.com/stream/d6ac3e7d-a29c-4c88-92b6-00c43ef2c7bf" \
  -H "Authorization: Bearer ${OPENCTI_TOKEN}"
```

`-N` turns off the buffering of `curl`, so that events show as they arrive. The platform answers `200` with `Content-Type: text/event-stream; charset=utf-8`, then sends a `connected` event:

```text
event: connected
data: {"lastEventId":"1793572800397-0","firstEventId":"1793572725744-0","firstEventDate":"2026-11-01T22:38:45.744Z","lastEventDate":"2026-11-01T22:40:00.397Z","streamSize":548,"connectionId":"33002d43-6b3b-400f-8f67-3c6dd8b00d38"}
```

`firstEventId` and `lastEventId` bound the events the platform still holds, `streamSize` counts them, and `connectionId` identifies this connection in **Stream consumers**. Without a start point, the stream then sends the events that happen from now on.

A connection that fails gets an HTTP status, with the reason in the status line:

| Status | Reason |
| --- | --- |
| `401` | `You are not authenticated, please check your credentials`: no token, a wrong or expired one, or a user without **Allow token usage** |
| `401` | `You are not authorized, please check your credentials`: the stream does not exist, the user is not among its members, or lacks **Access data sharing** (**Bypass all capabilities** for `/stream/live`) |
| `401` | `Consume generic stream is only authorized for bypass user`: the user of a `/stream` connection lacks **Bypass all capabilities** |
| `401` | `You need to have access to specific markings for this live stream`: the stream filters on markings the user cannot see |
| `410` | `This live stream is stopped`: start it in :menu[Data > Data sharing > Live streams] |

A connection authenticates when it opens. To cut a consumer off at once, stop or delete its live stream: the platform checks the stream between two batches of events, every 5 seconds at most, and ends the response.

## Events

Each event is a block of `id`, `event` and `data` lines, ended by an empty line. `id` is the ID of the event in the Redis stream: the time in milliseconds, a dash and a sequence number, such as `1793573158396-0`. `event` is the event name, and `data` a JSON object on one line.

| `event` | Sent when | `id` |
| --- | --- | --- |
| `connected` | The connection opens | None |
| `create` | An object is created, or starts to match the filters, or is sent as a dependency or by a recovery | The event that caused it |
| `update` | An object that matches the filters changes | The event |
| `delete` | An object is deleted, or stops matching the filters | The event |
| `merge` | Objects are merged into one that matches the filters | The event |
| `heartbeat` | Every 5 seconds, when nothing else is waiting to be written | The last event read |
| `consumer_metrics` | After each `heartbeat` | The last event read |
| `error` | The stream cannot go on, such as a recovery asked without a start date | None |
| `no-recover` | A recovery is asked on a stream with origin filters (see [Resume after a cut](#resume-after-a-cut)) | Now |

### The data of an event

The `data` of `create`, `update`, `delete` and `merge` events holds:

| Field | Content |
| --- | --- |
| `version` | `"4"`, the version of the event format. Live streams send only version 4 events |
| `type` | The type of change the platform recorded: `create`, `update`, `delete` or `merge` |
| `scope` | `external` for knowledge. Internal objects are not sent |
| `message` | A sentence for people, such as ``creates a Indicator `guest-wifi-velvet.example` `` |
| `origin` | Who made the change, or why the stream sent the event ([The origin of an event](#the-origin-of-an-event)) |
| `data` | The object as STIX 2.1, with the OpenCTI properties in the extension `extension-definition--ea279b3e-5c71-4632-ac08-831c66a786ba`: internal `id`, `type`, `created_at`, `updated_at`, `score`, `labels_ids`, `created_by_ref_id`... |
| `context` | The details of an update or a merge: [Updates](#updates), [Merges](#merges) |
| `event_id` | The ID the platform gave the change when it made it |

:::warning
On a live stream, act on the **`event` name**, not on `data.type`. When an update makes an object stop matching the filters, the stream sends it as a `delete` event whose `data.type` is still `update`, and when an update makes it match, as a `create` event whose `data.type` is `update`.
:::

A creation, as the SIEM live stream of the demo received it when a script created the indicator `guest-wifi-velvet.example`:

```text
id: 1793573158396-0
event: create
data: {"version":"4","type":"create","scope":"external","message":"creates a Indicator `guest-wifi-velvet.example`","origin":{"socket":"query","user_id":"88ec0c6a-13ce-5e39-b486-354fe4a7084f","group_ids":["337534e7-30f8-4bc8-9c54-782693914905"],"organization_ids":[]},"data":{"id":"indicator--a6b80844-c4b6-5dc7-aee8-3a0af2933801","spec_version":"2.1","type":"indicator","extensions":{"extension-definition--ea279b3e-5c71-4632-ac08-831c66a786ba":{"extension_type":"property-extension","id":"3ed349ba-f227-4fb2-8203-e9683b328a9d","type":"Indicator","created_at":"2026-11-01T22:45:58.356Z","updated_at":"2026-11-01T22:45:58.356Z","is_inferred":false,"labels_ids":["8a316f32-14e5-49ab-a685-62f7718a1a2d"],"created_by_ref_id":"13a8c72c-08b2-4262-bfac-82ecd17bad4c","detection":false,"score":75,"main_observable_type":"Domain-Name"}},"created":"2026-11-01T22:45:58.356Z","modified":"2026-11-01T22:45:58.356Z","revoked":false,"confidence":100,"labels":["hotel-sector"],"object_marking_refs":["marking-definition--34098fce-860f-48ae-8e50-ebd3cc5e41da"],"created_by_ref":"identity--ac6f7b3a-7aa5-558d-a1bd-4a4f52840f9e","name":"guest-wifi-velvet.example","pattern":"[domain-name:value = 'guest-wifi-velvet.example']","pattern_type":"stix","valid_from":"2026-11-01T22:45:58.340Z","valid_until":"2027-04-07T08:31:02.884Z"},"event_id":"1793573158356-0"}
```

Before it, with the same `id`, the stream sent the author `Example CERT` and the marking `TLP:GREEN` as two `create` events with `"origin":{"referer":"init-dependencies"}`: the consumer had not received them yet on this connection.

### Updates

An `update` carries the whole object after the change, and in `context` what changed:

- `patch`: the JSON Patch operations ([RFC 6902](https://www.rfc-editor.org/rfc/rfc6902)) that turn the previous object into this one.
- `reverse_patch`: the operations that turn this object back into the previous one.
- `changes`: the changed fields as the history of the object shows them, with the values removed and added.

The update of the score from 75 to 65, which also moved the end of validity, abridged:

```text
id: 1793573165501-0
event: update
data: {"version":"4","type":"update","scope":"external","message":"Update 2 elements","origin":{"socket":"query","user_id":"88ec0c6a-13ce-5e39-b486-354fe4a7084f","group_ids":["337534e7-30f8-4bc8-9c54-782693914905"],"organization_ids":[]},"data":{"id":"indicator--a6b80844-c4b6-5dc7-aee8-3a0af2933801","type":"indicator","name":"guest-wifi-velvet.example","valid_until":"2027-08-28T22:46:05.435Z","extensions":{"extension-definition--ea279b3e-5c71-4632-ac08-831c66a786ba":{"score":65}}},"context":{"patch":[{"op":"replace","path":"/valid_until","value":"2027-08-28T22:46:05.435Z"},{"op":"replace","path":"/modified","value":"2026-11-01T22:46:05.465Z"},{"op":"replace","path":"/extensions/extension-definition--ea279b3e-5c71-4632-ac08-831c66a786ba/score","value":65},{"op":"replace","path":"/extensions/extension-definition--ea279b3e-5c71-4632-ac08-831c66a786ba/updated_at","value":"2026-11-01T22:46:05.465Z"}],"reverse_patch":[{"op":"replace","path":"/valid_until","value":"2027-04-07T08:31:02.884Z"},{"op":"replace","path":"/extensions/extension-definition--ea279b3e-5c71-4632-ac08-831c66a786ba/score","value":75}],"changes":[{"field":"Indicator--valid_until","changes_removed":[{"raw":"2027-04-07T08:31:02.884Z"}],"changes_added":[{"raw":"2027-08-28T22:46:05.435Z"}]},{"field":"Indicator--x_opencti_score","changes_removed":[{"raw":"75"}],"changes_added":[{"raw":"65"}]}]},"event_id":"1793573165465-0"}
```

### Deletions

A `delete` carries the object as it was before the change. When an update makes the object leave the filters, as when its score drops from 65 to 40 below the `60` of the SIEM stream, the stream sends the previous state of the object with an empty `context`, so that the consumer learns nothing about what it can no longer see:

```text
id: 1793573168595-0
event: delete
data: {"version":"4","type":"update","scope":"external","message":"Update 2 elements","data":{"id":"indicator--a6b80844-c4b6-5dc7-aee8-3a0af2933801","type":"indicator","name":"guest-wifi-velvet.example","extensions":{"extension-definition--ea279b3e-5c71-4632-ac08-831c66a786ba":{"score":65}}},"context":{},"event_id":"1793573168569-0"}
```

When the object itself is deleted, `data.type` is `delete` and the event has no `context`.

### Merges

A `merge` carries the object that remains, with `context.patch` and `context.reverse_patch` as for an update, and `context.sources`: the objects merged into it, as STIX. On a live stream, `sources` only lists the merged objects the user can see, and `message` names only them. The consumer replaces the sources by the target and deletes the sources on its side.

### The origin of an event

`origin` tells who made the change, or flags the events the stream adds itself:

::::only{opencti=">=7.261030"}
| Field | Content |
| --- | --- |
| `socket` | How the change came: `query` for the API and the interface |
| `user_id`, `group_ids`, `organization_ids` | The user who made the change, their groups and organizations |
| `applicant_id` | The user a connector or a playbook acted for |
| `playbook_id` | The playbook that made the change |
| `referer` | Only on events the stream adds: `init-dependencies` for a dependency, `init-create` for an object sent by a recovery |

Other attributes of the origin, such as the IP address of the client, stay in the platform.
::::

::::only{opencti="<7.261030"}
`origin` holds the whole origin the platform recorded for the change: `socket`, `ip`, `user_id`, `group_ids`, `organization_ids`, `user_metadata`, `referer`, `applicant_id`, `call_retry_number` and `playbook_id`. On events the stream adds, `referer` is `init-dependencies` for a dependency and `init-create` for an object sent by a recovery.
::::

### Heartbeats and consumer metrics

Every 5 seconds (`app:live_stream:heartbeat_period`), when nothing else waits in the connection, the stream sends a `heartbeat` and a `consumer_metrics` event. Both carry as `id` the last event the stream read for this connection, including the events its filters left out.

```text
id: 1793573165501-0
event: heartbeat
data: "2026-11-01T22:46:05.501Z"

id: 1793573165501-0
event: consumer_metrics
data: {"deliveryRate":0.51,"processingRate":0.21,"resolutionRate":2,"productionRate":0,"timeLag":0,"estimatedOutOfDepth":0}
```

On the SIEM stream of the demo, the heartbeat after the indicator was deleted carried the ID `1793573171749-0`: the deletion itself was not sent, since the indicator had already left the filters, but the stream had read it.

The `heartbeat` data is the date of that last event. The rates are events per second: `productionRate` written to the platform stream, `processingRate` read by this connection, `deliveryRate` sent to it after filtering, `resolutionRate` sent as dependencies. `timeLag` is how many seconds the connection is behind the newest event, and `estimatedOutOfDepth` how many seconds remain, at these rates, before its position leaves the events the platform holds (`0` when it is not falling behind). **Stream consumers** in the interface shows the same figures.

## Resume after a cut

A consumer that records the `id` of the last event it processed can resume from it. It sends the ID in one of these, in this order of precedence:

| Where | Example |
| --- | --- |
| `from` query parameter | `/stream/d6ac3e7d-a29c-4c88-92b6-00c43ef2c7bf?from=1793573158000-0` |
| `from` header | `from: 1793573158000-0` |
| `Last-Event-ID` header, which SSE clients send on their own when they reconnect | `Last-Event-ID: 1793573158000-0` |

`from` takes an event ID, a date such as `2026-11-01T22:45:00Z`, or `0` to start from the oldest event the platform holds. Record the `id` of `heartbeat` events too: it moves past the events the filters left out, so a resume does not read them again.

Reconnected with `from=1793573158000-0`, the SIEM stream sent again, in order, the author and the marking as dependencies, then the creation, the update and the deletion of the indicator, with their original IDs.

The platform holds the last 2,000,000 events by default (`redis:trimming`). On a busy platform this covers days, not weeks. A consumer whose last ID left this window cannot resume from it: it needs a recovery.

### Recover from the database

`recover`, a date, asks the stream to rebuild the current state from the database before going live. With `from`, the stream first sends, as `create` events with `"origin":{"referer":"init-create"}`, the objects that match the filters and were updated between `from` and `recover`, with their dependencies. Then it sends the live events from the `recover` date on. `recover` is also read from the `recover` or `recover-date` header.

```bash
curl -N "https://opencti.example.com/stream/d6ac3e7d-a29c-4c88-92b6-00c43ef2c7bf?from=2026-01-01T00:00:00Z&recover=2026-11-01T22:47:30Z" \
  -H "Authorization: Bearer ${OPENCTI_TOKEN}"
```

On the demo data, the SIEM stream answered with the author `Example CERT` as a dependency, then the four indicators that match its filters, `203.0.113.31`, `reservations-velvet.example`, `guest-wifi-login.example` and `checkin-velvet.example`, then the live events.

- Without `from`, the stream sends an `error` event, `Recovery mode is only possible with a start date.`, and nothing else.
- During a recovery, the `id` of each event is the update date of the object it carries, so IDs follow the update dates rather than the stream. If a recovery is cut, start it again with the same `from` and `recover`, and make the consumer apply events by STIX ID, so that objects received twice change nothing.

::::only{opencti=">=7.260811"}
A live stream with origin filters cannot recover: the database does not keep the origin of each change. Asked for a recovery, such a stream sends a `no-recover` event, with the `resume_from` ID it starts from, then the live events.
::::

## Options

Each option is a query parameter of the stream URL, or a header of the same name.

| Option | Default | Effect |
| --- | --- | --- |
| `listen-delete=false` | `true` | No `delete` events: neither deletions nor objects that leave the filters |
| `no-dependencies=true` | `false` | The stream still sends the objects an event refers to, such as the author, the markings, the content of a container or the two ends of a relationship. It no longer sends the relationships of a matching object, nor the relationships that only touch a matching object. `no-relationships` is a former name of the same option |
| `with-inferences=true` | `false` | Adds the objects and relationships created by the rules engine, and searches them during a recovery |

These options apply to `/stream/{id}` and `/stream/live`. The raw stream only reads `from`.

## What a live stream sends

A live stream does more than filter the raw stream. These rules keep the copy of a consumer consistent with what its user can see:

1. **The filters decide.** An event is sent when its object matches the filters. An update that makes an object match is sent as `create`; one that makes it stop matching, as `delete`. Only some filter keys work on events: see [Filters on stream events](/develop/apis/filters/#filters-on-stream-events). Filter changes apply from the next batch of events.
2. **Dependencies come first.** Before an object, the stream sends the objects it refers to that the connection has not received yet: author, markings, the objects of a container, the source and target of a relationship. Then it sends the relationships of the object, unless `no-dependencies=true`.
3. **Each dependency is sent once per connection.** The stream remembers what it sent, by default the last 5,000 objects for 2 hours (`app:live_stream:cache_max_size`, `app:live_stream:cache_max_time`). A consumer must keep the dependencies it receives. A new connection starts with an empty memory.
4. **Relationships follow their ends.** A relationship that does not match the filters is still sent when its source or its target matches, as a dependency.
5. **Containers carry their content.** When an object that does not match the filters changes, and a container that matches holds it, such as a report on Velvet Harbor, the stream sends the change.
6. **Access rules apply.** The stream sends only what the user can see: markings, organizations, authorized members. It removes from `object_refs` the objects the user cannot see. It removes the organizations an object is shared with (`granted_refs`) unless the user has the **Restrict organization access** capability. A relationship is sent only when the user can see both its ends.
7. **Inferences stay out** unless `with-inferences=true`.

## Write a consumer

The stream connectors of the [connector catalog](/opencti/use/ingest/connectors/) already consume live streams for SIEMs, EDRs and SOARs, and another OpenCTI instance consumes them as described in [Import from another OpenCTI instance](/opencti/use/ingest/streams/). For your own tool, a consumer needs four things: read the events, act on the `event` name, record the last ID, and reconnect from it.

:::example[Keep a blocklist in step with the SIEM stream]
The SOC of Example CERT feeds an in-house blocklist service from the live stream `Hospitality indicators for the SIEM`, which publishes the indicators labelled `hotel-sector` with a score of 60 or more. This consumer, in Python with the standard library only, upserts or removes each indicator by its STIX ID, records the last event ID in `last-event-id.txt`, and resumes from it after a cut:

```python
import json
import os
import pathlib
import time
import urllib.request

OPENCTI_URL = os.environ["OPENCTI_URL"]      # https://opencti.example.com
TOKEN = os.environ["OPENCTI_TOKEN"]          # token of the consumer's own user
STREAM_ID = os.environ["STREAM_ID"]          # d6ac3e7d-a29c-4c88-92b6-00c43ef2c7bf
STATE = pathlib.Path("last-event-id.txt")


def handle(event, data):
    stix = data["data"]
    if event in ("create", "update"):
        print(f"upsert {stix['type']} {stix['id']} {stix.get('name', '')}")
    elif event == "delete":
        print(f"remove {stix['type']} {stix['id']}")
    elif event == "merge":
        print(f"merge into {stix['id']}: {[s['id'] for s in data['context']['sources']]}")


def listen():
    headers = {"Authorization": f"Bearer {TOKEN}", "Accept": "text/event-stream"}
    if STATE.exists():
        headers["Last-Event-ID"] = STATE.read_text().strip()
    request = urllib.request.Request(f"{OPENCTI_URL}/stream/{STREAM_ID}", headers=headers)
    with urllib.request.urlopen(request, timeout=60) as response:
        event, event_id, data = None, None, []
        for raw in response:
            line = raw.decode("utf-8").rstrip("\n")
            if line.startswith("id: "):
                event_id = line[4:]
            elif line.startswith("event: "):
                event = line[7:]
            elif line.startswith("data: "):
                data.append(line[6:])
            elif line == "":
                if event in ("create", "update", "delete", "merge"):
                    handle(event, json.loads("\n".join(data)))
                if event_id and event != "connected":
                    STATE.write_text(event_id)
                event, event_id, data = None, None, []


while True:
    try:
        listen()
    except Exception as error:
        print(f"connection lost ({error}), resuming in 10 seconds")
        time.sleep(10)
```

Started with `1793573158000-0` in `last-event-id.txt`, it printed:

```text
upsert identity identity--ac6f7b3a-7aa5-558d-a1bd-4a4f52840f9e Example CERT
upsert marking-definition marking-definition--34098fce-860f-48ae-8e50-ebd3cc5e41da TLP:GREEN
upsert indicator indicator--a6b80844-c4b6-5dc7-aee8-3a0af2933801 guest-wifi-velvet.example
upsert indicator indicator--a6b80844-c4b6-5dc7-aee8-3a0af2933801 guest-wifi-velvet.example
remove indicator indicator--a6b80844-c4b6-5dc7-aee8-3a0af2933801
```

The read timeout of 60 seconds is safe because a heartbeat arrives every 5 seconds: a silence of a minute means the connection is gone. A real consumer would write the blocklist where this one prints, and keep the identities and markings it receives to resolve `created_by_ref` and `object_marking_refs`.
:::

## Configuration

These settings apply to every stream of the platform. A platform engineer sets them in the configuration file or as environment variables (the key in upper case, with `__` between levels), as [Configuration](/reference/opencti/configuration/#technical-customization) describes.

| Key | Environment variable | Default | Effect |
| --- | --- | --- | --- |
| `app:live_stream:cache_max_size` | `APP__LIVE_STREAM__CACHE_MAX_SIZE` | `5000` | Objects one live stream connection remembers as sent, so as not to send them again as dependencies. Raise it when consumers receive the same authors and markings again and again on large streams; each connection holds its own memory |
| `app:live_stream:cache_max_time` | `APP__LIVE_STREAM__CACHE_MAX_TIME` | `2` | Hours an object stays in that memory |
| `app:live_stream:heartbeat_period` | `APP__LIVE_STREAM__HEARTBEAT_PERIOD` | `5000` | Milliseconds between two heartbeats. Raise it when the proxies between the platform and its consumers allow long silences; keep it below their idle timeout, or they close the connections |
| `redis:trimming` | `REDIS__TRIMMING` | `2000000` | Events the platform holds for streams to resume from, approximately. `0` keeps them all. More events cost Redis memory. See [Redis](/reference/opencti/configuration/#redis) |

:::only{opencti=">=7.261030"}
| Key | Environment variable | Default | Effect |
| --- | --- | --- | --- |
| `app:live_stream:resolution_batch_size` | `APP__LIVE_STREAM__RESOLUTION_BATCH_SIZE` | `500` | Dependencies a live stream loads at once when an object refers to many others, such as a report with tens of thousands of objects. The stream sends them in batches of this size as they are ready, instead of loading them all first. Lower it to use less memory per connection, raise it to send large containers faster |
:::

:::only{opencti=">=7.260811"}
| Key | Environment variable | Default | Effect |
| --- | --- | --- | --- |
| `app:stream_full_debug_mode` | `APP__STREAM_FULL_DEBUG_MODE` | `false` | Logs every event written to the stream, with its full content, at the `info` level. For debugging only: it writes a log line per change, with the data of the change. See [Logging](/reference/opencti/configuration/#logging) |
:::

For a platform whose consumers sit behind a proxy that closes idle connections after 30 seconds, and which keeps a week of events at about 500,000 events a day, with Docker Compose:

```yaml
services:
  opencti:
    environment:
      - APP__LIVE_STREAM__HEARTBEAT_PERIOD=15000
      - REDIS__TRIMMING=3500000
```

## What's next

- [Share knowledge with your security tools](/opencti/use/share/feeds/): create a live stream, choose its filters and watch its consumers.
- [Filters](/develop/apis/filters/): the filter format, and the keys that work on events.
- [Import from another OpenCTI instance](/opencti/use/ingest/streams/): consume a live stream on another OpenCTI instance.
- [HTTP endpoints](/reference/opencti/http-endpoints/): every endpoint of the platform and how it authenticates.
