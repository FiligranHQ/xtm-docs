---
title: Manage the tags, attack patterns and vulnerabilities
description: Keep the reference data of a tenant in order, its tags, attack patterns, kill chain phases and vulnerabilities, that classify scenarios, actions and assets.
doc_type: how-to
products:
  - openaev
personas: [admin, validation-engineer]
owner: docs-openaev
last_reviewed: 2026-11-07
migrated_from:
  - openaev:/administration/taxonomies/
---

Taxonomies are the reference data a tenant classifies its work with: tags on scenarios, assets and teams, attack patterns and kill chain phases on actions and injects, vulnerabilities on actions. Most of them come from collectors, MITRE ATT&CK for the attack patterns and the NVD for the vulnerabilities; you add your own tags and fix or complete the rest. This page shows what each taxonomy is for and how to keep it in order. Each tenant has its own taxonomies.

## Before you start

- To manage tags, your role has **Manage tags**, and **Delete tags** to delete one. The **Manager** group of a tenant has both.
- To manage attack patterns, kill chain phases and vulnerabilities, your role has **Manage tenant settings**, and **Delete tenant settings** to delete. The **Admin** group has them.
- The **Remediation** tab of a vulnerability needs an Enterprise Edition license.

## Where the taxonomies come from

::diagram{id=openaev-taxonomies-sources}

| Taxonomy | What it classifies | Where it comes from |
| --- | --- | --- |
| **Tags** | Scenarios, simulations, injects, assets, asset groups, teams, persons, organizations, documents, challenges and findings, to filter them and drive the [default asset rules](/openaev/configure/asset-rules/) | You |
| **Attack patterns** | Threat arsenal actions, and through them the injects and the coverage of a scenario, by technique of the MITRE ATT&CK matrices: Enterprise, PRE, Mobile and ICS | The MITRE ATT&CK collector, and your own entries |
| **Kill chain phases** | The attack patterns, by stage of the attack, such as Initial Access or Execution | The MITRE ATT&CK collector, and your own entries |
| **Vulnerabilities** | Threat arsenal actions, by Common Vulnerabilities and Exposures (CVE) identifier, with their score and their entry in the CISA catalog | The NVD CVE collector, and your own entries |

Collectors fill the tenant they work for. In xtm-docker, the MITRE ATT&CK and NVD CVE collectors fill the default tenant: a new tenant starts with seven sample vulnerabilities and no attack pattern until a collector runs for it. See [Connect the integrations of a tenant](/openaev/administer/multi-tenancy/#connect-the-integrations-of-a-tenant).

## Add a tag

:::steps
1. Open :menu[Settings > Taxonomies > Tags] and click **Create**.
2. Enter the **Value**, the text of the tag, and its **Color** as `#RRGGBB`.
3. Click **Create**.
:::

Use tags for what the other taxonomies do not hold: an organization, a site, a campaign, a regulation. Keep one spelling per idea: two tags that differ only by their case or an accent split your filters in two.

## Add or fix an attack pattern

:::steps
1. Open :menu[Settings > Taxonomies > Attack patterns]. Search by name or identifier to check that the technique is not there already.
2. Click **Create**, or open the menu of an existing attack pattern and choose **Update**.
3. Fill in the **External ID**, such as `T1566.001`, the **Kill chain phases** it belongs to, the **Name** and the **Description**.
4. Click **Create** or **Update**.
:::

::screenshot{id=openaev-taxonomies-attack-patterns alt="The Attack patterns of the tenant sorted by External ID, PowerShell T1059.001 under the [mitre-attack] Execution phase and Valid Accounts T1078 under Persistence, with the Create button at the top right."}

A kill chain phase, under :menu[Settings > Taxonomies > Kill chain phases], has a **Phase name**, a **Phase short name**, the **Kill chain name** it belongs to, such as `mitre-attack`, an **External Id** and an **Order**, its position in the chain. The **Order** is required, and is a whole number, 0 or more: the form refuses another value with **Must be a whole number** or **Must be greater than or equal to 0**.

The MITRE ATT&CK collector creates or updates the attack patterns of its tenant by their **External ID** at each run: a change you make to one of its techniques lasts until the next run. Give the techniques you add yourself identifiers MITRE ATT&CK does not use.

## Add a vulnerability

:::steps
1. Open :menu[Settings > Taxonomies > Vulnerabilities] and click **Create**.
2. On the **General** tab, enter the **VULNERABILITY ID**, such as `CVE-2024-3400`, and its **CVSS Version 3.1** score. Both are fixed once the vulnerability exists. Add the **Description**, the dates and the status from the NVD, the entry of **CISA's Known Exploited Vulnerabilities Catalog**, the weaknesses (CWE) and the references.
3. With the Enterprise Edition, on the **Remediation** tab, write the **Vulnerability Remediation**: what fixes it.
4. Click **Create**.
:::

::screenshot{id=openaev-taxonomies-vulnerability-create alt="The Add a new vulnerability panel on its General tab, with the required VULNERABILITY ID and CVSS Version 3.1 fields, then the Description, Quick Info and CISA's Known Exploited Vulnerabilities Catalog sections, and the Remediation tab next to General."}

Click a vulnerability in the list to read its details.

:::example[Classify the Velvet Harbor replay]
The security engineer of Grand Meridian Hotels builds the scenario that replays Velvet Harbor against the front-desk workstations, and the administrator of the tenant prepares its taxonomies first:

| Taxonomy | Entry |
| --- | --- |
| **Tags** | **Value** `velvet-harbor`, **Color** `#8C1D40`; **Value** `front-desk`, **Color** `#1E88E5` |
| **Attack patterns** | The MITRE ATT&CK collector, deployed for the tenant from its **Integrations** screen, has loaded Spearphishing Attachment (`T1566.001`), PowerShell (`T1059.001`), Valid Accounts (`T1078`), Data from Local System (`T1005`) and Web Protocols (`T1071.001`) |

The scenario and the front-desk assets carry the two tags, so a filter on `velvet-harbor` finds every simulation of the replay, and the actions of each inject show their technique in the coverage of the scenario.
:::

## What's next

- [Set the default asset rules](/openaev/configure/asset-rules/), which use the tags of a scenario.
- [Give each organization its own tenant](/openaev/administer/multi-tenancy/), and connect its collectors.
- [How access works in OpenAEV](/openaev/concepts/access/).
