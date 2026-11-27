---
title: How OpenCRQ's objects connect, from a risk scope to its price
description: The objects OpenCRQ keeps, from risk scopes and assets to threats, controls, scenarios and plans, what each one belongs to, and which prices a change to one reaches.
doc_type: explanation
products: [opencrq]
personas: [risk-manager, admin, developer]
owner: docs-opencrq
last_reviewed: 2026-11-27
---

OpenCRQ keeps a small set of connected objects, and every price it shows is read from them. Some belong to the whole tenant and serve every risk scope: the supporting assets and their findings, the threats, the controls you declare. Others belong to one **risk scope**: its primary assets, the risk scenarios that aim at them, and the treatment plans and budget reviews you decide with. A scenario's price is a **simulation** of the attacks those threats would run through those assets against those controls. Knowing which object holds a value tells you which prices move when you change it. This page describes each object, what it belongs to, where you find it, and how they point to each other. For how the price itself is computed, read [How OpenCRQ quantifies risk](/opencrq/concepts/how-risk-is-quantified/).

::diagram{id=opencrq-data-model-objects}

## What OpenCRQ keeps

| Object | What it is | Belongs to | Where you find it | API namespace |
| --- | --- | --- | --- | --- |
| Risk scope | The part of the organization you assess, with its sector, country, financial profile, declared estate and risk tolerance | The tenant | :menu[All risk scopes] | `organizations`, `financialProfiles`, `declaredEstate` |
| Primary asset | What the business would lose: a service, a data set, a process, with its business resources | One risk scope | The scope's **Primary Assets** | `primaryAssets` |
| Supporting asset | A device, an identity, an application or another part of the estate, declared or imported | The tenant, measured by every scope over the whole estate, or by the scopes it is mapped into | :menu[Assets] | `supportingAssets` |
| Link | A supporting asset serving a primary asset, with its roles, or a supporting asset serving another | The two assets it joins | A primary asset's **Business dependencies** tab | `assetDependencies` |
| Finding | A CVE on one supporting asset, or an ATT&CK technique an OpenAEV test ran on it without a security tool preventing or detecting it | One supporting asset | :menu[Findings], and the asset's **Findings** tab | `supportingAssets`, `findingScore` |
| Threat | A threat actor, an intrusion set or a campaign, with its techniques and the CVEs it exploits | The tenant, matched to each scope by sector and country | :menu[Threats] | `threats` |
| Control | One of the 51 controls of the defense catalogue, with the strength and coverage you declare | The catalogue is the platform's; the declaration the tenant's, and a scope can declare its own | :menu[Controls] | `controlCapabilities` |
| Compliance assessment | Your answers to the requirements of a framework, which feed the controls once published | The tenant | :menu[Compliance] | `complianceAssessments` |
| Risk scenario | An attack outcome on primary assets, with its frequency, its loss estimates and, if you set one, its own risk tolerance | One risk scope | :menu[Risk Scenarios] | `riskScenarios`, `scenarioLossEstimates` |
| Simulation | One run of the attack run engine on a scenario: its annualized loss, the band around it, how the attacks ended | One risk scenario | The scenario's **Simulation History** | `crqIndicators` |
| Treatment plan | The controls you would fund, each aimed at a better posture, priced against the scope's scenarios | One risk scope | :menu[Treatment Plans] | `treatmentPlans` |
| Budget review | A year of security spend, line by line with the controls each line paid for, set against the loss it avoided | One risk scope | :menu[Budget Reviews] | `budgetReviews` |

Each procedure of these namespaces is in the [API reference](/reference/opencrq/api/).

## Three levels: the platform, the tenant, the risk scope

Where an object lives decides who shares it:

- **The platform** holds the catalogues no tenant changes: the 51 controls of the defense catalogue, with what each strength level means, and MITRE ATT&CK. Every tenant reads the same ones. See the [controls reference](/reference/opencrq/controls/).
- **The tenant** holds its own copy of the risk catalogues, the attack outcomes, the loss drivers, the business resources, the sectors and the countries, and everything its sources bring: the supporting assets and their findings, the CVEs, the threats. It also holds the controls' declarations and the compliance assessments. Nothing crosses from one tenant to another: two tenants that connect the same OpenCTI each keep their own copy of its threats.
- **The risk scope** holds what you price and decide for one part of the organization: its primary assets, its risk scenarios, its treatment plans and its budget reviews, with the settings they read, its financial profile, declared estate and risk tolerance. A scope can also declare its own level for a control, axis by axis, over the tenant's.

