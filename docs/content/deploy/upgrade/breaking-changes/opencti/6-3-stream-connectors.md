---
title: Stream connectors data import
description: The two-way OpenCTI stream connectors for Tanium, HarfangLab and Microsoft Sentinel, split in 6.3 into an import connector and a stream connector each.
doc_type: how-to
products:
  - opencti
personas: [platform-engineer]
owner: docs-platform
last_reviewed: 2026-10-31
migrated_from:
  - opencti:/deployment/breaking-changes/6.3-removing-some-connectors/
---

Three OpenCTI stream connectors used to work both ways: they sent OpenCTI data to a security tool and imported its incidents. From OpenCTI 6.3, each is split into an external import connector, which brings data into OpenCTI, and a stream connector, which sends OpenCTI data out. The old connectors were removed in 6.6. This page lists the replacements.

::diagram{id=breaking-opencti-6-3-stream-connectors}

| | Version |
| --- | --- |
| Deprecated in | OpenCTI 6.3 |
| Removed in | OpenCTI 6.6 |

## Replaced connectors

| Removed connector | External import connector | Stream connector |
| --- | --- | --- |
| `tanium` (Tanium) | `tanium-incidents` | `tanium-intel` |
| `harfang-lab` (HarfangLab) | `harfang-lab-incidents` | `harfang-lab-intel` |
| `sentinel` (Microsoft Sentinel) | `sentinel-incidents` | `sentinel-intel` |

## What to do

:::steps
1. Deploy the stream connector that replaces yours, to keep sending OpenCTI data to the tool.
2. Deploy the external import connector too if you imported incidents from the tool.
3. Remove the old connector.
:::

## What's next

- [Import with connectors](/opencti/use/ingest/connectors/), how import connectors work.
- [How integrations run](/deploy/integrations/how-they-run/), to deploy a connector.
- [Breaking changes](/deploy/upgrade/breaking-changes/), the other changes per version.
