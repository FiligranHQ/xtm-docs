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

:::product-hero{eyebrow="Cyber risk quantification" title="Put a price on each cyber risk, and fund what lowers it most" primary-label="Start here" primary-to="#start-here" secondary-label="Install OpenCRQ" secondary-to="/deploy/install/opencrq/" shot=opencrq-risk-scenarios-list alt="The Risk Scenarios of Grand Meridian France as cards: Ransomware on the Booking Platform at $254K a year, down $9K, and Data Breach on Guest Records at $33K, down $64K, each with its p10 to p90 band and a line of past prices."}
OpenCRQ turns each line of your risk register, such as ransomware on the booking platform or a breach of customer records, into an annualized loss in your currency, with the chance of losing more. The price is not a score someone typed: OpenCRQ simulates the attacks that the threats targeting you would run against your assets and your controls, and keeps the figure current as they change. Then you compare what a control costs with the loss it removes.
:::

:::info[Not released yet]
OpenCRQ is not released yet. These pages follow its development and will be versioned from its first release, like OpenCTI.
:::

## Start here

Pick your role: each tab gives your goal and the pages to read first, in order.

::::tabs
:::tab[Risk managers]
You keep the risk register in money: what each scenario costs a year, what it rests on, and why it moved.

1. [How OpenCRQ quantifies risk](/opencrq/concepts/how-risk-is-quantified/): the model behind every figure, from attack frequency to annualized loss.
2. [Risk scopes, primary assets and supporting assets](/opencrq/concepts/risk-scopes-and-assets/): describe what a loss would hit, and the systems an attacker goes through.
3. [Set what an attack costs with the financial profile](/opencrq/use/quantify/loss-drivers/): answer the questions that give each loss its amount.
4. [Build and price a risk scenario](/opencrq/use/quantify/risk-scenarios/): create a scenario, read its price and follow it over time.
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
:::
:::tab[Platform engineers]
You deploy OpenCRQ beside the XTM platform and keep it running.

1. [Install OpenCRQ](/deploy/install/opencrq/): its service, its database and file storage, and its sign-in through XTM One.
2. [Configure OpenCRQ](/deploy/configure/opencrq/): the settings a deployment sets, and the connectors of each tenant.
3. [OpenCRQ configuration](/reference/opencrq/configuration/): every key, its variable and its default.
:::
::::

## How OpenCRQ prices a risk

You describe what you protect and what a loss would cost; the rest of the XTM platform measures who attacks and how well your defenses hold. OpenCRQ plays the attacks over thousands of simulated years, against your assets and your controls, and counts what lands. The result is a price that moves for a reason you can trace: a threat becomes active, a control is rolled out, a test shows a defense failing.

::diagram{id=opencrq-overview}

[How OpenCRQ quantifies risk](/opencrq/concepts/how-risk-is-quantified/) explains the model in full: the frequency of attacks, the attack run, the loss drivers and why the mean and the band say different things.

## Decide what to fund

::::split
:::column
A treatment plan is a set of controls you would fund: OpenCRQ prices your scenarios again as if it were done, so each plan shows the loss it removes against what it costs, and its return. A budget review does the same afterwards, for what was spent.

:::cards{cols=2}
  ::card[Compare plans by the loss they remove.]{icon=ListChecks title="Treatment plans" href="/opencrq/use/decide/treatment-plans/"}
  ::card[What last year's spend avoided.]{icon=ClipboardCheck title="Budget reviews" href="/opencrq/use/decide/budget-reviews/"}
:::
:::
:::column
::screenshot{id=opencrq-treatment-plans-list alt="The three treatment plans of Grand Meridian France as cards, best return first, from Zero-trust remote access at +80.4% to MFA for the front-desk accounts at -100%, under four counters: Active plans, Overdue, Negative return at 2 out of 3, and First-year cost."}
:::
::::

:::xtm-one[Ask about your risk in plain words]
With XTM One, an AI agent reads your scenarios, explains why a price moved between two runs, and drafts a scenario or a plan that a person approves. See [XTM One in OpenCRQ](/xtm-one/use/in-opencrq/).
:::

## Browse OpenCRQ

::section-overview{id=opencrq}
