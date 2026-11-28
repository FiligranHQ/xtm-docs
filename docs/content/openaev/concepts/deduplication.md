---
title: How OpenAEV recognizes what it already has
description: When OpenAEV matches an incoming endpoint, finding, payload, tag, person or STIX bundle to an object it already has, by which properties, and what it then updates, merges or refuses.
doc_type: explanation
products: [openaev]
personas: [admin, validation-engineer]
owner: docs-openaev
last_reviewed: 2026-11-27
---

OpenAEV has no single rule that makes two objects one. Each kind of object is recognized by properties of its own, and only some ways in look for an existing object: agents and executors that report machines, collectors that bring payloads and vulnerabilities, imports of scenarios, the security coverages of OpenCTI. Most of what you create by hand, an endpoint, an organization, a scenario, is a new object, and some names the platform refuses twice. Knowing which way in matches what tells you why a machine shows once with two agents, why a tag you typed in capitals is lowercase, and why an imported scenario reuses your teams but brings new injects. Every match stays inside one tenant.

::diagram{id=openaev-deduplication-endpoints}

## Endpoints: one machine, one endpoint

A machine reaches OpenAEV through the OpenAEV agent installed on it, through the EDR that manages it, or both. OpenAEV keeps one endpoint per machine, with one agent per way it is reached:

- **An agent it already knows.** Each agent reports the identifier it was installed with. When an agent with that identifier exists, with the same user, the same mode (service or session) and the same privilege (administrator or standard), it is that agent, and it is updated. With another user, mode or privilege, it is a new agent on the same endpoint: the OpenAEV agent installed as a service and as a user session shows twice on one endpoint.
- **A machine it already knows.** A new agent joins the endpoint that has at least one of its MAC addresses. The hostname is not compared, so two machines with the same hostname stay two endpoints. MAC addresses are compared in lowercase, without separators, and the addresses no network card has, such as all zeros or the broadcast address, are ignored.
- **An EDR's hosts.** The CrowdStrike Falcon, SentinelOne, Tanium, Microsoft Defender for Endpoint and Palo Alto Cortex executors sync the hosts they manage: a host is matched by its device identifier among the executor's own agents, then by a shared MAC address. An endpoint where the OpenAEV agent already runs gets the EDR's agent as a second agent. Each host group of CrowdStrike Falcon becomes an asset group, recognized by the group's identifier. See [Choose between the OpenAEV agent and your EDR](/openaev/use/executors/).
- **Caldera's agents** are recognized by their identifier, then by the same hostname and at least one IP address in common, when exactly one endpoint has them.

When an endpoint is matched, its hostname and architecture take the reported values, its IP and MAC addresses gain the new ones, and its agent gets its last seen date. CrowdStrike Falcon never changes the hostname, which it cuts at 15 characters, and Palo Alto Cortex never changes the architecture. The endpoint gets the tag `source:` followed by the name of the executor, in lowercase.

An endpoint you create in :menu[Assets], or import from a CSV file, is always a new one, even with the hostname of another. To feed endpoints from a script without duplicates, use the API's upsert, which matches by external reference, then by the same hostname and an IP address, then by the same hostname and a MAC address: see [REST API](/develop/apis/rest/).

## Findings: once per inject

A finding is recorded per inject: the same field, type and value reported again by the same inject updates the finding, its name and its date, and adds the asset it was found on. The same value reported by another inject, or by the next run of the scenario, whose injects are copies, is another finding.

:menu[Findings], the **Findings** tab of a scenario or a simulation and the **Findings** section of an endpoint show one line per type and value. The page of a finding gathers every finding of that type and value in the tenant: its **Occurrences**, **First seen** and **Last seen**, and the assets, teams and persons it impacts. So a credential found on the same workstation every week is one line, with one occurrence per run.

## What collectors and connectors bring

