---
title: Explore techniques
description: Find which threats use an attack pattern or a narrative in the OpenCTI Techniques menu, since when, and how to mitigate and detect it with courses of action, data components and data sources.
sidebar_position: 10
doc_type: how-to
products:
  - opencti
personas: [cti-analyst, soc-analyst]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/exploring-techniques/
---

The **Techniques** menu answers "how do attackers operate, and how do we answer": the attack patterns of MITRE ATT&CK and other frameworks, the narratives of information operations, the courses of action that mitigate them, and the data that reveals them. This page shows how to find who uses a technique and since when, then how to mitigate and detect it. The tabs and panels every entity page shares are described in [Explore knowledge](/opencti/use/explore/#read-an-entity-page); this page covers what is specific to techniques.

## Before you start

- Your role has the **Access knowledge** capability to read these pages.
- To link a technique to a threat, your role also needs **Create / Update knowledge**; to delete, **Delete knowledge**. See [Create knowledge manually](/opencti/use/ingest/manual/).
- The menu is only as full as what was imported. Attack patterns, courses of action, data components and data sources usually come from connectors such as MITRE ATT&CK, and disinformation techniques from DISARM: see [Connectors](/opencti/use/ingest/connectors/).

## What the Techniques menu holds

| Entry | What it is | Where its links show |
| --- | --- | --- |
| **Attack patterns** | A way adversaries compromise targets: a technique or sub-technique of MITRE ATT&CK, a CAPEC pattern, or a DISARM technique for disinformation | **Knowledge** tab |
| **Narratives** | A story a threat spreads in an information operation, with its subnarratives | **Knowledge** tab |
| **Courses of action** | An action that prevents or answers an attack: a patch, a firewall rule, a training, a policy | **Overview** only |
| **Data components** | A property of a data source that reveals a technique, such as the content of an application log | **Overview** only |
| **Data sources** | A subject of information that sensors and logs collect, such as application logs or network traffic | **Overview** only |

Courses of action, data components and data sources have no **Knowledge** or **Analyses** tab: their links to attack patterns are panels of their **Overview**.

## Find who uses a technique

:::steps

1. Open :menu[Techniques > Attack patterns] and search the technique by name or by its ID, such as `T1566`. The list shows each technique's kill chain phase and ID; filter on a kill chain phase to browse one tactic.
2. Open the attack pattern. **Details** gives its **External ID**, its **Parent attack patterns** or **Sub attack patterns**, its **Kill chain phases**, the **Platforms** it applies to and the **Required permissions**.
3. Open **Knowledge**. The side menu lists what uses the technique: **Threat actors**, **Intrusion sets**, **Campaigns**, **Incidents**, **Malware** and **Tools**. The number beside an entry counts its relationships.
4. Switch the list to **Relationships view** to see how sure and how recent each use is. **Start time** and **Stop time** say when the threat used the technique, as the source states it; **Platform creation date** says when you learned it. See [Lists of linked objects](/opencti/use/explore/#lists-of-linked-objects).
5. Open **Analyses** to list the reports, groupings and cases that mention the technique, and trace each use back to its source.

:::

::screenshot{id=opencti-techniques-attack-pattern-overview alt="The Overview of the attack pattern Phishing: its external ID T1566, its sub attack patterns, its kill chain phase, the platforms it applies to, and the courses of action that mitigate it."}

The other entries of the side menu:

- **Indicators** lists the indicators that point to the technique: what your detection team can deploy.
- **Observables** lists the observables related to the technique.
- **Vulnerabilities** lists the vulnerabilities the technique targets.
- **Victimology** shows only what is recorded as targeted by the technique itself, which is rare. The victims of the threats that use it are on each threat's own **Victimology**.

With the rule **Usage propagation of parent techniques** turned on, a threat that uses a sub-technique also shows as using its parent technique. In **Relationships view**, the wand icon of that line names the rule in its tooltip, **Inferred knowledge based on the rule**. See [How OpenCTI infers new relationships](/opencti/use/enrich/inferences/).

To see all the techniques of one threat at once, on the ATT&CK matrix, open the threat and its **Attack patterns**: see [Explore threats](/opencti/use/explore/threats/).

:::example[Prepare the detection of fake booking requests]
Velvet Harbor gets in through phishing, and your detection team asks what to watch. Open :menu[Techniques > Attack patterns], search `T1566` and open **Phishing**.

**Details** shows the external ID `T1566`, the kill chain phase **initial-access** and the sub attack patterns **Spearphishing Attachment** (`T1566.001`) and **Spearphishing Link** (`T1566.002`). **Detection** advises watching email gateways and the processes that mail clients start. **Courses of action** lists **User Training** and **Antivirus/Antimalware**, and **Data components** lists **Application Log Content**. In **Knowledge**, **Intrusion sets (1)** lists Velvet Harbor; **Analyses** lists the Example CERT report **Fake booking requests reach hotel front desks**. Send the detection team the technique, its data component and the report.
:::

## Mitigate and detect a technique

**Details** of an attack pattern ends with how to answer it: **Detection** gives the detection advice of the framework, **Courses of action** the mitigations, **Data components** the data that reveals it. Open them for more:

- **A course of action** gives its **External ID**, such as `M1017`, the **Log sources** and **Threat hunting techniques** to apply, and **Mitigated attack patterns**, the techniques it answers.
- **A data component** names its **Data source** and the **Attack patterns** it detects.
- **A data source** lists its **Data components**, the **Platforms** it covers and its **Layers**: where the data is collected, such as **host**, **network**, **cloud-control-plane**, **container** or **OSINT**.

Together they tell your detection team which logs to collect to see a technique. To list the mitigations of every technique of a threat at once, use the **Courses of action view** of the threat's **Attack patterns**. To check which of those techniques your security tools should cover, see [Match threats with your security platforms](/opencti/use/explore/entities/#match-threats-with-your-security-platforms).

## Follow a narrative

For disinformation analysis, a narrative is the story an information operation pushes, and DISARM techniques describe how it is pushed.

- In :menu[Techniques > Narratives], **Sub entity lines view** nests each narrative's subnarratives under it. On a narrative's **Overview**, **Parent narratives** or **Subnarratives** place it in that tree.
- A narrative's **Knowledge** lists the **Threat actors**, **Intrusion sets**, **Campaigns** and **Incidents** that use it, and the **Channels** that carry it, such as a social media account or a website.
- **Sightings**, in the side menu of **Knowledge**, lists where the narrative was seen: a country, a sector, an organization.

## What's next

- [Explore threats](/opencti/use/explore/threats/) to see the techniques of an actor on the matrix.
- [Measure your defenses against a threat](/opencti/use/analyze/security-coverage/) to test your defenses against those techniques.
- [Explore arsenal](/opencti/use/explore/arsenal/) for the malware, tools and channels that implement them.
- The [data model reference](/reference/opencti/data-model/#techniques) for the attributes and relationships of each type of the Techniques menu.
