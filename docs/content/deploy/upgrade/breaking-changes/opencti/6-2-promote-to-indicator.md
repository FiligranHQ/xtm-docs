---
title: Promote to indicator
description: The OpenCTI API calls that promote an observable now return the created Indicator, through a new GraphQL mutation and a new Python client method.
doc_type: how-to
products:
  - opencti
personas: [platform-engineer, developer]
owner: docs-platform
last_reviewed: 2026-10-31
migrated_from:
  - opencti:/deployment/breaking-changes/6.2-promote-to-indicator/
---

From OpenCTI 6.2, the API calls that promote an observable to an Indicator return the Indicator they create, instead of the observable. The old calls were removed in 6.8. This page lists the replacements, for scripts that call the GraphQL API or the Python client.

::diagram{id=breaking-opencti-6-2-promote-to-indicator}

| | Version |
| --- | --- |
| Deprecated in | OpenCTI 6.2 |
| Removed in | OpenCTI 6.8 |

## Replaced calls

| API | Removed | Replaced by |
| --- | --- | --- |
| GraphQL | `StixCyberObservableEditMutations.promote` | `StixCyberObservableEditMutations.promoteToIndicator` |
| Python client | `client.stix_cyber_observable.promote_to_indicator` | `client.stix_cyber_observable.promote_to_indicator_v2` |

## What to do

Call the new mutation or method in your scripts. If a script uses the value it returns, expect the new Indicator instead of the promoted observable, and adapt the fields it reads.

## What's next

- [Script OpenCTI in Python](/develop/build/python-client/), the methods of the client.
- [Breaking changes](/deploy/upgrade/breaking-changes/), the other changes per version.