| Object | Recognized by | When it comes again |
| --- | --- | --- |
| Payload of a collector | Its external identifier, given by the collector | Its properties, tags and attack patterns are replaced; its domains are added up |
| Vulnerability | Its CVE identifier, in any case | Every attribute is replaced, its weaknesses (CWE) too |
| Attack pattern | Its external identifier, such as `T1059.001` | Its name, description, platforms, kill chain phases and parent are replaced |
| Kill chain phase | Its STIX identifier, then its kill chain and short name | Its name, description and short name are replaced |
| Inject type of an injector | Its identifier, when the injector registers | Its labels, form and attack patterns are replaced; a type the injector no longer declares is deleted |
| Security platform | Its external reference, then its name in any case and its type | Updated. Two platforms of one type cannot share a name |

The author of a collector's payloads is an organization, recognized by its name in any case. See [How collectors measure what your security tools saw](/openaev/use/collectors/).

## Names the platform keeps unique

| Object | What makes it the same | What happens |
| --- | --- | --- |
| Tag | Its name, always stored in lowercase | An import, a CSV file or a collector that brings `Hotel-Sector` reuses `hotel-sector` and keeps its color. Creating a tag of that name again is refused |
| Person | Their email address, in any case | Adding a person with an email that exists adds that person to the tenant, unchanged: an account is one person across tenants |
| Team | Its name, in any case | A second global team with the name of another is refused, and so is a contextual team with the name of another of the same scenario or simulation |
| Document | Its content | Uploading a file whose content exists reuses the document and adds the new tags to it |
| Organization | Nothing, when you create one | Two organizations may share a name; an import or a collector reuses the one of the same name, in any case |

## What an import or a duplicate creates

Importing a scenario or a simulation, deploying one from XTM Hub, or loading the starter pack always creates a new scenario or simulation, with new injects and articles. An import is named after the original followed by ` (Import)`; the starter pack keeps its names. What they point to is reused when the tenant has it: tags by name, attack patterns and kill chain phases by identifier, organizations, channels and challenges by name, persons by email, global teams by name, payloads by identifier or by content. Importing the same file twice gives two scenarios that share the same teams and assets. See [Import a scenario or injects from a file](/openaev/use/scenarios/import/).

**Duplicate**, on a scenario, an inject or a payload, creates a new object named after the original followed by ` (duplicate)`. A duplicated scenario gets its own copies of its injects, contextual teams, articles and variables, and shares the rest. A duplicated payload loses the identifier of its collector, so the collector never updates the copy.

## Security coverages from OpenCTI

A security coverage that OpenCTI sends is recognized by its STIX identifier, and is the same scenario from one update to the next. OpenAEV ignores a bundle identical to the last one, and a bundle not newer than the last it processed: OpenCTI's enrichment shows the request and its message. A newer bundle updates the scenario: its name, description, severity and tags take the coverage's values, the injects that still test a technique of the coverage on a platform it names are kept with their targets, the others are deleted, and only the missing ones are built. See [Turn an OpenCTI security coverage into a scenario](/openaev/use/scenarios/from-opencti/).

:::example[One front-desk workstation, two agents]
The platform engineer of Grand Meridian Hotels installs the OpenAEV agent as a service on `fd-paris-07` on 8 January 2026. The next day, Meridian EDR, a CrowdStrike Falcon tenant, syncs its host group `Front desk - Paris`, which holds `fd-paris-07`.

- The Falcon host has a MAC address of the endpoint the OpenAEV agent created: the endpoint gets a second agent, of the CrowdStrike executor, and the tag `source:` followed by the executor's name in lowercase. :menu[Assets] shows `fd-paris-07` once.
- The asset group `Front desk - Paris` is created; the next sync updates its members instead of creating another.
- A night auditor's laptop, renamed `fd-paris-07` by mistake, has none of these MAC addresses: it stays an endpoint of its own, and the injects aimed at the asset group reach the right machine.
:::

## What's next

- [How OpenAEV keeps what you test and what it finds](/openaev/concepts/data-model/)
- [Install the OpenAEV agent on your endpoints](/openaev/use/agents/)
- [Choose between the OpenAEV agent and your EDR](/openaev/use/executors/)
- [Add the assets your simulations target](/openaev/use/targets/assets/)
