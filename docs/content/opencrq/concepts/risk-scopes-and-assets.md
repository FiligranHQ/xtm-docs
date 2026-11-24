---
title: Risk scopes, primary assets and supporting assets
description: How a risk scope, its primary assets and the supporting assets behind them define what OpenCRQ prices.
doc_type: explanation
products:
  - opencrq
personas: [risk-manager]
owner: docs-opencrq
last_reviewed: 2026-11-04
---

Before OpenCRQ can price a risk, it needs to know what you price it for and what an attacker can hit. A **risk scope** is the part of the organization you assess, such as a subsidiary or a country entity. Its **primary assets** are what the business would lose: a service, a data set, a process. **Supporting assets** are the estate that carries them: applications, servers, workstations, accounts. Links between the two say how each supporting asset serves a primary asset. Together they decide how often a scope is attacked, what an attack can reach, and what it costs.

::diagram{id=opencrq-risk-scopes-and-assets}

The terms follow ISO/IEC 27005: primary assets are business assets, supporting assets are the means that carry them. A [risk scenario](/opencrq/use/quantify/risk-scenarios/) then names an attack outcome and the primary assets it would hit, inside one risk scope.

## What each one holds and what it changes

| | Where you find it | What you set | What it changes in the price |
| --- | --- | --- | --- |
| Risk scope | :menu[All risk scopes], then the scope's **Details** | Identity, the estate it is measured over, its declared estate, the financial profile, the risk tolerance | How often attacks happen, which assets the attack run walks, how large the estate is beyond the assets listed, what each loss costs |
| Primary asset | The scope's **Primary Assets** | **Name**, **Owner**, **Description** and its **Business resources**; for the record, its security needs and recovery objectives | Which loss drivers OpenCRQ recommends for a scenario aimed at it, and which supporting assets the attacks aim for |
| Supporting asset | :menu[Assets], for the whole tenant | Its category, exposure and criticality, declared or imported | Which attack techniques apply, and where the attacks get in |
| Link | A primary asset's **Business dependencies** tab | The roles of each supporting asset toward the primary asset | Which supporting assets the attack run aims at: every supporting asset linked to one of the scenario's primary assets. The roles set the **Impact if compromised** of the **Targets** tab, which shows which supporting assets matter most to the scenario |

## A risk scope

A risk scope belongs to a tenant, and everything you price inside it reads its settings. A new tenant starts with one risk scope, **Enterprise**, measured over the whole estate: the **Company** step of :menu[Get started] gives it its name, sector, country, revenue band and employee count. You create other scopes from :menu[All risk scopes] with **Create**, in five steps: **Identity**, **Estate**, **Declared estate**, **Financial profile** and **Review & Create**. Only the name is required, but a scope with blanks prices less precisely.

