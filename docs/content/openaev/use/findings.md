---
title: Review what your injects discovered
description: Find the vulnerabilities, open ports, accounts and credentials your injects extracted from their output, see which assets carry them and since when, and read a CVE's context.
doc_type: how-to
products:
  - openaev
personas: [validation-engineer, soc-analyst]
owner: docs-openaev
last_reviewed: 2026-11-07
migrated_from:
  - openaev:/usage/findings/
---

A finding is one fact an inject discovered on your environment: a CVE on a server, an open port, an account, a password, a share. OpenAEV extracts it from the output of the inject, attaches it to the assets, teams or persons it concerns, and dates every time it is seen again. This page lists the findings, opens one to see where and when it appeared, and explains why some values are partly hidden. Findings come from the output parser of an action: see [Turn an action's output into findings](/openaev/use/threat-arsenal/output-parsers/).

::diagram{id=openaev-findings-flow}

## Before you start

- Your role has **Access findings**. Without it, :menu[Findings] does not show in the menu.
- An inject whose action has an output parser has run, in a simulation or an atomic test.

## How findings are made

When an inject's output arrives, its parser turns each value it recognizes into a finding, with a type. A finding is one value of one type in one place: an asset, or a person for what a phishing captures. When an inject finds it again in that place, the same inject or another one, in a simulation or an atomic test, OpenAEV records a new occurrence of the finding and moves its last-seen date, instead of creating a second one. The :menu[Findings] list then groups the same value and type found in every place into one line.

Deleting the inject, the simulation or the atomic test that found a finding keeps the finding and its dates.

| Kind | Types |
| --- | --- |
| Vulnerabilities | CVE, enriched with the CVSS score and the CISA catalog |
| Network | IPv4, IPv6, Port, PortsScan |
| Accounts and secrets | Credentials, Username, AdminUsername, Email, Sid, Group, Computer |
| Active Directory | Delegation, PasswordPolicy, KerberoastableAccount, AsreproastableAccount, AccountWithPasswordNotRequired |
| Files | File, Share |
| Free values | Text, Number |

## List the findings

:::steps

1. Open :menu[Findings]. Each line is one value of one type, with the **Assets** where it was found, **First seen** and **Last seen**. The **Asset groups** column stays empty: open the finding to read its asset groups. The list starts with the most recently seen.
2. Narrow it: type part of a value in the search bar, or filter on **Type**, **Created at**, **Updated at**, **Assets** or **Asset groups**. The list keeps your filters.
3. Click a line to open the finding.

:::

The asset groups of a finding are the groups its assets belong to, whether you added the asset to the group or it matches the filter of a dynamic group, and whatever the inject targeted. A filter on **Asset groups** that is, or is not, a group finds the findings of its assets in the same way. **Empty** and **Not empty** only count the groups an asset was added to.

::screenshot{id=openaev-findings-list alt="The Findings list: the credential ni****** the phishing exercise captured, two Ports scan findings, 10.20.8.21:443 (https) and 10.20.8.15:5432 (postgresql), the CVE CVE-2021-42013 and the masked credential booking_app:Su******, the last four on the asset booking-web-01, with their first and last seen dates."}

The same list, limited to its scope, is in the **Findings** tab of a simulation, of a scenario (every simulation of the scenario together) and of an inject's result, and on the page of an endpoint.

## Read a finding

The finding's page counts its **Occurrences** and its **Impacted assets**, and, when people are concerned, such as credentials captured by phishing, its **Impacted teams** and **Impacted persons**.

- **Information** gives its **Type**, **Value**, **Field** (the output field it came from), **First seen**, **Last seen** and **Tags**.
- **Impacted assets** and **Asset groups** link to every asset the value was found on and to the groups of these assets, **Scenarios** and **Simulations** to those its injects ran in. They gather every occurrence of the value and type, and show only when they hold something. The asset sections require **Access assets**; the **Impacted assets** counter leads to its section. Beyond 500 occurrences, they read the 500 most recent, and say so.
- **Finding timeline** lists every inject execution that produced it: the **Inject**, its **Context** (the simulation and scenario, or **Atomic testing**), the **Impacted targets** and when it was **Seen**. Switch between **List view** and **Timeline view**.
- For a CVE, a **Vulnerability** panel adds its **CVSS Version 3.1** score, its **Status**, **Published** date, **Source** and **Description**, its weakness enumeration (CWE), its references, and, when it is in the catalog, the **CISA's Known Exploited Vulnerabilities Catalog** entry with its **Due Date** and **Required Action**. **View on NVD** opens it at the National Vulnerability Database. The **Remediation** guidance of the panel requires the Enterprise Edition.

::screenshot{id=openaev-findings-detail-cve alt="The page of the CVE-2021-42013 finding on 1 impacted asset: its Information, the Vulnerability panel with a CVSS score of 9.8 Critical, its listing in CISA's Known Exploited Vulnerabilities catalog, its remediation and references, the Impacted assets section with booking-web-01, the Asset groups section with Velvet Harbor targets, and the Finding timeline of the injects that found it on booking-web-01."}

The vulnerability data comes from the CVE taxonomy of the platform: a CVE it does not know reads **There is no information about this vulnerability yet.** See [Taxonomies](/openaev/administer/taxonomies/).

## Why some values are partly hidden

A finding whose type holds a password, a hash or a key is sensitive. OpenAEV masks its secret part wherever it shows it: the lists, the finding's page, the results of injects, dashboards, widgets and the reports made from them.

- Only the secret part is masked: a credential `booking_app:Summit2026!` reads `booking_app:Su******`. The account stays visible, because it tells you which account is exposed.
- The mask keeps the first two characters, so you recognize a secret you already know, and has a fixed width, so it does not give away the length. A secret shorter than five characters is masked entirely.
- **PasswordPolicy** values are never masked: they name a policy setting, not a secret. Kerberoastable and ASREPRoastable accounts show the account name only.

Search, filters, sorting, notification triggers and dashboards work on the masked value only. A search on `booking_app` or `Su` finds the credential; a search on `Summit` finds nothing. Two passwords that start with the same two characters on the same account show as one line. A saved filter or trigger written on a full secret no longer matches: rewrite it on the account name.

The platform keeps the full value in its database, for deduplication and attack paths. Protect the database and its backups like any store of credentials.

:::example[Find out what the booking web front gives away]
On 12 April 2026, the validation engineer of Grand Meridian Hotels runs the atomic test **Booking web front discovery on booking-web-01**. Its action replays what Velvet Harbor learned on the web front of the booking system, and its output parser turns what it prints into findings. The engineer opens :menu[Findings]: four of its lines come from this test, all on the asset `booking-web-01`; the fifth, `ni******`, is the account the phishing exercise of the front desks captured. Two **Ports scan** findings show that the web front reaches the booking database, `10.20.8.15:5432 (postgresql)`, and a **Credentials** finding reads `booking_app:Su******`: the database account of the booking application, found in a configuration file.

The engineer opens the **CVE** finding `CVE-2021-42013`. Its **Vulnerability** panel gives a **CVSS Version 3.1** score of 9.8, shows it in **CISA's Known Exploited Vulnerabilities Catalog** with its **Required Action**, and its remediation asks for Apache HTTP Server 2.4.51 or later. Its **Asset groups** section names `Velvet Harbor targets`, the dynamic group of every asset the replay aims at, and its **Finding timeline** shows the atomic test that found it. The engineer sends the three findings to Hélène Marchetti, the CISO: from the web front, an attacker reaches the booking database with a valid account.
:::

## What's next

- [Read the result of an inject](/openaev/use/injects/results/): the findings of one inject, next to its expectations.
- [Turn an action's output into findings](/openaev/use/threat-arsenal/output-parsers/).
- [Taxonomies](/openaev/administer/taxonomies/): the CVE data behind the vulnerability panel.
- [Custom dashboards](/openaev/use/dashboards/): chart findings over time.
