---
title: Set default asset groups by scenario tag
description: Link a tag to asset groups so that every inject of a tagged scenario or simulation targets them, and set the groups of the rules that scenarios from OpenCTI use.
doc_type: how-to
products:
  - openaev
personas: [admin, validation-engineer]
owner: docs-openaev
last_reviewed: 2026-11-07
migrated_from:
  - openaev:/usage/default-asset-rules/
---

A default asset rule links a tag to asset groups. In every scenario or simulation that carries the tag, new injects target those asset groups without anyone picking them. It saves choosing the same groups on every inject, and it is how the scenarios OpenCTI generates find their targets: they arrive with known tags, and the rules of those tags decide which endpoints they run on. The rules are under :menu[Settings > Customization > Default asset rules].

::diagram{id=openaev-asset-rules-flow}

## Before you start

- Your role has **Access tenant settings** to see the rules, **Manage tenant settings** to create and change them, and **Delete tenant settings** to delete them.
- The asset groups exist: see [Add the assets your simulations target](/openaev/use/targets/assets/).

## How a rule applies

A rule acts at two moments, and only on injects whose type can target asset groups:

- **When an inject is created** in a scenario or a simulation, the asset groups of the rules matching its tags are added to the ones the inject already has. A group is never added twice.
- **When a tag is added** to a scenario or a simulation, and a rule matches it, OpenAEV asks **Asset rule detected**: click **Yes** to add the rule's asset groups to every existing inject, **No** to leave them as they are. New injects get the groups either way.

A rule only adds. Removing a tag from a scenario, or a group from a rule, leaves the injects as they are: remove the groups from the injects yourself.

## Create a rule

:::steps
1. Open :menu[Settings > Customization > Default asset rules] and click **Create**. The **Create an asset rule** panel opens.
2. Under **Tag**, choose the tag in **Select a Tag**.
3. Under **Asset groups**, choose the groups the tag must target.
4. Click **Create**.
:::

The list shows each rule with its **Tag** and its **Asset Groups**. To change the groups of a rule, open the menu at the end of its line and choose **Update**; to remove it, **Delete**.

::screenshot{id=openaev-asset-rules alt="The Default asset rules: security coverage: windows sends to Front desk workstations, the shipped opencti and macOS and Linux rules keep their groups, and velvet-harbor sends to Front desk workstations and one more group, Booking servers."}

## The rules of scenarios from OpenCTI

Four rules exist in every tenant and cannot be deleted, nor their tag changed. You only choose their asset groups:

| Tag | Scenarios that carry it | Asset groups on a new tenant |
| --- | --- | --- |
| `opencti` | Scenarios and injects created from OpenCTI | **All endpoints**, when the starter pack is on |
| `security coverage: windows` | Scenarios generated from an OpenCTI security coverage that applies to Windows | **All Windows x86_64** and **All Windows arm64** |
| `security coverage: linux` | The same, for Linux | **All Linux x86_64** and **All Linux arm64** |
| `security coverage: macos` | The same, for macOS | **All MacOS x86_64** and **All MacOS arm64** |

These groups select hosts by platform and architecture, so every Windows, Linux or macOS host is in one of them. To keep the scenarios from OpenCTI away from part of your estate, such as production servers, point these rules at narrower groups. The starter pack, which also creates **All endpoints**, is on by default (`openaev.starterpack.enabled`, in the [starter pack settings](/reference/openaev/configuration/#openaev-starterpack)). See [Turn an OpenCTI security coverage into a scenario](/openaev/use/scenarios/from-opencti/).

:::example[Keep OpenCTI scenarios on the front desks]
Grand Meridian Hotels lets OpenCTI generate a scenario each time a security coverage of Velvet Harbor changes, but wants those scenarios to run only on the front-desk workstations, never on the booking database server. The administrator opens :menu[Settings > Customization > Default asset rules]:

| Rule | **Asset groups** |
| --- | --- |
| `opencti` | **Front desk workstations**, instead of **All endpoints** |
| `security coverage: windows` | **Front desk workstations**, instead of **All Windows x86_64** and **All Windows arm64** |

Then a rule of their own, **Tag** `velvet-harbor`, **Asset groups** **Front desk workstations** and **Booking servers**, for the scenarios the validation engineer tags by hand. When the engineer adds `velvet-harbor` to the existing scenario `Velvet Harbor replay`, **Asset rule detected** asks; **Yes** adds both groups to its three injects that run on endpoints, and leaves the email inject, which targets a team, as it is.
:::

## What's next

- [Add the assets your simulations target](/openaev/use/targets/assets/): build the asset groups the rules point at.
- [Turn an OpenCTI security coverage into a scenario](/openaev/use/scenarios/from-opencti/): the scenarios the protected rules serve.
- [How injects reach assets and people](/openaev/use/targets/): how an inject gets its targets.
