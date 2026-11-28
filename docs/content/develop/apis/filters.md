---
title: Filters
description: The filter group format that OpenCTI list queries, feeds, live streams, triggers and playbooks share, its operators and special keys, and the simpler format of the OpenAEV API.
doc_type: reference
products:
  - opencti
  - openaev
personas: [developer]
owner: docs-platform
last_reviewed: 2026-11-01
sidebar_position: 3
no_screenshot: "This page describes a JSON format of the APIs: the filter menu of the interface builds the same structure, and the diagram shows it."
migrated_from:
  - opencti:/reference/filters/
  - openaev:/reference/apis/filters/
---

A filter group selects objects by their attributes and relationships. In OpenCTI, the same structure filters a list in the GraphQL API, a dashboard widget, a CSV feed, a TAXII collection, a live stream, a trigger and a playbook; the filter menu of the interface builds it for you. This page describes the structure, how the operators behave, the special keys that follow relationships, and what changes when the filter runs on stream events. The [filter keys reference](/reference/opencti/filter-keys/) lists, for this version, every key, the entity types it applies to and the operators it accepts. Every request below was run on the demo data of OpenCTI 7.261030.

## Structure

A filter group has a `mode` and two lists, both required even when empty: `filters` and nested `filterGroups`.

```ts
type FilterGroup = {
  mode: 'and' | 'or'          // how the members of the group combine
  filters: Filter[]
  filterGroups: FilterGroup[] // nested groups, each with its own mode
}

type Filter = {
  key: string[]       // one key; a plain string is accepted
  values: any[]       // strings, except for the special keys below
  operator?: string   // default: 'eq'
  mode?: 'and' | 'or' // how the values combine, default: 'or'
}
```

- **The group `mode`** combines its filters and its nested groups: with `and`, every member must match; with `or`, one is enough.
- **The filter `mode`** combines the values of one filter: with `eq` and `or`, the attribute equals one of the values; with `and`, it holds all of them.
- **One key per filter.** `key` is a list for historical reasons. Give it one key: filters with several keys are reserved to internal cases.
- **Values are strings**, numbers included: `["80"]` for a confidence above 80.

A group missing `filterGroups` is refused before it runs: `Expected value of type "FilterGroup" to include required field "filterGroups"`.

::diagram{id=develop-apis-filters}

:::example[Reports of Example CERT to review]
The CTI lead of Example CERT wants the reports the team authored that are either highly confident or not marked yet. The condition is "type is Report, author is Example CERT, and (confidence above 80 or no marking)":

```json
{
  "mode": "and",
  "filters": [
    {"key": "entity_type", "values": ["Report"]},
    {"key": "createdBy", "values": ["13a8c72c-08b2-4262-bfac-82ecd17bad4c"]}
  ],
  "filterGroups": [
    {
      "mode": "or",
      "filters": [
        {"key": "confidence", "values": ["80"], "operator": "gt"},
        {"key": "objectMarking", "values": [], "operator": "nil"}
      ],
      "filterGroups": []
    }
  ]
}
```

Passed as the `filters` variable of `stixCoreObjects(first: 20, filters: $filters)`, it returns 11 reports, among them:

```json
{"entity_type":"Report","representative":{"main":"Velvet Harbor infrastructure moves to new hosting"},"confidence":100,"objectMarking":[]}
{"entity_type":"Report","representative":{"main":"Velvet Harbor targets hotel booking systems"},"confidence":60,"objectMarking":[]}
```

The second report has a confidence of 60 but no marking, so the nested `or` group keeps it. The reports of Example CERT marked TLP:GREEN with a confidence of 60 or 70 are left out.
:::

## Where filters go

| Where | How the filter group is passed |
| --- | --- |
| List queries of the GraphQL API (`reports`, `indicators`, `stixCoreObjects`...) | The `filters` argument, of type `FilterGroup`. See [Query OpenCTI with GraphQL](/develop/apis/graphql/) |
| Live streams, CSV feeds, TAXII collections, triggers, playbooks, dashboard widgets | Stored with the object, as a JSON string in its `filters` field |

A stored filter is the same structure, serialized. The live stream of the demo that feeds a SIEM stores this, read from `streamCollections { edges { node { name filters } } }`:

