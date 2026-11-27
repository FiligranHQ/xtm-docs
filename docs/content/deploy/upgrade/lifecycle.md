---
title: Versioning and LTS
description: How XTM releases are numbered, how the OpenCTI Long-Term Support (LTS) program works, and which LTS lines are supported until when.
doc_type: explanation
products:
  - suite
personas: [platform-engineer, admin]
owner: docs-platform
last_reviewed: 2026-10-31
migrated_from:
  - opencti:/administration/product-life-cycle/
---

XTM releases are numbered by date and published when they are ready. OpenCTI also offers Long-Term Support (LTS) lines: a release that receives only critical and security fixes for one year, for organizations that cannot upgrade often. This page explains the version numbers, the LTS program and how to choose between the two paths.

::diagram{id=deploy-upgrade-lifecycle}

## Version numbers

OpenCTI, OpenAEV, XTM One and XTM Composer use the same date-based format, `x.dddddd.z`: OpenCTI 7.261030.1, OpenAEV 3.261029.0, XTM One 1.261030.0, XTM Composer 3.261021.1.

| Format | Release | Parts |
| --- | --- | --- |
| `x.dddddd.z` | Regular release | `x` major version, `dddddd` release date (YYMMDD), `z` sequence number for releases of the same day |
| `x.dddddd.z-lts.n` | OpenCTI LTS release | `x` major version, `dddddd` date the LTS line was branched (YYMMDD), `z` sequence number for releases of the same day, `n` hotfix number of the line, from 1 |

For example, `7.260811.0-lts.1` is the first release of the OpenCTI LTS line branched on 11 August 2026, which the documentation calls `7.260811-lts`. `7.261030.1` is the second OpenCTI release of 30 October 2026.

Regular releases come out when they are ready, from daily to weekly. A security fix or a bug fix is not held back for a release date, and new features arrive continuously.

## The LTS program

An LTS line evolves slowly: it receives only critical bug fixes and security fixes, never new features. LTS is an option, not a required path.

| Rule | Value |
| --- | --- |
| A new LTS line | About every 6 months |
| Stabilization before release | 1 month after the line is branched |
| Support | 1 year from the release of the line |
| Overlap with the next line | 6 months, to validate the next line while the current one is supported |
| Fixes | Critical bugs and security issues, released when needed |

A bug is critical when it makes a feature unusable for its users: a core feature or workflow fails, data is lost or corrupted, performance drops enough to block normal work, or a regression appears.

### Supported LTS lines

| Line | Latest release documented | Supported until |
| --- | --- | --- |
| 7.260811-lts | 7.260811.0-lts.1 | September 2027 |
| 7.260309-lts | 7.260309.0-lts.8 | April 2027 |

The documentation is built for the latest release and for each supported LTS line: choose your version in the header to read the pages as they apply to it.

### The life of an LTS line

The 7.260309-lts line follows the rules above:

| When | What happens |
| --- | --- |
| 9 March 2026 | 7.260309-lts is branched from the latest release |
| March to April 2026 | One month of stabilization, then the first release |
| 11 August 2026 | The next line, 7.260811-lts, is branched |
| September 2026 | 7.260811-lts is released: the two lines overlap |
| April 2027 | 7.260309-lts reaches its end of support |

## LTS or the latest release

| Choose | When your organization |
| --- | --- |
| An LTS line | Tests and validates every update at length, works under strict change management (regulated industries), wants few functional changes in production, or has little time for frequent upgrades |
| The latest release | Wants new features and fixes as soon as they exist, can deploy and test often, has a flexible change process, or runs a development or test environment |

## Get an LTS release

LTS releases are container images, published like regular releases, for example `opencti/platform:7.260811.0-lts.1`. Running one needs an LTS license key:

1. Contact your Account Executive or Customer Success Manager.
2. Request an LTS license for your deployment.
3. Set the license key and the image tags you receive in your deployment. With xtm-docker, see [Stay on a version or an LTS line](/deploy/upgrade/#stay-on-a-version-or-an-lts-line).

Until an LTS license is applied, an LTS release opens on the **OpenCTI LTS license agreement** dialog and refuses the other API requests, with the error `LTS_REQUIRED_ACTIVATION`, connectors included: a standard or trial license does not open it. Plan the license before you switch the images. See [Long-term support versions](/start/editions/#long-term-support-versions).

## What's next

- [Upgrade](/deploy/upgrade/), to move to a new release or a new LTS line.
- [Breaking changes](/deploy/upgrade/breaking-changes/), what each version asks of you.
- [Compatibility matrix](/deploy/upgrade/compatibility/), the versions tested together.
