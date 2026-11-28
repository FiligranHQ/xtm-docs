---
title: OpenCRQ
description: Use OpenCRQ to price each cyber risk of your organization in money, see the figure
  move with the threats and your defenses, and fund the controls that remove the most loss.
doc_type: landing
hide_table_of_contents: true
sidebar_label: Overview
products:
  - opencrq
sidebar_position: 0
---

:::product-hero{eyebrow="Cyber risk quantification" title="Put a price on each cyber risk, and fund what lowers it most" primary-label="Start here" primary-to="#start-here" secondary-label="Deploy the XTM platform" secondary-to="/deploy/install/docker-compose/" shot=opencrq-risk-scenarios-list alt="The Risk Scenarios of Grand Meridian France as cards: Ransomware on the Booking Platform at $254K a year, down $9K, and Data Breach on Guest Records at $33K, down $64K, each with its p10 to p90 band and a line of past prices."}
OpenCRQ turns each line of your risk register, such as ransomware on the booking platform or a breach of customer records, into an annualized loss in US dollars, with the chance of losing more. The price is not a score someone typed: OpenCRQ simulates the attacks that the threats targeting you would run against your assets and your controls, and keeps the figure current as they change. Then you compare what a control costs with the loss it removes.
:::

OpenCRQ comes in two editions. The Community Edition, under the Apache 2.0 license, prices risk in one tenant. The Enterprise Edition, which OpenCRQ takes from XTM One's Filigran license, adds several tenants in one platform and the XTM One assistant.

## Start here

The first time you open a new tenant, OpenCRQ shows **Get started**, first in the menu: eight steps, **Company**, **Assets**, **Threats**, **Defenses**, **Finances**, **Tolerance**, **Protect** and **Scenarios**, that take you to a first quantified risk scenario in a few minutes. The menu shows how many are done. Once no step reads **To do**, even with some only started, **Get started** moves to the end of the menu, after **Settings**.

Then pick your role: each tab gives your goal and the pages to read first, in order.

::::tabs
:::tab[Risk managers]
You keep the risk register in money: what each scenario costs a year, what it rests on, and why it moved.

1. [How OpenCRQ quantifies risk](/opencrq/concepts/how-risk-is-quantified/): the model behind every figure, from attack frequency to annualized loss.
2. [Risk scopes, primary assets and supporting assets](/opencrq/concepts/risk-scopes-and-assets/): describe what a loss would hit, and the systems an attacker goes through.
3. [How OpenCRQ merges an asset several sources report](/opencrq/concepts/asset-consolidation/): which value wins, and when two reports are one asset.
4. [How OpenCRQ works](/opencrq/concepts/architecture/): its processes, ports, storage and the services it calls.
5. [Set what an attack costs with the financial profile](/opencrq/use/quantify/loss-drivers/): answer the questions that give each loss its amount.
6. [Build and price a risk scenario](/opencrq/use/quantify/risk-scenarios/): create a scenario, read its price and follow it over time.
:::
:::tab[CISOs and executives]
You decide what to fund, and show the board what the spend buys.

