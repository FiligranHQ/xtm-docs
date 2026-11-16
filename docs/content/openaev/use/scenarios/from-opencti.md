---
title: Turn an OpenCTI security coverage into a scenario
description: Let OpenAEV build a scenario from a threat's security coverage in OpenCTI, aim it at the right assets, fill its gaps, and send the scores of every run back to OpenCTI.
doc_type: how-to
products:
  - openaev
  - opencti
personas: [validation-engineer, soc-manager]
owner: docs-openaev
last_reviewed: 2026-11-07
migrated_from:
  - openaev:/usage/scenario/security-coverage/
---

When the SOC creates an automated security coverage on a threat in OpenCTI, OpenAEV receives it and builds a scenario on its own: one inject per technique of the threat, run on the assets of the platforms the coverage names, at the recurrence it sets. After each simulation, the scores go back to the coverage in OpenCTI. This page explains what OpenAEV builds, how to aim it at the right endpoints, how to fill the techniques it cannot test, and where the results go. The coverage itself is created in OpenCTI: see [Measure your defenses against a threat](/opencti/use/analyze/security-coverage/).

## Before you start

- An administrator has connected OpenAEV to OpenCTI, so that the **OpenAEV Coverage** connector shows in OpenCTI. See [Connect OpenAEV to OpenCTI to test security coverages](/openaev/use/coverage-enrichment/).
- OpenAEV agents run on the endpoints to test, and they belong to asset groups. See [OpenAEV agents](/openaev/use/agents/) and [Assets](/openaev/use/targets/assets/).
- Your role in OpenAEV has **Access assessment** to read the scenario, **Manage assessment** to change its injects, **Manage tenant settings** to change the default asset rules, and **Manage threat arsenal** to add a payload.
- A security platform reports to OpenAEV through a collector, such as your EDR, so that detection and prevention can be measured. See [Collectors](/openaev/use/collectors/).

## What OpenAEV builds

::diagram{id=openaev-scenarios-from-opencti-injects}

OpenCTI sends the coverage, the covered threat and its entities as a Structured Threat Information Expression (STIX) bundle. OpenAEV creates one scenario per coverage, or updates the one it already built:

| Property | Value |
| --- | --- |
| **Name** and **Description** | Those of the coverage, such as `Coverage of Velvet Harbor` |
| **Category**, **Main focus**, **Severity** | **Attack Scenario**, **Incident Response**, **High** |
| **Tags** | One per platform of the coverage's **Platform(s) affinity**: `security coverage: windows`, `security coverage: linux`, `security coverage: macos` |
| **Threat intelligence** | **Open in OpenCTI**, on the scenario's **Overview**, opens the covered threat |
| Schedule | The coverage's recurrence, from 2 minutes after the scenario is created, until the end of its **Duration** |

The injects come from the entities the threat holds:

| In the coverage | Inject OpenAEV creates |
| --- | --- |
| An attack pattern that OpenAEV knows, with a payload for the platforms to test | One inject per technique, named `[T1059.001] PowerShell - ` followed by the payload |
| An attack pattern with no payload for a platform and architecture to test | A placeholder, disabled: `[T1566.001] Placeholder - Windows x86_64` |
| An attack pattern OpenAEV does not know | A placeholder, disabled: `[T1566.001] Placeholder` |
| A domain name indicator | `Resolve DNS` followed by the domain, run on the endpoints |
| A file (artifact) | An inject that drops the file on the endpoints; a placeholder when OpenAEV cannot download it from OpenCTI |
| A vulnerability | An inject from the actions OpenAEV has for that CVE; a placeholder otherwise |

OpenAEV matches an attack pattern by its MITRE ATT&CK ID or its name. A parent technique, such as T1059, also matches the payloads of its sub-techniques.

### Which endpoints are tested

The platform tags of the scenario decide the targets. Each tag has a rule in :menu[Settings > Customization > Default asset rules] that lists asset groups, and every inject runs on the asset groups of the scenario's tags. By default, the rule of `security coverage: windows` lists the dynamic groups **All Windows x86_64** and **All Windows arm64**: every Windows endpoint of the tenant.

OpenAEV then looks for a payload for each technique and each platform and architecture of those endpoints. When the endpoints run Windows on x86_64 only, one payload for Windows x86_64 is enough; when they mix architectures, a payload that covers them all is preferred, otherwise one inject per platform and architecture is created. A combination left without a payload becomes a placeholder.

A coverage without **Platform(s) affinity** gives the scenario no tag, so its injects have no target: set the platforms in OpenCTI.

::screenshot{id=openaev-scenarios-from-opencti-overview alt="The Overview of the scenario Coverage of Velvet Harbor that OpenAEV built: Attack Scenario, high, Scheduled, 17 injects and 2 asset groups, the coverage's description, the Endpoint type affinity and the Linux and Windows platforms, its security coverage tags, the Open in OpenCTI button, and its first scheduled simulation."}

::screenshot{id=openaev-scenarios-from-opencti-injects alt="The Injects tab of the same scenario: four enabled injects, PowerShell, two Data from Local System and Web Protocols, each on its platform, among the disabled placeholders of the techniques no payload runs, such as T1566.001 for Windows x86_64 and Linux arm64."}

## Aim the scenario and fill its gaps

Set the targets before the coverage is created: an inject keeps the asset groups it was built with, and a later change of the rule only reaches the injects built after it.

:::steps

