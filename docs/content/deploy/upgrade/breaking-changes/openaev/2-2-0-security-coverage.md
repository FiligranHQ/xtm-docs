---
title: Scenario generation from OpenCTI security coverage
description: OpenAEV 2.2.0 works only with OpenCTI 7 and the reverse. Upgrade both together, or move to OpenAEV 2.2.1, which works with older OpenCTI again.
doc_type: how-to
products:
  - openaev
  - opencti
personas: [platform-engineer]
owner: docs-platform
last_reviewed: 2026-10-31
migrated_from:
  - openaev:/deployment/breaking-changes/2.2.0-opencti-security-coverage/
---

OpenCTI 7 changed the API that OpenCTI and OpenAEV use to work together, for example to generate scenarios from the security coverage of OpenCTI. As a result, OpenAEV 2.2.0 works only with OpenCTI 7, and OpenCTI 7 only with OpenAEV 2.2.0 or later. This page says how to upgrade a connected pair without breaking the connection.

::diagram{id=breaking-openaev-2-2-0-security-coverage}

| | Version |
| --- | --- |
| Introduced in | OpenAEV 2.2.0 |
| Resolved in | OpenAEV 2.2.1 |

## Which versions work together

| OpenAEV | OpenCTI 6 | OpenCTI 7 |
| --- | --- | --- |
| Earlier than 2.2.0 | Yes | No |
| 2.2.0 | No | Yes |
| 2.2.1 and later | Yes | Yes |

## What to do

- Upgrade OpenAEV to 2.2.1 or later rather than 2.2.0: it works with OpenCTI 6 and 7.
- If you must run OpenAEV 2.2.0, upgrade OpenCTI to version 7 at the same time. Upgrading one without the other breaks the connection between them.
- Before you upgrade OpenCTI to version 7, upgrade OpenAEV to 2.2.1 or later.

With xtm-docker, one `docker compose pull` moves every module to its latest release together: see [Upgrade](/deploy/upgrade/).

## What's next

- [Threat-informed validation](/workflows/threat-informed-validation/), the workflow that generates scenarios from OpenCTI.
- [Compatibility matrix](/deploy/upgrade/compatibility/), the other constraints between modules.
- [Breaking changes](/deploy/upgrade/breaking-changes/), the other changes per version.
