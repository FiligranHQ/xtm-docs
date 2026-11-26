---
title: Glossary
description: The terms of the XTM platform and of its modules, each in one sentence with a link to the page that explains it, from integration and AI agent to container, draft and decay.
doc_type: reference
products:
  - suite
personas: [evaluator, cti-analyst, admin]
owner: docs-platform
last_reviewed: 2026-11-01
sidebar_position: 3
no_screenshot: A glossary defines the words of the platform and describes no screen of its own.
---

This glossary gives the words you meet across the XTM platform and its modules, each in one sentence, with a link to the page that explains it. Some words mean different things in different modules, such as an organization in OpenCTI and a tenant in OpenAEV: the glossary says which module a word belongs to. The terms are grouped by subject, then listed alphabetically.

::diagram{id=start-glossary-integrations}

## The platform

| Term | Meaning |
| --- | --- |
| AI agent | An agent of XTM One: it reads what the modules know, decides what matters and acts, such as summarizing or drafting, under the governance you define. You talk to it, run it in a flow, or call it from OpenCTI and OpenAEV. See [XTM One](/xtm-one/). Not to be confused with an OpenAEV agent |
| Ask Ariane | The assistant of OpenCTI, OpenAEV and OpenCRQ, opened from their top bar. An AI agent of XTM One answers it: by default the **CTEM Assistant**, which hands a question about one module to the assistant of that module, such as the **OpenCTI Assistant**. Some screens call it **Use Ariane** (OpenAEV) or **Agentic AI (Ariane Assistant)** (the XTM Hub settings of OpenCTI). See [XTM One in OpenCTI](/xtm-one/use/in-opencti/) |
| Community Edition | The free, open source edition of a module, under the Apache License 2.0. See [Editions and licensing](/start/editions/) |
| CTEM | Continuous threat exposure management: the cycle of knowing your threats, testing your exposure to them and fixing what matters most, which the modules run together. See [Run a CTEM cycle across the platform](/workflows/ctem-cycle/) |
| CTEM Assistant | The default AI agent of XTM One: it answers as the **General Assistant** in the chat, and routes each request to the right specialist agent, such as the **OpenCTI Assistant**, the **OpenAEV Assistant** or the **OpenCRQ Assistant**. See [Get answers from an agent in the chat](/xtm-one/use/chat/) |
| Enterprise Edition | The edition that adds features such as playbooks, segregation, single sign-on and audit, turned on by a license from Filigran. See [Editions and licensing](/start/editions/) |
| Instance | One deployed module, such as your OpenCTI instance. XTM One's screens call the instances registered with it its platforms. See [Connect the modules to each other](/deploy/configure/connect-modules/) |
| LTS | Long-term support: a release line that receives only critical and security fixes for one year, for organizations that cannot upgrade often. It needs an LTS license. See [Release lifecycle](/deploy/upgrade/lifecycle/) |
| Module | One of the four applications of the XTM platform: XTM One, OpenCTI, OpenAEV and OpenCRQ. XTM Composer is a component, and XTM Hub a portal: neither is a module. See [The XTM stack: modules and shared services](/deploy/plan/architecture/) |
| OpenAEV | The adversarial exposure validation module: it runs realistic attack scenarios against your environment and measures how your defenses respond. See [OpenAEV](/openaev/) |
| OpenCRQ | The cyber risk quantification module: it prices each cyber risk in money, as the loss it can be expected to cause over a year. See [OpenCRQ](/opencrq/) |
| OpenCTI | The cyber threat intelligence module: it stores what you know about threats as a knowledge graph, and shares it with your teams and tools. See [OpenCTI](/opencti/) |
| XTM Hub | Filigran's community portal for ready-made content, such as dashboards, playbooks and feeds, with free trials, the roadmap and the Academy. See [Connect to XTM Hub](/opencti/configure/xtm-hub/) |
| XTM One | The AI layer at the center of the XTM platform, Agentic Threat Management: the module whose AI agents work across OpenCTI, OpenAEV and OpenCRQ. Name its features by what they do, not "Copilot". See [XTM One](/xtm-one/) |
| XTM platform | The whole offering of Filigran for eXtended Threat Management: its four modules, XTM Composer and XTM Hub. See [The XTM stack: modules and shared services](/deploy/plan/architecture/) |
| xtm-docker | The official Docker Compose deployment of the XTM platform, which starts the whole stack: XTM One, OpenCTI, OpenAEV, OpenCRQ, XTM Composer and the shared services. See [Run the XTM platform locally](/deploy/quick-start/) |