1. In OpenAEV, open :menu[Settings > Customization > Default asset rules]. In the row of the platform tag, such as `security coverage: windows`, open the menu and choose **Update**.
2. Under **Asset groups**, keep only the groups to test, then click **Update**. The rule applies to every scenario generated from OpenCTI for that platform.
3. In OpenCTI, create the automated security coverage on the threat. Within a few seconds, the scenario appears in :menu[Scenarios] in OpenAEV, and **Go to OpenAEV** in OpenCTI opens it.
4. Open the scenario, then its **Injects** tab, and check the targets of each inject. To change them for this scenario only, edit the inject.
5. For each placeholder, read its title. With a platform and architecture, such as `[T1566.001] Placeholder - Windows x86_64`, no payload runs the technique there: search the :menu[Threat Arsenal] by the MITRE ATT&CK ID, and add the payload that is missing, from a payload collector such as Atomic Red Team or by hand. See [Threat arsenal](/openaev/use/threat-arsenal/). With the ID alone, OpenAEV does not know the attack pattern: add it in :menu[Settings > Taxonomies > Attack patterns], or let a collector import MITRE ATT&CK.
6. In OpenCTI, open the coverage, click **Enrichment**, then **Refresh the knowledge using this connector** on the OpenAEV connector. OpenAEV removes the placeholders, builds the injects that are now possible, and keeps the injects it had already built, with their targets.

:::

The first simulation starts 2 minutes after the scenario is created. If you need more time to adjust it, click **Stop** in the header of the scenario to remove the schedule, then set it again with **Scheduling**.

## Results back in OpenCTI

Each simulation of the scenario runs like any other: see [Run a simulation and read its results](/openaev/use/simulations/). When a simulation has **Finished**, every expectation answered or expired, and no later simulation of the scenario exists, OpenAEV sends its results to the coverage a minute or two later:

- a score per metric, prevention, detection, vulnerability and human response: the share of expectations met, from 0 to 100;
- a score for each attack pattern, domain name and file the injects tested, linked to the coverage by a **has covered** relationship;
- a score per security platform that reported, such as your EDR, so OpenCTI shows which tool caught what;
- the validity dates: from the start of the simulation to the expiry of its last expectation.

A technique with no score in OpenCTI had no inject that ran: a placeholder, or an inject disabled by hand. The link in OpenCTI opens the scenario in OpenAEV.

When the **Duration** of the coverage ends, the scenario stops running. A refresh of the coverage in OpenCTI schedules it again.

:::example[Replay Velvet Harbor on the hotels of Grand Meridian]
The SOC of Grand Meridian Hotels wants to know, every week, whether its defenses stop the techniques Velvet Harbor uses, on the front-desk workstations (Windows) and on the booking servers (Linux).

| Who, where | What they set or see |
| --- | --- |
| Lionel, SOC manager, the intrusion set Velvet Harbor in OpenCTI | An automated security coverage, `Coverage of Velvet Harbor`, every week for 30 days, **Type affinity** **Endpoint**, **Platform(s) affinity** `windows` and `linux` |
| OpenAEV, **Injects** tab of `Coverage of Velvet Harbor` | 17 injects. Four are enabled, on the techniques the threat arsenal of Grand Meridian covers: `[T1059.001] PowerShell - ...`, `[T1071.001] Web Protocols - ...`, and two `[T1005] Data from Local System - ...`, the guest records search on the front desks and the export on the booking database. Thirteen are disabled placeholders, such as `[T1566.001] Placeholder - Windows x86_64` for the attachment no payload opens yet, or `[T1087] Placeholder` for a technique OpenAEV does not know |
| Validation engineer, Threat Arsenal | A payload for T1566.001 on Windows x86_64 that writes the booking request's attachment to disk and opens it |
| Lionel, the coverage in OpenCTI | **Refresh the knowledge using this connector** |

At the refresh, the placeholder of T1566.001 for Windows becomes `[T1566.001] Spearphishing Attachment - ...`. Each weekly simulation then sends its scores back to the coverage in OpenCTI, technique by technique, and the placeholders left show what the threat arsenal still misses.
:::

:::xtm-one[Ask XTM One about the results]
With XTM One connected and the Enterprise Edition, **Ask Ariane** in OpenAEV reads the scenario and its simulations as you: ask it which techniques of Velvet Harbor Meridian EDR did not prevent, or to draft a readout of the last run for the CISO. See [XTM One in OpenAEV](/xtm-one/use/in-openaev/).
:::

## When the scenario does not appear

- **The connector is missing in OpenCTI**: the **Automated using enrichment** choice stays greyed. Check the connection: see [Connect OpenAEV to OpenCTI to test security coverages](/openaev/use/coverage-enrichment/#check-the-connection).
- **The coverage did not change**: OpenAEV refuses a bundle identical to the last one, and a bundle older than the last it processed. The **Enrichment** panel of the coverage in OpenCTI shows the request and its message.
- **The injects have no target**: the coverage has no **Platform(s) affinity**, or the rule of its platform tag lists no asset group.

## What's next

- [Run a simulation and read its results](/openaev/use/simulations/), to follow the runs of the scenario.
- [Measure your defenses against a threat](/opencti/use/analyze/security-coverage/), to read the scores in OpenCTI.
- [Check your defenses against the threats you track](/workflows/threat-informed-validation/), the whole loop across the suite.
- [Default asset rules](/openaev/configure/asset-rules/), for the rules of every platform tag.
