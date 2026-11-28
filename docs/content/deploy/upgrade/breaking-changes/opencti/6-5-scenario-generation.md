---
title: Scenario generation placeholders
description: The OpenCTI GraphQL mutations that generate OpenAEV scenarios, replaced in 6.5 by versions that return inject placeholders and missing attack patterns.
doc_type: how-to
products:
  - opencti
  - openaev
personas: [platform-engineer, developer]
owner: docs-platform
last_reviewed: 2026-10-31
migrated_from:
  - opencti:/deployment/breaking-changes/6.5-generation-scenario-opencti-openbas-placeholders/
---

OpenCTI generates simulation scenarios in OpenAEV (formerly OpenBAS) from a container, a threat or a victim. In OpenCTI 6.5, the three GraphQL mutations that do it were replaced by new ones with a different signature and a more detailed response; the old ones were removed in 6.8. This page lists the replacements, for scripts that call the API.

::diagram{id=breaking-opencti-6-5-scenario-generation}

| | Version |
| --- | --- |
| Deprecated in | OpenCTI 6.5 |
| Removed in | OpenCTI 6.8 |

## Replaced mutations

| Removed | Replaced by |
| --- | --- |
| `obasContainerGenerateScenario` | `obasContainerGenerateScenarioWithInjectPlaceholders` |
| `obasThreatGenerateScenario` | `obasThreatGenerateScenarioWithInjectPlaceholders` |
| `obasVictimGenerateScenario` | `obasVictimGenerateScenarioWithInjectPlaceholders` |

The new mutations take a `SimulationConfig` object:

| Field | Meaning |
| --- | --- |
| `simulationType` | The type of simulation: technical or simulated |
| `selection` | The number of injects generated per attack pattern and platform |
| `interval` | The interval between injects, in minutes |

They return a `GenerationResponse` object:

| Field | Meaning |
| --- | --- |
| `urlResponse` | The URL of the generated scenario in OpenAEV |
| `attackPatternNotAvailableInOpenBAS` | The attack patterns the OpenAEV catalog does not cover |
| `hasInjectPlaceholders` | Whether OpenAEV created inject placeholders |

## What to do

On OpenCTI 6.5 to 6.x, if a script of yours calls one of the removed mutations, call its replacement with a `SimulationConfig`, and read the scenario URL from `urlResponse`.

The OpenCTI versions this documentation describes, 7.260309 and later, have neither the old nor the new mutations: the GraphQL API refuses a call to any of them. Remove these calls from your scripts when you upgrade to OpenCTI 7. In OpenCTI 7, OpenAEV generates the scenarios from the security coverage of OpenCTI: see [Scenario generation from OpenCTI security coverage](/deploy/upgrade/breaking-changes/openaev/2-2-0-security-coverage/).

## What's next

- [Threat-informed validation](/workflows/threat-informed-validation/), the workflow that generates scenarios from threat intelligence.
- [Breaking changes](/deploy/upgrade/breaking-changes/), the other changes per version.