## Integrations

| Term | Meaning |
| --- | --- |
| Catalog | The list of integrations a platform can deploy, in its **Integrations** menu. See [Import with connectors](/opencti/use/ingest/connectors/) |
| Collector | An OpenAEV integration that reads data from other systems: the alerts of your security tools, to measure the response to injects, or assets and threat intelligence. See [How OpenAEV works](/openaev/concepts/architecture/) |
| Connector | An OpenCTI integration that imports, enriches, streams or exports data. See [Import with connectors](/opencti/use/ingest/connectors/) |
| Executor | An OpenAEV integration that runs the actions of injects on endpoints: the OpenAEV agent, or a third-party agent such as CrowdStrike Falcon driven through its vendor's API. See [How OpenAEV works](/openaev/concepts/architecture/) |
| Injector | An OpenAEV integration that delivers injects to people and systems, such as an email, an HTTP request or an action on endpoints. See [How OpenAEV works](/openaev/concepts/architecture/) |
| Integration | Any component that connects a module to another system. Connectors, collectors, injectors and executors are kinds of integration. See [How integrations run](/deploy/integrations/how-they-run/) |
| OpenAEV agent | The program OpenAEV installs on your endpoints: it fetches the jobs of a simulation and starts an implant that runs each one. See [OpenAEV agents](/openaev/use/agents/) |
| XTM Composer | The component that deploys the integrations you pick in the catalog of OpenCTI or OpenAEV, runs them as containers and reports their status. Some screens of the modules name it **Integration Manager**. See [XTM Composer](/deploy/integrations/composer/) |

## OpenCTI knowledge

| Term | Meaning |
| --- | --- |
| Author | The identity a piece of knowledge is attributed to, such as the vendor of a feed: each source writes under its own author. See [How intelligence gets into OpenCTI](/opencti/use/ingest/) |
| Confidence | How credible a piece of information is, whoever said it, from 0 to 100. See [Reliability and confidence](/opencti/concepts/reliability-confidence/) |
| Container | An object that holds other objects sharing a context, such as a report, a grouping or a case. See [Containers](/opencti/concepts/containers/) |
| Decay | The fall of an indicator's score as it ages, following a decay rule, until the indicator is revoked. See [How indicators age and expire](/opencti/use/analyze/indicators-lifecycle/) and [Age indicators with decay rules](/opencti/configure/decay/) |
| Deduplication | OpenCTI never stores the same object twice: an object that already exists is updated, which is called an upsert. See [Deduplication](/opencti/concepts/deduplication/) |
| Entity | A node of the knowledge graph, such as an Intrusion Set, a Malware or a Sector. See [Data model](/opencti/concepts/data-model/) |
| Indicator | A detection pattern, such as a STIX pattern or a Sigma or YARA rule, with a score and a validity period, that detection tools consume. See [How indicators age and expire](/opencti/use/analyze/indicators-lifecycle/) |
| Inference | A relationship that no one wrote: the rules engine deduced it from existing knowledge. See [How OpenCTI infers new relationships](/opencti/use/enrich/inferences/) |
| Marking | A label of sensitivity, such as TLP:AMBER or PAP:RED, that hides an object from anyone whose groups are not cleared for it. See [Restrict access with markings](/opencti/administer/markings/) |
| Max confidence level | The confidence above which a user, a connector or a feed cannot create or change knowledge. See [Reliability and confidence](/opencti/concepts/reliability-confidence/) |
| Nested object | Part of the context a STIX object carries inside itself, such as its author or its markings, which OpenCTI stores as a relationship to a separate object. See [Nested objects](/opencti/concepts/nested-objects/) |
| Observable | A technical fact seen in data, such as an IP address, a domain or a file hash. See [Explore observations](/opencti/use/explore/observations/) |
| PIR | Priority intelligence requirement: what your organization most needs to know, which OpenCTI answers by flagging and scoring the threats that match. See [Follow your priority threats with PIRs](/opencti/use/analyze/pir/) |
| Relationship | An edge of the knowledge graph between two objects, such as `uses` or `targets`. See [Data model](/opencti/concepts/data-model/) |
| Reliability | How much you trust a source, such as a vendor or a partner. See [Reliability and confidence](/opencti/concepts/reliability-confidence/) |
| Report | The most common container: what one source says about a subject, with the threats, malware and indicators it names. See [Explore analysis](/opencti/use/explore/analysis/) |
| Rules engine | The engine that derives new knowledge from existing knowledge with inference rules an administrator turns on. See [Turn inference rules on or off](/opencti/configure/rules/) |
| Security coverage | For one threat, how much of its attack patterns and vulnerabilities your defenses prevent and detect, measured by OpenAEV or entered by hand. See [Measure your defenses against a threat](/opencti/use/analyze/security-coverage/) |
| Sighting | A record that an indicator or an observable was seen, by whom and how many times. See [Explore events](/opencti/use/explore/events/) |
| STIX | Structured Threat Information Expression 2.1, the standard OpenCTI follows to model and exchange threat intelligence. See [Data model](/opencti/concepts/data-model/) |