1. [Track cyber risk in money with dashboards](/opencrq/use/decide/dashboards/): the annual loss exposure of the organization and its trend, on one screen.
2. [Choose the treatment plan that removes the most loss](/opencrq/use/decide/treatment-plans/): compare plans by the loss they remove for what they cost.
3. [Review a year's security budget against the loss it avoided](/opencrq/use/decide/budget-reviews/): what last year's spend bought, line by line.
:::
:::tab[CTI and validation teams]
Your intelligence and your test results set the price: see what OpenCRQ reads from them.

1. [Connect OpenCTI with a Live Stream](/opencrq/use/sources/opencti/): which threats to stream, and what OpenCRQ keeps of them.
2. [Follow the threats that target your scopes](/opencrq/use/threats/threat-center/): how a threat is admitted to a scenario and how its activity counts.
3. [Import OpenAEV results](/opencrq/use/sources/openaev/): how simulations audit your controls, technique by technique.
4. [Set how strong your controls are and read how they held](/opencrq/use/defenses/controls/): the posture the attack runs face.
:::
:::tab[Administrators]
You give people access and connect OpenCRQ to the sources that feed it. People sign in to OpenCRQ with their XTM One account: OpenCRQ keeps no password.

1. [Give people access with users, groups and roles](/opencrq/administer/users-and-roles/): add members from XTM One's directory, and the platform's own administrators.
2. [Let integrations sign in with service accounts](/opencrq/administer/service-accounts/): the accounts connectors and scripts write as.
3. [Connect OpenCTI with a Live Stream](/opencrq/use/sources/opencti/): the first connector to set, since threats drive every price.
4. [Follow the background jobs and run one again](/opencrq/administer/background-jobs/): what OpenCRQ computes on its own, and how to see it is done.
5. [Check or stop the usage telemetry sent to Filigran](/opencrq/administer/telemetry/): what anonymous counts leave the instance, and how to turn them off.
:::
:::tab[Platform engineers]
You deploy the XTM platform, which always runs OpenCRQ with the other modules, and keep OpenCRQ healthy and up to date.

1. [Install with Docker Compose](/deploy/install/docker-compose/): the whole XTM platform, OpenCRQ with every other module, behind your reverse proxy.
2. [Configure OpenCRQ](/deploy/configure/opencrq/): finish its setup and sign in to its first tenant through XTM One, then the settings a deployment sets and the connectors of each tenant.
3. [OpenCRQ configuration](/reference/opencrq/configuration/): every key, its variable and its default.
:::
::::

## Find your way around

The menu on the left groups its entries between separators: **Home**, **Dashboards**, **Risk Scenarios**, **Treatment Plans** and **Budget Reviews** first; then your risk scopes and **All risk scopes**; then **Assets**, **Findings** and **Threats**; then **Controls** and **Compliance**; last **Integrations**, shown to those with **Manage Security**, and **Settings**. When you reach two tenants or more, the first row of the menu names the tenant you are in: click it to switch. The chevron next to the OpenCRQ logo opens **Filigran products**, which opens OpenCTI and OpenAEV when they are connected to the tenant, their product pages otherwise, and XTM Hub.

To find a scenario, an asset, a threat, a risk scope or a control by its name, type part of it in **Search the platform...** at the top of the screen and press Enter. The results list the matches of the tenant by type, each with its count: **Risk Scenarios**, **Primary Assets**, **Assets**, **Threats**, **Risk scopes** and **Controls**. **View all** opens the full list when a type has more matches than the card shows. [Find a scenario, an asset or a threat by its name](/opencrq/use/search/) says what the search matches and how to share it.

## How OpenCRQ prices a risk

You describe what you protect and what a loss would cost; the rest of the XTM platform measures who attacks and how well your defenses hold. OpenCRQ prices the attacks against your assets and your controls, and counts what lands, over a thousand worlds by default: each world is a draw of the uncertain inputs. The result is a price that moves for a reason you can trace: a threat becomes active, a control is rolled out, a test shows a defense failing.

::diagram{id=opencrq-overview}

[How OpenCRQ quantifies risk](/opencrq/concepts/how-risk-is-quantified/) explains the model in full: the frequency of attacks, the attack run, the loss drivers, and the distribution a price comes from, its mean and the chance of losing more.

## Decide what to fund

::::split
:::column
A treatment plan is a set of controls you would fund: OpenCRQ prices your scenarios again as if it were done, so each plan shows the loss it removes against what it costs, and its return ([choose the treatment plan that removes the most loss](/opencrq/use/decide/treatment-plans/)). A budget review does the same afterwards, for what was spent ([review a year's security budget against the loss it avoided](/opencrq/use/decide/budget-reviews/)).
:::
:::column
::screenshot{id=opencrq-treatment-plans-list alt="The three treatment plans of Grand Meridian France as cards, best return first, from Zero-trust remote access at +80.4% to MFA for the front-desk accounts at -100%, under four counters: Active plans, Overdue, Negative return at 2 out of 3, and First-year cost."}
:::
::::

:::xtm-one[Ask about your risk in plain words]
With XTM One, an AI agent reads your scenarios, explains why a price moved between two runs, and creates or updates a scenario for you. An XTM One administrator decides which of these changes wait for a person's approval. See [XTM One in OpenCRQ](/xtm-one/use/in-opencrq/).
:::

## Browse OpenCRQ

::section-overview{id=opencrq}