```json
{"name":"Hospitality indicators for the SIEM","filters":"{\"mode\":\"and\",\"filterGroups\":[],\"filters\":[{\"key\":[\"entity_type\"],\"values\":[\"Indicator\"],\"operator\":\"eq\",\"mode\":\"or\"},{\"key\":[\"objectLabel\"],\"values\":[\"8a316f32-14e5-49ab-a685-62f7718a1a2d\"],\"operator\":\"eq\",\"mode\":\"or\"},{\"key\":[\"x_opencti_score\"],\"values\":[\"60\"],\"operator\":\"gte\",\"mode\":\"or\"}]}"}
```

To create or change such an object through the API, send the filter group with `JSON.stringify` (or `json.dumps`) in the `filters` field of its input. The filters of a live stream, a trigger or a playbook run on events, not on the database: see [Filters on stream events](#filters-on-stream-events).

## Keys and values

The platform checks every key before it runs a query. A key that does not exist fails the query instead of returning an empty list, so a typo cannot pass for "no result":

```json
{"errors":[{"message":"Incorrect filter keys not existing in any schema definition","path":["reports"],"extensions":{"code":"BAD_USER_INPUT_ERROR","data":{"http_status":400,"genre":"BUSINESS","keys":["object-label"]}}}],"data":{"reports":null}}
```

Values are checked less. Keys that point to other objects, such as `createdBy`, `objectMarking`, `objects` or `creator_id`, take the **internal ID** of those objects. A name or a STIX ID matches nothing, without an error: `createdBy` with `Example CERT`, or with the STIX ID `identity--ac6f7b3a-7aa5-558d-a1bd-4a4f52840f9e`, returns no report, while the internal ID `13a8c72c-08b2-4262-bfac-82ecd17bad4c` returns 13. Labels are the exception: `objectLabel` matches the label ID or its value, such as `hotel-sector`.

Keys come in three kinds, all in the [filter keys reference](/reference/opencti/filter-keys/):

- **Attributes**, such as `name`, `published`, `confidence`, `x_opencti_score`.
- **Relationships stored on the object**, such as `objectLabel`, `objectMarking`, `createdBy`, `objects` (the content of a container).
- **Special keys**, which the platform resolves into other filters: `regardingOf`, `dynamicRegardingOf`, `dynamicFrom`, `dynamicTo`, `ids` (any ID of the object), `alias` (`aliases` and `x_opencti_aliases` together), `computed_reliability` (the reliability of the object, or of its author when it has none), `sightedBy`, `workflow_id` and more. Some of them have their own value format, described in [Keys that follow relationships](#keys-that-follow-relationships).

## Operators

`eq` applies when `operator` is left out. The [filter keys reference](/reference/opencti/filter-keys/#operators) lists the 23 operators, their label in the filter menu, and the value types each one applies to. The rules that are easy to miss:

| Operator | Rule |
| --- | --- |
| `eq`, `not_eq` | With several values, `mode` decides: `or` matches one of them, `and` all of them. `eq` with an empty `values` means "has no value", `not_eq` with an empty `values` "has a value" |
| `nil`, `not_nil` | The key has no value, or has one. Send `"values": []` |
| `only_eq_to`, `not_only_eq_to` | For keys with several values, such as labels. With `mode` `or`, the default, the object holds a single value, one of those given; with `and`, it holds exactly all the values given and no other. `objectLabel` `only_eq_to` `hotel-sector` keeps the reports whose only label is `hotel-sector` |
| `gt`, `gte`, `lt`, `lte` | Numbers and dates. A date is an ISO 8601 date, such as `2025-12-01T00:00:00Z`, or a relative date, such as `now-1d` |
| `within` | Exactly two values, the start and the end. Each is a relative date (`now`, `now-7d`, `now-1M/M`) or a full ISO 8601 date with milliseconds, as `2026-01-01T00:00:00.000Z`. Without the milliseconds, the query fails: `The values for filter with "within" operator are not valid` |
| `contains`, `starts_with`, `ends_with` and their `not_` forms | Short texts. `contains` looks for the exact sequence of words given |
| `search` | Full-text search on the words of the value, in any order. On the demo data, `name` `search` `booking hotel` returns 8 reports, `name` `contains` `hotel booking` returns 3 |
| `has_changed`, `not_has_changed` | Only in filters that run on events: see [Filters on stream events](#filters-on-stream-events) |
| `match`, `wildcard`, `script` | API only, for search engine queries. `script` is refused unless the platform allows script filters |

The reports published in January 2026, with `within`:

```json
{"mode":"and","filters":[{"key":"published","values":["2026-01-01T00:00:00.000Z","2026-01-31T23:59:59.999Z"],"operator":"within"}],"filterGroups":[]}
```

```json
{"data":{"reports":{"pageInfo":{"globalCount":4},"edges":[
  {"node":{"name":"Night-time exports from a hotel booking platform","published":"2026-01-09T09:00:00.000Z"}},
  {"node":{"name":"Velvet Harbor targets hotel booking systems","published":"2026-01-12T09:00:00.000Z"}},
  {"node":{"name":"Velvet Harbor phishing kit mimics hotel booking confirmations","published":"2026-01-14T08:00:00.000Z"}},
  {"node":{"name":"Velvet Harbor reuses guest Wi-Fi portal infrastructure","published":"2026-01-15T07:30:00.000Z"}}]}}}
```

## Keys that follow relationships

### regardingOf

`regardingOf`, **In regards of** in the filter menu, keeps the objects that have a relationship of the given types with the given objects, whichever side they are on. Its `values` are two sub-filters, `id` and `relationship_type`; leave one out to accept any:

```json
{"mode":"and","filters":[{"key":"regardingOf","values":[
  {"key":"id","values":["cd92e87a-28a6-4817-81a1-b3305df36c8c"]},
  {"key":"relationship_type","values":["uses"]}]}],"filterGroups":[]}
```

On `stixCoreObjects`, this returns the 9 objects Velvet Harbor uses: 5 attack patterns, such as `[T1566.001] Spearphishing Attachment`, the infrastructure `Velvet Harbor C2`, and the malware `HarborGate backdoor`, `Harbor Stealer` and `Velvet Loader`.

`regardingOf` reads the IDs that each object stores of its related objects. For performance, some relationships are stored on one side only, and `regardingOf` misses them from the other side:

| Relationship | Found from | Missed from |
| --- | --- | --- |
| `related-to` from an observable | The observable: the observables related to a malware | The other end: the objects related to an IP address |
| `located-at` from an IPv4 or IPv6 address, or a city, to a country or a region | The address or the city | The country or the region |
| `indicates` | The indicator: the indicators of Velvet Harbor (7 on the demo data) | The indicated object: the objects an indicator indicates (none found) |

To list relationships of these kinds from the side that does not store them, query `stixCoreRelationships` with `fromId` or `toId` instead. The filter menu shows a warning icon on a **In regards of** filter whose values involve one of these cases.

### dynamicRegardingOf

`dynamicRegardingOf`, **In regards of (dynamic)** in the filter menu, does the same with the related objects chosen by a filter instead of by ID. Its `values` are a `relationship_type` sub-filter and a `dynamic` sub-filter that holds a filter group. The intrusion sets that target the hospitality sector:

```json
{"mode":"and","filters":[
  {"key":"entity_type","values":["Intrusion-Set"]},
  {"key":"dynamicRegardingOf","values":[
    {"key":"relationship_type","values":["targets"]},
    {"key":"dynamic","values":[{"mode":"and","filters":[
      {"key":"entity_type","values":["Sector"]},
      {"key":"name","values":["Hospitality"]}],"filterGroups":[]}]}]}],
 "filterGroups":[]}
```

```json
{"data":{"stixCoreObjects":{"pageInfo":{"globalCount":1},"edges":[{"node":{"entity_type":"Intrusion-Set","representative":{"main":"Velvet Harbor"}}}]}}}
```

The platform first runs the `dynamic` filter, then applies `regardingOf` to the IDs it found. It keeps the first 5,000 of them (`elasticsearch:max_pagination_result`): with a broader `dynamic` filter, results can be missing, and the filter menu warns about it.

### dynamicFrom and dynamicTo

On relationships, `dynamicFrom` and `dynamicTo` apply a filter group to the source or the target. Their `values` hold the filter group itself. The `uses` relationships whose source is a malware:

```json
{"mode":"and","filters":[
  {"key":"relationship_type","values":["uses"]},
  {"key":"dynamicFrom","values":[{"mode":"and","filters":[{"key":"entity_type","values":["Malware"]}],"filterGroups":[]}]}],
 "filterGroups":[]}
```

```json
{"data":{"stixCoreRelationships":{"pageInfo":{"globalCount":2},"edges":[
  {"node":{"relationship_type":"uses","from":{"representative":{"main":"Velvet Loader"}},"to":{"representative":{"main":"[T1059.001] PowerShell"}}}},
  {"node":{"relationship_type":"uses","from":{"representative":{"main":"Harbor Stealer"}},"to":{"representative":{"main":"[T1071.001] Web Protocols"}}}}]}}}
```

## Filters on stream events

Live streams, triggers and playbooks do not query the database: they test each event against the filter, on the STIX object the event carries. Only these keys work there:

`entity_type`, `ids`, `representative`, `description`, `confidence`, `createdBy`, `creator_id`, `objectLabel`, `objectMarking`, `objectAssignee`, `objectParticipant`, `objectAuthorized`, `objects`, `workflow_id`, `revoked`, `priority`, `severity`, `indicator_types`, `pattern`, `pattern_type`, `x_opencti_score`, `x_opencti_detection`, `x_opencti_main_observable_type`, `report_types`, `response_types`, `information_types`, `takedown_types`, `note_types`, `incident_type`, `x_opencti_cisa_kev`, `x_opencti_epss_score`, `x_opencti_epss_percentile`, `x_opencti_cvss_base_score`, `x_opencti_cvss_base_severity`, `x_opencti_ssvc_exploitation`, `x_opencti_ssvc_automatable`, `x_opencti_ssvc_technical_impact`, `pir_score`, `fromId`, `fromTypes`, `toId`, `toTypes`, `connectedToId`, `connectedToId_sideEvents`.

::::only{opencti=">=7.261119"}
`x_opencti_closing_reason` works there too, and so do the custom fields, by their technical name, such as `x_opencti_cf_member_impact`.
::::

Saving a live stream, a trigger or a playbook with another key fails with `Stix filtering is not compatible with the provided filter key`, and the error data lists the keys allowed. `regardingOf` and the dynamic keys do not work on events.

::::only{opencti=">=7.260811"}
On events, the keys of attributes and relationships also accept `has_changed` and `not_has_changed`, without values. `objectAuthorized` refuses them, `pir_score` ignores them, and `connectedToId` and `connectedToId_sideEvents` never match with them. On an update, `has_changed` matches when the event changed the key; on a creation, when the key has a value. For example, `x_opencti_score` `has_changed` keeps the updates that change the score of an indicator and leaves out a label added to it.
::::

## OpenAEV filters

The search endpoints of the OpenAEV API, such as `POST /api/scenarios/search`, take a filter group in the `filterGroup` field of their body, beside the pagination:

```json
{
  "page": 0,
  "size": 20,
  "textSearch": "",
  "filterGroup": {
    "mode": "and",
    "filters": [
      {"key": "scenario_name", "operator": "contains", "mode": "or", "values": ["Velvet Harbor"]}
    ]
  }
}
```

The OpenAEV format differs from the OpenCTI one:

- **No nested groups.** A filter group has `mode` and `filters` only.
- **`key` is a string.**
- **The operators** are `eq`, `not_eq`, `contains`, `not_contains`, `starts_with`, `not_starts_with`, `ends_with`, `not_ends_with`, `gt`, `gte`, `lt`, `lte`, `empty` and `not_empty`. `empty` and `not_empty` take the place of `nil` and `not_nil`.
- **`ends_with` and `not_ends_with`** work in the API only: the filter menu of the interface does not offer them. Like `starts_with`, they ignore case, and `not_ends_with` with several values keeps the objects that end with none of them. `asset_name` `ends_with` `.example.com` keeps the assets whose name ends with that domain.
- **`size`** is 1,000 at most.

The keys are the attributes of the object the endpoint searches, such as `scenario_name` or `scenario_tags`. See [REST API](/develop/apis/rest/) for the endpoints.

## What's next

- [Filter keys](/reference/opencti/filter-keys/): every key and operator of this version of OpenCTI.
- [Query OpenCTI with GraphQL](/develop/apis/graphql/): pass filters to list queries.
- [Data streaming](/develop/apis/streams/): how a live stream applies its filters to events.
- [Share knowledge with your security tools](/opencti/use/share/feeds/): build feed and stream filters in the interface.
