---
title: How collectors measure what your security tools saw
description: How OpenAEV collectors read the alerts of your EDR, SIEM and other tools to score detection and prevention, the reference data other collectors import, and how to run them.
doc_type: explanation
products:
  - openaev
personas: [validation-engineer, platform-engineer, soc-analyst]
owner: docs-openaev
last_reviewed: 2026-11-07
migrated_from:
  - openaev:/usage/collectors/
---

A collector is the integration that reads data from another system for OpenAEV. Most collectors read your security tools, an endpoint detection and response (EDR) tool, a security information and event management (SIEM) system or an email gateway, to tell whether each attack of a simulation was detected or prevented: they fill the detection and prevention expectations. Others import what you build simulations from: attack techniques, payloads, vulnerabilities, assets and people. This page explains how a collector matches an alert to an inject, which collectors exist, and how to run them. Where collectors sit among agents, executors and injectors is in [How OpenAEV works](/openaev/concepts/architecture/#collectors).

::diagram{id=openaev-collectors-matching}

## How a collector scores detection and prevention

1. **The inject sets what to look for.** When an inject runs on an endpoint, OpenAEV creates its expectations, such as **Detection** and **Prevention**, for each agent that runs it. Each expectation carries signatures: the host name and IP addresses of the endpoint, the time window of the execution, and the name of the implant's process, `oaev-implant-<inject>-agent-<agent>`, which no other process has.
2. **The collector asks for the expectations it has not filled yet.** At each run, a collector of a security tool reads from the API the pending detection and prevention expectations.
3. **It searches its tool.** An EDR collector looks for alerts on that host whose parent process is the implant: the alert then belongs to this inject, on this endpoint. A SIEM collector relies on the EDR whose logs it gathers: it confirms in the SIEM what the EDR collector has matched, so the EDR collector must fill the expectation first.
4. **It fills the expectation.** A match marks the expectation detected, or prevented when the tool blocked the action, with the alert as its evidence. Each collector of a security tool is tied to a security platform, listed in :menu[Security platforms], and the result carries its name.
5. **What nobody filled expires.** A built-in collector, **Expectations Expiration Manager**, closes the expectations nobody filled before their expiration time: 6 hours by default for detection and prevention, the `openaev.expectation.technical.expiration-time` key of the [`openaev.expectation` block](/reference/openaev/configuration/#openaev-expectation). The attack then counts as **Not Detected**, or **Not Prevented**.

So a collector needs the inject to run through an implant, with the OpenAEV agent or an executor of your EDR, and the tool to keep the process tree of its alerts. How each expectation is scored and shown is in [Expectations](/openaev/use/expectations/).

## The collectors

The catalog, in :menu[Integrations], **Available** tab, lists the collectors the platform knows, by what they are for:

| Use | Collectors |
| --- | --- |
| Security response: fill detection and prevention | CrowdStrike, Microsoft Defender, Microsoft Defender for Office 365, Microsoft Sentinel, SentinelOne, Palo Alto Cortex XDR, Palo Alto Cortex XSOAR, Tanium Threat Response, Splunk Enterprise Security, Elastic Security, IBM QRadar, LogRhythm, NetWitness |
| AI security: score the defenses of AI applications | Cisco AI Defense, HiddenLayer AIDR, Lakera Guard, Palo Alto Prisma AIRS, Prompt Security |
| Attack knowledge | MITRE Att&ck, MITRE ATLAS, NVD NIST (vulnerabilities) |
| Payloads for the Threat Arsenal | Atomic Red Team, OpenAEV Library |
| Assets, as endpoints without an agent | Microsoft Intune, Microsoft Azure, Amazon Web Service Resources, XTM One (its AI agents, as AI targets) |
| Players | Microsoft Entra, Google Workspace |

Two collectors are built into the platform and need nothing: **Expectations Expiration Manager**, above, and **Expectations Vulnerability Manager**, which scores the vulnerability expectations against the findings.

::screenshot{id=openaev-integrations-collectors-deployed alt="The Deployed tab of Integrations filtered on collectors: Meridian EDR, the CrowdStrike Falcon collector, started next to the built-in Expectations Expiration Manager and Expectations Vulnerability Manager."}

## Run a collector

Collectors other than the two built-in ones are Python programs, published as `openaev/collector-*` images. Run one in either of two ways:

- **With XTM Composer**, from :menu[Integrations], **Available** tab: open the collector, click **Deploy**, fill in its settings and start it. This requires an Enterprise Edition license, the **Manage tenant settings** capability and a running [XTM Composer](/deploy/integrations/composer/). XTM Composer then runs the container and shows its logs.
- **Yourself**, with Docker or Python, with `OPENAEV_URL`, `OPENAEV_TOKEN` and a `COLLECTOR_ID` of your own, plus the settings of the collector in its `README.md`: see [Run an integration without XTM Composer](/deploy/integrations/manual/). With several tenants, also set `OPENAEV_TENANT_ID`.

A collector deployed from the catalog does not sign in with your token. OpenAEV gives it the token of the tenant's **Catalog connectors** service account, created at the first deployment in the tenant, in the group of the same name. Its role holds what the integrations call: **Manage tenant settings**, **Manage assessment**, **Manage documents**, **Manage security platforms**, **Manage tags**, **Manage assets**, **Manage teams & persons**, **Manage payloads**, **Manage threat arsenal**, **Delete threat arsenal** and **Resolve inject secret**. Its role and group cannot be edited or deleted. Every integration the tenant deployed from the catalog, injectors included, shares this account, so it keeps working when the person who deployed it leaves. The integrations deployed before the upgrade move to this account when the platform starts. To replace the token, a user with **Manage tenant users, groups and roles** calls `POST /api/catalog-connector/token/rotate`: OpenAEV revokes the old token and pushes the new one to every integration the tenant deployed from the catalog.

In the catalog, the settings of Elastic Security show how it matches alerts. The description of `ELASTIC_QUERY_TEMPLATE` gives the collector's built-in query: leave the setting empty to keep that query as the collector is updated, or copy it and edit it to fit your fields. `ELASTIC_EVENTS_INDEX`, the index of process events the collector reads to find the implant behind an alert, defaults to `logs-windows.sysmon_operational-*,logs-endpoint.events.process-*,logs-system.security-*`: Sysmon, Elastic Defend and the Windows Security channel, with process creation auditing (event 4688). A host with no process-creation event in that index is matched on its host name, IP addresses and time window only.

A collector only opens outbound connections, to the OpenAEV URL and to the tool it reads: OpenAEV never connects to it. So it can run next to the tool, for example beside an on-premises SIEM while OpenAEV runs as a service. In :menu[Integrations], **Deployed** tab, each collector shows whether it runs and when the platform last heard from it.

A collector you **Stop** in :menu[Integrations] no longer counts for the injects that run afterwards, from the moment it is stopping. Their detection and prevention expectations wait only for the other collectors of the tenant. When no other collector answers that type of expectation for the security platforms the inject expects, the expectation fails at once, without waiting 6 hours to expire. It reads **No automated security platform detected**. Before you launch, the **Security System Collector** health check of the scenario or inject warns about such an expectation. A collector you run yourself, without a deployment in :menu[Integrations], always counts.

:::example[Score the Velvet Harbor simulation with Meridian EDR]
Meridian EDR, the endpoint detection of Grand Meridian Hotels, is CrowdStrike Falcon. The platform engineer deploys the **CrowdStrike** collector from the catalog, with a Falcon API client that reads detections, and it shows as started in the **Deployed** tab.

The validation engineer runs the Velvet Harbor simulation on the front-desk workstations. Its PowerShell (T1059.001) inject runs on `fd-paris-01` at 02:14, through the implant `oaev-implant-<inject>-agent-<agent>`. Falcon raises a detection on `fd-paris-01` whose parent process is that implant, and kills the process. The CrowdStrike collector matches it on its next run: `fd-paris-01` reads **Detected** and **Prevented**, with the Falcon alert as evidence. On `fd-paris-02`, where the policy only detects, it reads **Detected**, and **Not Prevented** once the prevention expectation expires 6 hours later: a gap the SOC closes in the Falcon prevention policy.
:::

:::xtm-one[Turn an undetected attack into a detection rule]
When OpenAEV is connected to XTM One, an AI agent can draft detection rules from an inject that ran, from the commands and behavior of its actions: the rule your SOC was missing when a collector found nothing. The XTM One collector also imports the AI agents of XTM One as targets, so simulations can test them. Connecting OpenAEV to XTM One is in [Connect the modules](/deploy/configure/connect-modules/).
:::

## What's next

- [Expectations](/openaev/use/expectations/): how detection, prevention and the other expectations are scored.
- [Choose between the OpenAEV agent and your EDR](/openaev/use/executors/): the executors whose implants collectors match.
- [How integrations run](/deploy/integrations/how-they-run/): deploy collectors with or without XTM Composer.
- [Build a collector](/develop/build/collector/): write your own.