A scenario, a plan or a review never moves to another scope. To price the same outcome for two subsidiaries, create a scenario in each scope.

## How a price connects to the rest

A risk scenario names one attack outcome, chosen when you create it, and the primary assets it would hit. Everything else its price reads, it reaches through them:

- **The attackers** are the tenant's threats that the model judges likely to seek that outcome, weighed more when they target the scope's sector or country. See [How a threat gets into a scenario](/opencrq/use/threats/threat-center/#how-a-threat-gets-into-a-scenario).
- **The estate the attacks walk** is the supporting assets the scope measures, completed up to its declared estate. The attacks aim at those linked to the scenario's primary assets. See [Risk scopes, primary assets and supporting assets](/opencrq/concepts/risk-scopes-and-assets/).
- **The openings** are the findings on those assets, and what each asset's category and exposure let an attacker try.
- **The defenses** are the controls, at the level the scope declares, or the tenant where the scope declares nothing, lowered where OpenAEV's tests held less often. See [Set how strong your controls are](/opencrq/use/defenses/controls/).
- **The cost** is the scenario's loss estimates, derived from the scope's financial profile for the loss drivers its outcome and business resources recommend, or entered by hand. See [Set what an attack costs with the financial profile](/opencrq/use/quantify/loss-drivers/).

Each simulation is kept: the scenario's **Simulation History** lists them, and the price the scenario and the dashboards show is the latest one. A treatment plan prices the scope's scenarios again as if the controls it funds were in place; a budget review prices what each of its lines paid for on the latest simulation of the scope's scenarios.

## What a change reaches

Because the shared objects serve every scope, a change to one reaches further than the screen you make it on:

| You change | What it reaches |
| --- | --- |
| A supporting asset, a link or a finding | Every scenario that walks that asset: each scope that measures it, through the scenarios aimed at the primary assets it serves |
| A threat, in OpenCTI | Every scenario of the tenant whose outcome it may seek, in every scope |
| A control's level for the tenant | Every scope that does not declare that control itself |
| A control's level for one scope | That scope's scenarios only |
| A scope's financial profile | The loss estimates it derives, in that scope's scenarios |
| A primary asset's business resources | Nothing already aimed at it: a scenario takes the business resources of its primary assets when you create it or change its primary assets |

OpenCRQ prices the scenarios again on its own when what their price reads changes. See [Build and price a risk scenario](/opencrq/use/quantify/risk-scenarios/) for when, and [Follow the background jobs](/opencrq/administer/background-jobs/) to see it done.

:::example[Grand Meridian's two scopes, seen from their objects]
Grand Meridian Hotels prices its risk in two risk scopes of one tenant: Grand Meridian France, the hotels in France and the head office that runs the booking platform, and Grand Meridian Spain, whose front desks take card payments on their own terminals.

- The head-office VPN gateway is a supporting asset of the tenant. OpenCTI says Ember Tide and Velvet Harbor target CVE-2024-3400, the flaw of edge gateways, so the risk manager attaches it to the gateway as a finding. Every scenario whose attacks walk the gateway now reads that opening.
- The scenario Data Breach on Front-Desk Card Payments belongs to Grand Meridian Spain and aims at its primary asset Front-desk card payments, which the front-desk payment terminals process. Its attackers are the tenant's threats that seek **Data Breach & Resale**.
- Each scope declares its own Endpoint Detection and Response: **Intermediate** in France, on the servers and most workstations, **Basic** in Spain, an antivirus on the front-desk workstations. A level set for the whole tenant would change neither.
- The NIST CSF 2.0 assessment that rates access control stays in **Draft**. Published, it would feed the tenant's identity controls, and Grand Meridian Spain, which declares no identity control of its own, would see its scenario priced again.

Each new price shows as a run in the scenario's **Simulation History**, and the treatment plans and the budget review of its scope read it.
:::

## What's next

- [Risk scopes, primary assets and supporting assets](/opencrq/concepts/risk-scopes-and-assets/): what each one holds and what it changes in the price.
- [How OpenCRQ quantifies risk](/opencrq/concepts/how-risk-is-quantified/): the model a simulation runs.
- [How OpenCRQ merges an asset several sources report](/opencrq/concepts/asset-consolidation/): one supporting asset from several sources.
- [OpenCRQ API](/reference/opencrq/api/): every procedure of the namespaces above.