## OpenCTI work and automation

| Term | Meaning |
| --- | --- |
| Background task | One action applied to many objects at once, such as relabeling or deleting hundreds of entities. See [Change many objects at once](/opencti/use/automate/background-tasks/) |
| Case | A container where your team works an incident response, a request for information or a request for takedown, with its tasks. See [Handle incidents and requests in cases](/opencti/use/analyze/cases/) |
| Custom dashboard | A screen of widgets, computed from the knowledge base each time it opens, that answers a standing question. See [Build dashboards](/opencti/use/dashboards/) |
| Dissemination list | A named list of email addresses, outside OpenCTI, that receives your finished intelligence documents. See [Send intelligence by email](/opencti/use/share/dissemination-lists/) |
| Draft | A workspace where you create, change and delete knowledge without touching the knowledge base, until someone approves it. See [Prepare changes in a draft](/opencti/use/ingest/drafts/) |
| Entity workflow | The list of statuses an entity type goes through, such as `NEW`, `IN_PROGRESS` and `CLOSED`. See [Track the progress of reports and cases](/opencti/use/automate/entity-workflows/) |
| Exclusion list | Values you know to be legitimate, such as your own address ranges, that no indicator may contain. See [Keep benign values out of indicators](/opencti/configure/exclusion-lists/) |
| Feed | A source OpenCTI polls on a schedule, such as a TAXII collection or an RSS, CSV or JSON feed; OpenCTI also publishes its own feeds. See [How intelligence gets into OpenCTI](/opencti/use/ingest/) and [Share knowledge with your security tools](/opencti/use/share/feeds/) |
| Fintel template, fintel design | A template writes what a finished intelligence (FINTEL) document says, per entity type; a design gives it your logo and colors. See [Brand your PDF reports](/opencti/configure/fintel-designs/) |
| Investigation | A private graph workspace where you expand an alert's observables along their relationships. See [Pivot and investigate](/opencti/use/analyze/pivot/) |
| Live stream | The events of the knowledge base, published as they happen, that another platform or a detection tool follows. See [Share knowledge with your security tools](/opencti/use/share/feeds/) |
| Notifier, trigger, digest | A trigger watches the knowledge base for events, a digest groups its alerts on a schedule, and a notifier delivers them: in the platform, by email or to a webhook. See [Get alerted when knowledge changes](/opencti/use/automate/notifications/) |
| Playbook | An automation drawn on a canvas: it starts on an event, filters, enriches or changes the data step by step, and saves or sends the result. See [Automate with playbooks](/opencti/use/automate/playbooks/) |
| Retention policy | A rule that deletes data that has not changed for a given time. See [Delete old data automatically](/opencti/configure/retention/) |
| TAXII | Trusted Automated Exchange of Intelligence Information, the protocol to exchange STIX collections. See [Import from a TAXII server](/opencti/use/ingest/taxii-feed/) |
| Workbench | An analyst workbench holds the objects found in one imported file, to correct them before import; drafts are now the default way. See [Review what a file import found](/opencti/use/ingest/workbench/) |