- **Sector**, **Country** and **Employee Count** place the scope in the catalogue of attack frequencies, the size most of all. Without an employee count, the revenue band gives the size: the band the **Annual revenue** of the financial profile sets, or the one chosen in **Get started**. A sector or a country left blank, or one the catalogue does not measure, reads the row of every sector or every country; with neither an employee count nor a revenue band, every size weighs alike. See [Set the threat event frequency](/opencrq/use/threats/threat-event-frequency/).
- **Containment time (MTTC)** is the delay, in hours, from your detection until the attacker can no longer progress: a best, typical and worst case. The attack run uses it to decide whether a detected attack is stopped before its objective (**Evicted in time**). All three values, or none: left blank, OpenCRQ uses a platform default.
- **Estate measured** says which supporting assets the scope's attack runs walk. **Whole estate** measures every asset of the tenant: the choice of a whole company, or of an entity that shares its estate. **Mapped assets only** measures the assets you map into the scope: the choice of a subsidiary or of a unit running its own estate. A scope on mapped assets that maps none and declares no estate cannot be priced: its scenarios read **No mapped asset**.
- The **Declared estate** says how many assets of each kind the scope holds, whether you listed them or not: workstations and laptops, servers, user accounts, web applications, network devices, and seven more kinds. See [The declared estate](#the-declared-estate).
- The **financial profile** is a questionnaire on revenue, downtime, data held and contracts. Each scenario's loss estimates derive from it. See [Set what an attack costs with the financial profile](/opencrq/use/quantify/loss-drivers/).
- The **Risk tolerance** sets the annual loss the scope accepts and the highest probability of exceeding it. It applies to each scenario on its own, never to their sum, and a scenario can set its own.

## Primary assets

A primary asset belongs to one risk scope. To add one, open the risk scope in the sidebar, below **Budget Reviews**, click :menu[Primary Assets], then **Create**: the **Create a new primary asset** panel asks for its **Name**, an optional **Owner** and **Description**, and its **Business resources**. Its **Business resources** say what kind of value it holds: **Personal data**, **Payment card data**, **Operations and services**, **Services delivered to others under commitments**, and six more. At least one is required, and the choice matters: OpenCRQ recommends the loss drivers of a scenario from its attack outcome and its business resources. A scenario takes the business resources of its primary assets when you create it or change its primary assets; changing those of a primary asset later leaves the scenarios already aimed at it as they were. A ransomware scenario on **Operations and services** recommends lost margin from downtime; a data breach on **Personal data** recommends notification and regulatory fines. The [business resources reference](/reference/opencrq/business-resources/) lists each pairing.

The panel also records what a business impact analysis says of the asset: a **Reference** that no other primary asset of the scope uses, its **Business function**, its **Security needs** (**Availability**, **Integrity**, **Confidentiality** and **Traceability**, each from **Low** to **Very High**) and its **Recovery objectives** (**RTO**, **RPO** and **MTPD**). The list shows the highest of the four needs as the **Overall level** and filters on it and on the business function. These fields document the asset; the price does not read them.

## Supporting assets and the estate

Supporting assets live in the tenant's library, under :menu[Assets], shared by every risk scope. You declare them or import them, for example from Microsoft Defender. A detection and prevention source that OpenAEV imports, such as an EDR or a SIEM, gets the **Security platform** category. The code your organization builds and ships goes under **Source code**: a **Code repository**, a **CI/CD pipeline**, a **Package registry** or a **Software package**. The attack run reads three things about each one:

- its category (web application, server, workstation, user account, network device, source code, point of sale...), which decides the techniques an attacker can play on it: a user account opens phishing and stolen credentials, a workstation drive-by compromise, and a code repository the theft of credentials left in code, and of the code itself;
- its exposure: an internet-facing asset opens initial access by exploiting a public service or an external remote service;
- its criticality.

The assets linked directly to a scenario's primary assets weigh more when an attack reaches for its objective: the CRQ tab of a scenario shows this as **Target boost**. See [Build the attack surface from your assets](/opencrq/use/attack-surface/assets/).

## The declared estate

The supporting assets you declare or import are rarely the whole estate: an integration covers part of it, and nobody lists two thousand workstations by hand. The **Declared estate** of a risk scope says how large the estate really is. For each of twelve kinds of asset, from **Host › Workstations and laptops** and **Host › Servers and virtual machines** to **Identity › User accounts**, **Web application** and **Network device**, you answer with a band (**None**, **1 to 10**, **11 to 50**, up to **More than 5,000**), or with **Specify** for an **Exact number** or a **Range**. For the kinds an attacker can reach from outside (servers, containers, cloud resources, web applications, AI targets, network devices), you also answer **At least one reachable from the Internet?**. A kind you leave unanswered declares nothing.

You answer it in the **Declared estate** step when you create the scope, or later from its **Details**: the **Declared estate** tab shows the answers, and **Answer the questionnaire** opens them for editing beside the number of assets of each kind already measured. The declaration changes the price in two ways:

- **The attack run** completes the assets it walks up to your answer. For each kind, it adds internal assets of **Unknown** criticality until the count reaches the lower bound of your answer, and each simulated world draws its own count within the range and keeps it for every year it plays. When you answer that at least one is reachable from the Internet and none of the measured ones is, one added asset faces the internet. The added assets support no primary asset.
- **The workstation and server counts** size the forensics and restoration loss drivers. Without them, OpenCRQ estimates the counts from the employee count and the size of the organization.

A scope measured over **Mapped assets only** that maps no asset is priced from its declaration alone. Under :menu[Assets], inside a risk scope, the card above the list sets the measured assets of each category beside the declared ones. The **Events** tab of the scope's **Details** keeps the history of its identity, mapped assets, financial profile and declared estate: who changed which answer, from what to what, and when.

## Links and their roles

A link from a supporting asset to a primary asset carries one or more roles. Each role says which security criteria of the primary asset the supporting asset shares: confidentiality (C), integrity (I), availability (A) and traceability (T).

| Role | C | I | A | T |
| --- | --- | --- | --- | --- |
| **Stores**, **Processes** | inherited | inherited | inherited | one notch lower |
| **Reads / renders** | inherited | one notch lower | one notch lower | not inherited |
| **Transmits** | inherited | inherited | inherited | not inherited |
| **Controls / administers** | inherited | inherited | inherited | inherited |
| **Logs / monitors** | one notch lower | not inherited | not inherited | inherited |
| **Supports availability** | not inherited | not inherited | inherited | not inherited |
| **Performs / executes** | one notch lower | inherited | inherited | one notch lower |
| **Unknown** | inherited | inherited | inherited | inherited |

A link with several roles keeps the strongest of each. **Unknown** inherits everything, so a link you have not qualified never counts for less than a qualified one. A link between two supporting assets (a web application running on a database server) carries no role: it passes on what its dependent inherits, halved at each step, up to five levels.

The **Impact if compromised** shows which supporting assets matter most to the scenario. To price it, the attack run aims at every supporting asset linked to the scenario's primary assets, whatever its roles, and weighs them 10 times more at the steps that reach for the objective (the **Target boost** of the CRQ tab). To have the attacks aim at an asset you reach through another supporting asset, such as a database server behind a web application, link it to the primary asset too.

Integrations also observe relations between two supporting assets that are not links: a user account assigned to a device, from Microsoft Entra ID or Intune, and an attack path OpenAEV showed from one asset to the next, **Leads to** and **Reached from**, either **OpenAEV, proven by a test** or **OpenAEV, inferred**. They show on the asset's **Access relations** tab, carry no role, and make no asset a target of a scenario. The attack run reads one of them: a user account assigned to a device tells it which device a phished person works on. See [Build the attack surface from your assets](/opencrq/use/attack-surface/assets/).

Each attack outcome breaches some criteria: ransomware reaches availability and confidentiality, and integrity in part; a data breach reaches confidentiality only. The share of those criteria a link passes on is the **Impact if compromised** that a scenario's **Targets** tab shows for each supporting asset.

::screenshot{id=opencrq-risk-scopes-and-assets-dependencies alt="The Business dependencies graph of Guest records, 5 supporting assets, 4 critical and 2 exposed: the booking web application processes the records, the booking database server stores them, the front-desk workstations and the night.audit account read and render them, with the head-office VPN gateway one level behind."}

:::example[Grand Meridian France and its booking platform]
Grand Meridian France is a risk scope of the Grand Meridian Hotels tenant: **Sector** Hospitality, **Country** France, **Employee Count** 4,200, **Containment time (MTTC)** 4, 24 and 72 hours. It is measured over the **Whole estate**, six assets today, so its attack runs also walk the payment terminals of the Spanish hotels.

It holds two primary assets: the **Online booking platform** (**Operations and services**, **Services delivered to others under commitments**) and the **Guest records** (**Personal data**). The booking web application **Performs / executes** the booking platform and **Processes** the guest records, the booking database server **Stores** the guest records and **Supports availability** of the booking platform, and the shared night.audit account **Reads / renders** the guest records.

In the scenario Ransomware on the Booking Platform, the **Targets** tab reads 80% for the web application and 40% for the database server. Ransomware reaches availability and confidentiality in full and integrity in part; **Performs / executes** passes on all of availability and integrity but only part of confidentiality, which gives 80%. **Supports availability** passes on availability alone, which gives 40%.
:::

## What's next

- [Build and price a risk scenario](/opencrq/use/quantify/risk-scenarios/) on these primary assets.
- [Set what an attack costs with the financial profile](/opencrq/use/quantify/loss-drivers/).
- [Build the attack surface from your assets](/opencrq/use/attack-surface/assets/).
- [How OpenCRQ quantifies risk](/opencrq/concepts/how-risk-is-quantified/).
