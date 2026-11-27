---
title: New filter API
description: Convert the filters of your scripts, connectors and API calls to the recursive filter format that OpenCTI 5.12 introduced, with the renamed filter keys.
doc_type: how-to
products:
  - opencti
personas: [platform-engineer, developer]
owner: docs-platform
last_reviewed: 2026-10-31
migrated_from:
  - opencti:/deployment/breaking-changes/5.12-filters/
---

OpenCTI 5.12 replaced the [filter format](/develop/apis/filters/) of its API with a recursive one that combines filters with `and` and `or`, nests groups, and tests empty fields. OpenCTI converts the filters it stores, but not those of your own scripts, connectors and API calls. This page shows how to convert them.

::diagram{id=breaking-opencti-5-12-filters}

| | Version |
| --- | --- |
| Changed in | OpenCTI 5.12 |

## Why the format changed

The old format could not express complex combinations: no filters within filters, one fixed combination (`and` between filters, `or` between the values of a filter), and the operator was part of the key, as in `entity_type_not_eq`. The frontend and backend formats differed, and the keys were static lists maintained by hand. The [Filigran blog post on advanced filtering](https://blog.filigran.io/introducing-advanced-filtering-possibilities-in-opencti-552147565faf) tells the full story.

```ts
// OpenCTI < 5.12: a flat list
type Filter = {
  key: string,       // one of the filter keys of the entity type
  values: string[],
  operator: string,
  filterMode: 'and' | 'or',
}
```

The new format is a recursive group of filters:

```ts
// OpenCTI >= 5.12
type FilterGroup = {
  mode: 'and' | 'or'
  filters: Filter[]
  filterGroups: FilterGroup[] // nested groups
}

type Filter = {
  key: string[]
  values: string[]
  operator: 'eq' | 'not_eq' | 'gt' // and more
  mode: 'and' | 'or',
}

// Reports and RFIs, not marked TLP:RED, with labelX or no label
const filters = {
  mode: 'and',
  filters: [
    { key: 'entity_type', values: ['Report', 'Case-Rfi'], operator: 'eq', mode: 'or' },
    { key: 'objectMarking', values: ['<id-for-TLP:RED>'], operator: 'not_eq', mode: 'or' },
  ],
  filterGroups: [{
    mode: 'or',
    filters: [
      { key: 'objectLabel', values: ['<id-for-labelX>'], operator: 'eq', mode: 'or' },
      { key: 'objectLabel', values: [], operator: 'nil', mode: 'or' },
    ],
    filterGroups: [],
  }],
};
```

## What changed

- Each filter has its own operator and mode, and groups nest through `filterGroups`.
- Keys are strings, no longer a fixed list per entity type.
- `values` can no longer contain `null`: the new operators `nil` and `not_nil` test whether a field is empty.
- Some keys were renamed to match the entity schema: see [Renamed filter keys](#renamed-filter-keys).

## What migrates on its own

| Filters | Migrated |
| --- | --- |
| [Stored filters](/develop/apis/filters/), saved in the database | Yes, when OpenCTI 5.12 first starts |
| Filters in your connectors, GraphQL queries and Python scripts | No: convert them below |
| Filters of the views, kept in the URL and the browser's local storage | No: they are lost. Build them again in the interface |

A URL with filters in the old format shows a warning, and OpenCTI removes its filters. Build the filters again and share the new URL.

## Convert a filter

:::steps
1. Rename the `key` if it is in the [renamed filter keys](#renamed-filter-keys).
2. Rename `filterMode` to `mode`.
3. If `values` contains `null`, split the filter in two, as shown below.
4. Put the converted filters in a group: `{ mode: 'and', filters: [ ... ], filterGroups: [] }`.
:::

```ts
const oldFilter = {
  key: 'old_key',
  values: ['value1', 'value2'],
  operator: 'XX',
  filterMode: 'XX',
}

const convertedFilter = {
  key: 'converted_key_if_necessary',
  values: ['value1', 'value2'],
  operator: 'XX',
  mode: 'XX',
}

const newFilters = {
  mode: 'and',
  filters: [convertedFilter1, convertedFilter2], // every converted filter
  filterGroups: [],
}
```

### Filters with a null value

Split a filter whose `values` contain `null` in two:

- A filter for `null`, with `values: []` and the operator `nil` (if it was `eq`) or `not_nil` (if it was `not_eq`).
- A filter with the other values.

```ts
// Must have a label, and not Label1 or Label2
const oldFilter = {
  key: 'labelledBy',
  values: [null, 'id-for-Label1', 'id-for-Label2'],
  operator: 'not_eq',
  filterMode: 'and',
}

const newFilters = {
  mode: 'and',
  filters: [
    { key: 'objectLabel', values: ['id-for-Label1', 'id-for-Label2'], operator: 'not_eq', mode: 'and' },
    { key: 'objectLabel', values: [], operator: 'not_nil', mode: 'and' },
  ],
  filterGroups: [],
}
```

:::tip[Nest a group to keep the logic]
An `eq` filter with a `null` value, combined in `and` mode with other filters, needs a nested group to mean the same thing: see the second example below.
:::

## Renamed filter keys

| Key before 5.12 | Key from 5.12 |
| --- | --- |
| `labelledBy` | `objectLabel` |
| `markedBy` | `objectMarking` |
| `objectContains` | `objects` |
| `killChainPhase` | `killChainPhases` |
| `assigneeTo` | `objectAssignee` |
| `participant` | `objectParticipant` |
| `creator` | `creator_id` |
| `hasExternalReference` | `externalReferences` |
| `hashes_MD5` | `hashes.MD5` |
| `hashes_SHA1` | `hashes.SHA-1` |
| `hashes_SHA256` | `hashes.SHA-256` |
| `hashes_SHA512` | `hashes.SHA-512` |

## Examples

### Reports with label1 or label2

`(entity_type = Report) AND (label = label1 OR label2)`

```ts
const oldBackendFilter = [
  { key: 'entity_type', values: ['Report'], operator: 'eq', filterMode: 'or' },
  { key: 'labelledBy', values: [label1_id, label2_id], operator: 'eq', filterMode: 'or' },
];

const newFilters = {
  mode: 'and',
  filters: [
    { key: 'entity_type', values: ['Report'], operator: 'eq', mode: 'or' },
    { key: 'objectLabel', values: [label1_id, label2_id], operator: 'eq', mode: 'or' },
  ],
  filterGroups: [],
};
```

### Reports with label1, label2 or no label

`(entity_type = Report) AND (label = "No label" OR label1 OR label2)`

```ts
const oldBackendFilter = [
  { key: 'labelledBy', values: [label1_id, label2_id, null], operator: 'eq', filterMode: 'or' },
  { key: 'entity_type', values: ['Report'], operator: 'eq', filterMode: 'or' },
];

const newFilters = {
  mode: 'and',
  filters: [
    { key: 'entity_type', values: ['Report'], operator: 'eq', mode: 'or' },
  ],
  // this combination of mode and operator needs a nested group
  filterGroups: [
    {
      mode: 'or',
      filters: [
        { key: 'objectLabel', values: [label1_id, label2_id], operator: 'eq', mode: 'or' },
        { key: 'objectLabel', values: [], operator: 'nil', mode: 'or' }, // values and mode do not matter
      ],
      filterGroups: [],
    },
  ],
};
```

## What's next

- [Filters](/develop/apis/filters/), the filter format and its operators.
- [Breaking changes](/deploy/upgrade/breaking-changes/), the other changes per version.