## Access and sharing

| Term | Meaning |
| --- | --- |
| Authorized members | The users, groups or organizations an object is restricted to, each with an access level. See [Restrict one object to chosen people](/opencti/administer/sharing/) |
| Capability | The right to one kind of action, such as **Manage parameters**. Roles group capabilities. See [Grant rights with roles](/opencti/administer/roles/) |
| Group | How users get their rights in OpenCTI: its roles, its markings and its confidence level. See [Organize users in groups](/opencti/administer/groups/) |
| Organization (OpenCTI) | A segregation unit of OpenCTI, such as a business unit or a customer, whose users see only the knowledge shared with it. One knowledge base, not separate copies: an organization is not a tenant. See [Separate data between organizations](/opencti/administer/organizations/) |
| Role | A named set of capabilities that decides what its users can do. See [Grant rights with roles](/opencti/administer/roles/) |
| Service account | A user for a machine, such as a connector or a script, that authenticates with an API token only. See [Create accounts for integrations](/opencti/administer/service-accounts/) |
| Single sign-on | Signing in through your identity provider, with OpenID Connect, SAML or LDAP. See [Configure single sign-on](/opencti/administer/sso/) |
| Tenant (OpenAEV) | A separate space of an OpenAEV instance in multi-tenancy, an Enterprise Edition feature; an integration that serves one tenant names it in `OPENAEV_TENANT_ID`. Not an OpenCTI organization. See [Multi-tenancy](/openaev/administer/multi-tenancy/) |

## OpenAEV and OpenCRQ

| Term | Meaning |
| --- | --- |
| Annualized loss expectancy | In OpenCRQ, the loss in US dollars that a risk scenario can be expected to cause over a year. See [How OpenCRQ quantifies risk](/opencrq/concepts/how-risk-is-quantified/) |
| Atomic testing | In OpenAEV, one inject run on its own, outside a scenario. See [Atomic testing](/openaev/use/atomic-testing/) |
| Expectation | In OpenAEV, what should happen after an inject, such as the attack being detected or prevented, which collectors or people fill in. See [Injects and expectations](/openaev/concepts/injects-expectations/) |
| Inject | In OpenAEV, one action of a simulation, delivered at its time to people or systems: an email, a media article, an attack technique on endpoints. See [Injects and expectations](/openaev/concepts/injects-expectations/) |
| Scenario, simulation | In OpenAEV, a scenario is a reusable sequence of injects, and a simulation is one run of it against your environment. See [Scenarios and simulations](/openaev/concepts/scenarios-simulations/) |

## Operations

| Term | Meaning |
| --- | --- |
| Manager | A background process of the OpenCTI platform, such as the rules, history, notification or retention manager. See [How OpenCTI works](/opencti/concepts/architecture/) |
| Observability metrics | The OpenTelemetry metrics a platform exposes to your own monitoring system, such as Prometheus. Not usage telemetry. See [Metrics](/deploy/operate/metrics/) |
| Platform identifier | The unique identifier of an OpenCTI instance, which support, the XTM Hub and licenses tied to one instance ask for. See [Set the platform parameters](/opencti/configure/parameters/) |
| Shared services | The databases, queue and storage the modules of the stack use: Elasticsearch, Redis, RabbitMQ, S3 storage and PostgreSQL. See [The XTM stack](/deploy/plan/architecture/) |
| Usage telemetry | The anonymous usage data a platform sends to Filigran, without intelligence or personal data. Not observability metrics. See [Usage telemetry and privacy](/opencti/configure/telemetry/) |
| Worker | A process that writes the data connectors send into the OpenCTI knowledge base. See [How OpenCTI works](/opencti/concepts/architecture/) |
