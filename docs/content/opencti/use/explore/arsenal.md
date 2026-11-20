---
title: Explore arsenal
description: Know a malware, a tool or a channel in the OpenCTI Arsenal menu - who uses it, against whom, how it works - and decide how urgent a vulnerability is.
sidebar_position: 9
doc_type: how-to
products:
  - opencti
personas: [cti-analyst]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/exploring-arsenal/
---

The **Arsenal** menu holds what threats use: their malware, the legitimate tools they turn against you, the channels they spread messages through, and the vulnerabilities they exploit. This page shows how to know a malware or a tool, what its page tells you about who uses it, and how to decide whether a vulnerability is urgent. How to read the tabs every entity page shares is explained once in [Read an entity page](/opencti/use/explore/#read-an-entity-page).

## Before you start

- Your role has the **Access knowledge** capability to read the arsenal.
- To create, change or link an object, your role also needs **Create / Update knowledge**; to delete one, **Delete knowledge**. See [Create knowledge manually](/opencti/use/ingest/manual/).

## How OpenCTI represents an arsenal

| Entry | Use it for | Linked to threats by |
| --- | --- | --- |
| **Malware** | Code built to harm or to gain access: loaders, stealers, backdoors, ransomware | `uses` |
| **Channels** | A place or a means to spread information, such as a forum, a website or a social media account, mostly to track foreign information manipulation and interference (FIMI) | `uses` |
| **Tools** | Legitimate software that attackers misuse, such as remote administration tools or the binaries the LOLBAS project lists | `uses` |
| **Vulnerabilities** | A weakness that attackers exploit, usually a CVE | `targets` |

An arsenal object is never tied to a single threat: the same malware or tool can be used by several, and each `uses` relationship carries its own author, confidence and dates. That is what makes the **Knowledge** tab of an arsenal object useful for attribution.

## Know a malware

:::steps

1. Open :menu[Arsenal > Malware]. Each card shows the aliases (**Known as**), the **Intrusion sets** that use the malware, and the **Targeted countries** and **Targeted sectors**. Search a name or an alias, or filter on **Malware types**.
2. Click the malware. **Overview** opens on its **Details**.
3. Open **Knowledge**. The **Diamond** shows under **Last used by** the threats that use it, and the side menu splits them into **All threats**, **Threat actors**, **Intrusion sets** and **Campaigns**.
4. Go on with **Victimology** for its targets, **Attack patterns** for what it does on a system, **Variants** for related malware, and **Indicators** and **Observables** for what to detect.

:::

::screenshot{id=opencti-arsenal-malware-diamond alt="The Knowledge tab of the malware HarborGate backdoor opened on the diamond model, where Last used by shows two threats: Copper Lantern and Velvet Harbor."}

The **Details** of a malware describe the code itself:

| Field | What it means |
| --- | --- |
| **Malware types** | What the malware does, such as dropper, backdoor or remote-access-trojan |
| **Is family** | **Yes** for a family, tracked across its versions; **No** for one specific sample |
| **Architecture execution env.**, **Implementation languages** | The processors it runs on and the languages it is written in |
| **Capabilities** | What it can do once installed, such as capture credentials or exfiltrate data |
| **Kill chain phases** | The phases of an attack where it acts |
| **First seen**, **Last seen** | When it was first and last seen in the wild, as the sources report it |
| **Nested objects** | The samples and operating systems linked to it as [nested references](/opencti/concepts/nested-objects/) |

:::only{opencti=">=7.261030"}
A malware also has a **Score**, from 0 to 100, which your team or the source sets, for example to rank the malware that matter most to you. OpenCTI does not compute it.
:::

:::example[Check whether HarborGate points to Velvet Harbor]
A hotel chain found HarborGate backdoor on a booking server and asks whether Velvet Harbor is behind it. In :menu[Arsenal > Malware], open **HarborGate backdoor**: **Details** gives the types backdoor and remote-access-trojan, and **Is family** **Yes**. In **Knowledge**, **Last used by** shows Copper Lantern and Velvet Harbor, and **All threats (2)** lists both. **Analyses** shows one report on both, "HarborGate backdoor on booking servers". Answer: HarborGate is shared by two threats of the sector, so it does not attribute the intrusion on its own; look at the infrastructure and the techniques too.
:::

**AI Insights**, next to the tabs of a malware, rates its recent activity and forecasts how it may evolve. It requires the Enterprise Edition: see [Read the home dashboard and AI summaries](/opencti/use/explore/insights/).

## Know a tool or a channel

A tool or a channel page works like a malware page. Its **Knowledge** opens on the same diamond, and its side menu lists the threats that use it.

- A tool gives its **Tool types**, such as remote access or credential exploitation, and its **Tool version**. A tool on a threat's profile means you look for misuse of software you may run yourself: its **Attack patterns** say how the threat uses it.
- A channel gives its **Channel types**: Facebook or Twitter by default, or the values your team adds to the vocabulary. In the creation form of a channel, **Channel type** takes one or more of these values, and the list of channels filters on **Channel type**. Its **Knowledge** adds **Victimology**, for the audiences it targets, **Channels**, for the channels linked to it, and **Observables**, for its accounts and URLs. Follow a channel to map an information operation: which campaigns use it, which other channels relay it.

## Assess a vulnerability

:::steps

1. Open :menu[Arsenal > Vulnerabilities] and search the CVE identifier. To check a whole bulletin, paste its CVEs in [bulk search](/opencti/use/explore/search/#check-a-list-of-values).
2. Open the vulnerability. **Details** gives the signs of exploitation. **EPSS - Score** and **EPSS - Percentile** rate the probability of exploitation in the next 30 days, **CISA KEV** says whether it is a known exploited vulnerability, and **First seen active** when exploitation began. It also gives the **Associated CWE(s)**, the kind of weakness.
3. Read the severity in the tabs of the **Details** card, one per scoring system. **CVSS V3** opens first.
4. Check **Affected software** for the software versions that have the vulnerability, and **Remediated by** for the versions that fix it.
5. Open **Knowledge** to see the threats that target the vulnerability, the malware and tools that exploit it, and the incidents where it was used.

:::

Each CVSS tab gives the base **Score** with its severity, the **Vector** string, and each metric of that version of the Common Vulnerability Scoring System, under its name and abbreviation:

| Tab | Metrics |
| --- | --- |
| **CVSS V2** | **Access Vector (AV)**, **Access Complexity (AC)**, **Authentication (AU)**, the three impacts, then **Exploitability (E)**, **Remediation Level (RL)**, **Report Confidence (RC)** and the **Temporal Score** |
| **CVSS V3** | **Attack Vector (AV)**, **Attack Complexity (AC)**, **Privileges Required (PR)**, **User interaction (UI)**, **Scope (S)**, the three impacts (with **Confidentiality impact (C)**), then **Exploit Code Maturity (E)**, **Remediation Level (RL)**, **Report Confidence (RC)** and the **Temporal Score** |
| **CVSS V4** | **Attack Vector (AV)**, **Attack Complexity (AC)**, **Attack Requirements (AT)**, **Privileges Required (PR)**, **User Interaction (UI)**, the impacts on the vulnerable system (**VS Confidentiality Impact (VC)**, **VS Integrity Impact (VI)**, **VS Availability Impact (VA)**) and on subsequent systems (**SS Confidentiality Impact (SC)**, **SS Integrity Impact (SI)**, **SS Availability Impact (SA)**), and **Exploit Maturity (E)** |

The three impacts measure what an attacker gains: **Confidentiality Impact (C)**, **Integrity Impact (I)** and **Availability Impact (A)**. The temporal metrics and the **Temporal Score** say how the severity changes once exploits and fixes exist. The **Score** next to **CISA KEV** is not a CVSS score: it is a value from 0 to 100 that your team or the source sets.

:::only{opencti=">=7.261119"}
A CVSS score that no source gave shows `-`, not 0: the **Temporal Score** of a vulnerability scored on its base metrics only, or the **Score** of a CVSS version no source used.
:::

To record a severity yourself, create the vulnerability or open the CVSS tabs of its edit panel. Enter the vector string of your source in **CVSS2 Vector**, **CVSS3 Vector** or **CVSS4 Vector**, and leave **CVSS2 Score**, **CVSS3 Score** or **CVSS4 Score** empty: OpenCTI fills each metric from the vector and computes the base score and its severity. At creation, a score you type is kept as your source states it. Changing the vector later computes the score again, and changing one metric rewrites the vector. OpenCTI refuses a CVSS 3 vector that does not start with `CVSS:3.`, and a CVSS 4 vector that does not start with `CVSS:4.0/`.

:::only{opencti=">=7.261030"}
An **SSVC** tab adds the Stakeholder-Specific Vulnerability Categorization decision points. **SSVC Exploitation** says whether exploitation is **None**, at the **Proof of Concept** stage or **Active**. **SSVC Automatable** says whether an attacker can automate it, **Yes** or **No**. **SSVC Technical Impact** says whether the attacker gains **Partial** or **Total** control of the affected system.
:::

::screenshot{id=opencti-arsenal-vulnerability-overview alt="The Overview of a vulnerability: its EPSS score and percentile, its presence in the CISA KEV catalog, its CVSS 3 score and vector, and the software it affects."}

::::example[Decide whether CVE-2024-3400 is urgent]
A hotel asks whether to patch its edge gateways tonight. In :menu[Arsenal > Vulnerabilities], search `CVE-2024-3400` and open it. **Details** shows an **EPSS - Score** of 0.94 and an **EPSS - Percentile** of 0.99, **CISA KEV** **Yes**, and the associated CWE `CWE-77`. The **CVSS V3** tab gives a score of 10.0, with a network attack vector, no privileges and no user interaction. **Knowledge** shows that Velvet Harbor targets it. A critical score, known exploitation and a threat active in the sector: patch now.

To list every vulnerability in the same situation, filter the list on **CISA KEV**, operator **Equals**, value `true`, and **EPSS - Score**, operator **Greater than/ Equals**, value `0.5`. Filter on **EPSS - Percentile** instead, value `0.9`, to keep the vulnerabilities more likely to be exploited than nine CVEs out of ten.

:::only{opencti=">=7.261119"}
These filters leave out the vulnerabilities that no source has scored yet. To find them, filter on **EPSS - Score**, operator **Empty**: the list keeps CVE-2023-46805, which has a CVSS 3 score of 8.2 but no EPSS score, a vulnerability to enrich before you rely on the list. **Not empty** keeps the scored ones only. Every numeric filter, such as **Score** or **CVSS3 Score**, offers these two operators.
:::
::::

## What's next

- [Explore threats](/opencti/use/explore/threats/) for the actors that use this arsenal.
- [Explore techniques](/opencti/use/explore/techniques/) for how they use it.
- [Measure your defenses against a threat](/opencti/use/analyze/security-coverage/) to test whether your defenses detect a threat.
- The [data model reference](/reference/opencti/data-model/#arsenal) for the attributes and relationships of each type of the Arsenal menu.
