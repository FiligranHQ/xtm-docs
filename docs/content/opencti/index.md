---
title: OpenCTI
description: Use OpenCTI to structure, explore, enrich, automate and share cyber
  threat intelligence, and find the pages to read first for your role.
doc_type: landing
hide_table_of_contents: true
sidebar_label: Overview
products:
  - opencti
sidebar_position: 0
---

:::product-hero{eyebrow="Threat intelligence platform" title="Turn scattered intelligence into a knowledge graph your teams act on" primary-label="Start here" primary-to="#start-here" secondary-label="Deploy the XTM platform" secondary-to="/deploy/install/docker-compose/" shot=opencti-report-overview alt="An OpenCTI report: its description, report type and publication date, the distribution of the entities it contains, and its reliability, confidence and processing status."}
OpenCTI stores what you know about threats as Structured Threat Information Expression (STIX) 2.1 objects and the relationships between them: reports, threat actors, malware, indicators, vulnerabilities, sightings. Feeds and connectors bring the data in; analysts explore, enrich and qualify it; playbooks, dashboards and feeds put it to work.
:::

## Start here

Pick your role: each tab gives your goal and the pages to read first, in order.

::::tabs
:::tab[CTI analysts]
You structure what you know about threats, link it, and share it under the right markings.

1. [Data model](/opencti/concepts/data-model/): which entity, relationship and container holds each thing you know.
2. [Explore knowledge](/opencti/use/explore/): what every entity page shows, and how to find what the platform already knows.
3. [Import from files](/opencti/use/ingest/files/): turn a PDF report or a STIX bundle into knowledge you review before it is written.
4. [Export knowledge to a file](/opencti/use/share/export/): send a report to a partner CERT as a PDF or a STIX file, within its markings.
:::
:::tab[SOC analysts]
You get the context of a suspicious IP address, domain or hash in seconds, then work the incident.

1. [Search for knowledge](/opencti/use/explore/search/): find an observable and what is known about it.
2. [Pivot and investigate](/opencti/use/analyze/pivot/): follow its links to the infrastructure and the threat behind it.
3. [Handle incidents and requests in cases](/opencti/use/analyze/cases/): open an incident response case, gather the evidence and split the work into tasks.
4. [How indicators age and expire](/opencti/use/analyze/indicators-lifecycle/): why an indicator scores what it does, and when it leaves your detection tools.
:::
:::tab[CTI team leads]
You set the team's priorities, organize its work and automate what it does by hand.

1. [Follow your priority threats with PIRs](/opencti/use/analyze/pir/): tell the platform which threats matter to the business you serve.
2. [How intelligence gets into OpenCTI](/opencti/use/ingest/): choose the sources that feed the team, and how their data reaches the knowledge base.
3. [Automate with playbooks](/opencti/use/automate/playbooks/): triage, enrich and route new intelligence without anyone touching it.
4. [Build dashboards](/opencti/use/dashboards/): follow the team's work and the threats, and give each customer its own view.
:::
:::tab[SOC managers]
You get intelligence into every detection tool, with indicators that expire on time, and measure what your defenses catch.

1. [Share knowledge with your security tools](/opencti/use/share/feeds/): stream or serve indicators to the SIEM and the EDR, filtered on score and validity.
2. [How indicators age and expire](/opencti/use/analyze/indicators-lifecycle/): how scores fall and indicators are revoked.
3. [Age indicators with decay rules](/opencti/configure/decay/): tune how long each kind of indicator stays active.
4. [Measure your defenses against a threat](/opencti/use/analyze/security-coverage/): which techniques of a threat your defenses prevent and detect.
:::
:::tab[Administrators]
You decide who signs in, who sees what, and what each team may change.

1. [How access control works](/opencti/administer/access-control/): users, groups, roles, organizations and markings, and how they combine.
2. [Configure single sign-on](/opencti/administer/sso/): sign in with your identity provider and map its groups.
3. [Organize users in groups](/opencti/administer/groups/): give each team its roles, its markings and its default settings.
4. [Grant rights with roles](/opencti/administer/roles/): let analysts create without deleting, and keep administration to a few.
:::
:::tab[Platform engineers]
You deploy the XTM platform, which always runs OpenCTI with the other modules, and keep OpenCTI healthy and up to date.

1. [How OpenCTI works](/opencti/concepts/architecture/): the platform, the workers, the managers and the connectors, and what each one needs.
2. [Requirements](/deploy/plan/requirements/): what the host must provide, and the versions of the databases, queues and storage OpenCTI supports.
3. [Install with Docker Compose](/deploy/install/docker-compose/): the whole XTM platform, OpenCTI with every other module, behind your reverse proxy.
4. [Upgrade](/deploy/upgrade/): move to a new release and what to check before.
:::
:::tab[CISOs and executives]
You see what threatens the business and what the teams achieve, without the procedures.

1. [Build dashboards](/opencti/use/dashboards/): the threats targeting your sector this quarter, on one screen.
2. [Follow your priority threats with PIRs](/opencti/use/analyze/pir/): the threats ranked against the scope you care about, ready for a briefing.
3. [Measure your defenses against a threat](/opencti/use/analyze/security-coverage/): how much of a threat your defenses stop.
:::
::::

## Automate with playbooks

::::split
:::column
A playbook processes intelligence as it arrives: it filters, enriches and changes data, then saves it, shares it or tells the right people. No code, and every run is traced.

:::cards{cols=2}
  ::card[Which component filters, enriches, saves or notifies.]{icon=BookOpen title="Choose a playbook component" href="/opencti/use/automate/playbooks/choose-a-component/"}
  ::card[From an empty canvas to a running playbook.]{icon=ListChecks title="Create a playbook" href="/opencti/use/automate/playbooks/create/"}
:::
:::
:::column
::screenshot{id=opencti-playbook-canvas alt="A playbook that listens for new reports, adds their content, then routes hospitality-sector reports to be marked in progress, saved and sent to the CTI team, and notifies the duty analyst otherwise."}
:::
::::

:::xtm-one[AI agents in your playbooks]
A playbook can hand a step that filters cannot express, such as reading a report's description, to an XTM One AI agent: the **Transform with AI agent** component returns the bundle, changed or not, and the playbook carries on. See [Create a playbook](/opencti/use/automate/playbooks/create/).
:::

## Browse OpenCTI

::section-overview{id=opencti}
