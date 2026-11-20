---
title: Manage Elasticsearch indices
description: How OpenCTI organizes its Elasticsearch or OpenSearch indices, the rollover policy that keeps their shards small, and the permissions its user needs.
doc_type: how-to
products:
  - opencti
personas: [platform-engineer]
owner: docs-platform
last_reviewed: 2026-10-31
migrated_from:
  - opencti:/deployment/advanced/rollover/
---

OpenCTI stores its knowledge in about fifteen Elasticsearch or OpenSearch indices and rolls them over so that no shard grows too big. This page says which indices exist, which policy rolls them over and how to change it, how to check that it works, and which permissions the OpenCTI user needs on a secured cluster.

::diagram{id=deploy-operate-rollover}

## How OpenCTI uses its indices

OpenCTI writes to each index through an alias, and reads all its generations with a wildcard. At the first start, it creates each index with the suffix `-000001` (`elasticsearch:index_creation_pattern`), for example `opencti_history-000001`, and a lifecycle policy:

| Engine | Policy | Rolls over when |
| --- | --- | --- |
| Elasticsearch | `opencti-ilm-policy`, index lifecycle management (ILM) | A primary shard reaches 50 GB, or the index reaches 75,000,000 documents |
| OpenSearch | `opencti-ism-policy`, index state management (ISM) | The same thresholds |

At rollover, the engine creates the next generation, `-000002`, and moves the write alias to it. The `opencti` prefix is `elasticsearch:index_prefix`.

The indices are:

`opencti_deleted_objects`, `opencti_draft_objects`, `opencti_files`, `opencti_history`, `opencti_inferred_entities`, `opencti_inferred_relationships`, `opencti_internal_objects`, `opencti_internal_relationships`, `opencti_stix_core_relationships`, `opencti_stix_cyber_observable_relationships`, `opencti_stix_cyber_observables`, `opencti_stix_domain_objects`, `opencti_stix_meta_objects`, `opencti_stix_meta_relationships`, `opencti_stix_sighting_relationships`.

Four of them usually grow fastest:

| Index | Holds |
| --- | --- |
| `opencti_stix_meta_relationships` | The links between objects and their labels, markings, external references and authors |
| `opencti_history` | The history of every object |
| `opencti_stix_cyber_observables` | The observables |
| `opencti_stix_core_relationships` | The STIX relationships |

## Change the rollover thresholds

The thresholds are set in the OpenCTI configuration:

| Key | Environment variable | Default |
| --- | --- | --- |
| `elasticsearch:max_primary_shard_size` | `ELASTICSEARCH__MAX_PRIMARY_SHARD_SIZE` | `50gb` |
| `elasticsearch:max_docs` | `ELASTICSEARCH__MAX_DOCS` | `75000000` |

OpenCTI writes the policy when it creates its indices, so set them before the first start. On OpenSearch, an existing `opencti-ism-policy` is never overwritten: to change the thresholds afterwards, edit the policy in OpenSearch.

## Check the rollover

In xtm-docker, from the `xtm-docker` folder:

```bash
docker compose exec elasticsearch curl -s "http://localhost:9200/_cat/aliases/opencti_*?v&s=alias"
docker compose exec elasticsearch curl -s "http://localhost:9200/opencti_history-*/_ilm/explain?filter_path=indices.*.policy,indices.*.phase"
```

:::verify
The first command lists each alias, such as `opencti_history`, with its index generations. The second shows `"policy":"opencti-ilm-policy"` for every `opencti_history` index.
:::

## Permissions on a secured cluster

On a cluster with security on, the user OpenCTI connects with needs these permissions. They are given in the OpenSearch format.

- On the indices that match `opencti*` (or your `elasticsearch:index_prefix`): `indices_all`.
- On the cluster:
  - `cluster_composite_ops_ro`
  - `cluster_manage_index_templates`
  - `cluster:admin/ingest/pipeline/put`
  - `cluster:admin/opendistro/ism/policy/write`
  - `cluster:monitor/health`
  - `cluster:monitor/main`
  - `cluster:monitor/state`
  - `cluster:monitor/task/get`
  - `indices:admin/index_template/put`
  - `indices:data/read/scroll/clear`
  - `indices:data/read/scroll`
  - `indices:data/write/bulk`

:::warning[Keep the indices permissions in the cluster permissions]
The `indices:` permissions must also be in the cluster permissions, even if the index permissions already include them. Without them, OpenCTI can fail to start.
:::

## Platforms created before OpenCTI 5.9

A platform first started before OpenCTI 5.9 has no lifecycle policy on its indices. To add one:

:::steps
1. Upgrade the platform to the latest version.
2. Check that the policy and the index templates exist after the migration.
3. Split every shard larger than 50 GB into new indices, with the tools of your engine.
4. Point the write alias to the latest index, and check that the policy runs.
:::

Ask on the [Filigran community Slack](https://community.filigran.io) if you need help with this migration.

## What's next

- [Back up and restore](/deploy/operate/backup/), with Elasticsearch snapshots.
- [Scale: workers, platforms and clusters](/deploy/operate/scale/), for an Elasticsearch cluster.
- [OpenCTI configuration](/reference/opencti/configuration/), every `elasticsearch` key.
