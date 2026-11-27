---
title: Collectors
description: How an OpenAEV collector registers with its security platform, reads the expectations it has not answered, and writes verdicts and alert traces, as of OpenAEV 3.261029.0.
doc_type: explanation
products:
  - openaev
personas: [developer, platform-engineer]
owner: docs-develop
last_reviewed: 2026-11-09
migrated_from:
  - openaev:/integrations/collectors/collectors/
---

A collector is the OpenAEV integration that reads another system for the platform. Most collectors read a security tool, such as an endpoint detection and response (EDR) tool or a security information and event management (SIEM) system, to tell whether each attack of a simulation was detected or prevented. A collector is a program of its own, in any language, that only calls the REST API of the platform. This page explains that protocol as OpenAEV 3.261029.0 implements it: the registration and the security platform, the expectations a collector reads, the signatures it matches, and the verdicts it writes. To write one step by step, follow [Build an OpenAEV collector](/develop/build/collector/); to choose among the existing collectors, read [How collectors measure what your security tools saw](/openaev/use/collectors/).

::diagram{id=develop-integration-model-collectors}

## What a collector does

A collector of a security tool runs in a loop, every period it chooses:

1. **It registers**, with the security platform it speaks for. Each registration is also its heartbeat.
2. **It reads the expectations it has not answered yet**: the detection and prevention expectations of the injects that ran on endpoints.
3. **It searches its tool** for the alerts that match each expectation's signatures, such as the name of the implant's process.
4. **It writes a verdict** on each expectation it can answer, and the alerts it relied on as traces.
5. **It stamps its last execution**, which the interface shows.

Everything goes from the collector to OpenAEV: the platform never calls a collector, and no RabbitMQ queue is involved. Filigran's `pyoaev` Python library implements this loop for the collectors of the [OpenAEV-Platform/collectors](https://github.com/OpenAEV-Platform/collectors) repository; the sections below describe it from the platform side.

Other collectors import reference data instead, such as MITRE ATT&CK, Atomic Red Team payloads, vulnerabilities, assets or players. They register the same way, then write through the API of what they import, and read no expectation.

## The security platform

A verdict carries the name of a security platform, the tool that saw the attack, listed in :menu[Security platforms]. A collector creates or finds its own with `POST /api/tenants/<tenant id>/security_platforms/upsert`:

| Field | What it does |
| --- | --- |
| `asset_name` | The name results show, such as `Meridian SOC` |
| `security_platform_type` | `EDR`, `XDR`, `SIEM`, `SOAR`, `NDR`, `ISPM`, `EMAIL_SECURITY`, `LLM_FIREWALL`, `AI_GATEWAY` or `VULNERABILITY_SCANNER` |
| `asset_external_reference` | The collector's identifier, by convention: the upsert finds the platform by this reference first |
| `asset_description`, `asset_tags` | Optional |

The upsert looks for a platform with the same external reference, then with the same name and type, and creates one only when neither exists. So a collector redeployed with a new identifier takes over the platform of its predecessor. The answer is the security platform; its `asset_id` goes into the registration.

## Registration

The collector posts a multipart request to `POST /api/tenants/<tenant id>/collectors`, with the token of its user in `Authorization: Bearer <token>`. The `input` part is a JSON document with the content type `application/json`; an optional `icon` part, a PNG image, becomes its icon. Without the tenant in the path, the platform needs the user to reach a single tenant, and refuses the registration otherwise.

| Field of `input` | Required | What it does |
| --- | --- | --- |
| `collector_id` | Yes | The identifier of this collector, a UUIDv4 you choose once and keep. It is the `source` of its verdicts |
| `collector_name` | Yes | The name the interface and the results show |
| `collector_type` | Yes | A stable technical name, such as `openaev_crowdstrike`. The platform creates the collector type the first time it sees it, and names the icon after it |
| `collector_period` | No | The time between two runs. The platform stores it at the first registration and never changes it afterwards |
| `collector_security_platform` | No | The `asset_id` of its security platform. Without it, the results carry no security platform |
| `collector_author` | No | The organization the payloads and contracts it imports are attributed to. Without it, the collector's name |

A registration with a known `collector_id` in the same tenant updates the name, the type and the security platform. The platform recognizes a registration that changes nothing and keeps it out of the audit log, so register at every run: the interface shows an external collector as running while its last registration or its last execution is less than two minutes old.

## Reading the expectations

The collector reads the expectations no source has answered for it yet, the ones without a result from its `collector_id`:

| Route, under `/api/tenants/<tenant id>/injects/expectations/` | Returns |
| --- | --- |
| `assets/<collector id>` | The detection and prevention expectations of agents. With `?expiration_time=<minutes>`, only those created in the last minutes |
| `detection/<collector id>`, `prevention/<collector id>` | One type only |
| `ai/<collector id>` | The detection and prevention expectations of AI targets, which have no agent, for the collectors of AI defense tools |

Each route returns at most 10,000 expectations, the oldest first. An expectation another collector answered stays in the list of yours until yours answers it too: each source gives its own result, and the platform combines them.

Each expectation is a JSON object; the keys a collector needs:

| Key | What it holds |
| --- | --- |
| `inject_expectation_id` | The identifier to write the verdict on |
| `inject_expectation_type` | `DETECTION` or `PREVENTION` |
| `inject_expectation_signatures` | What to look for: a list of `{"type": ..., "value": ...}` |
| `inject_expectation_inject`, `inject_expectation_agent`, `inject_expectation_asset` | The inject, the agent and the endpoint it is about |
| `inject_expectation_created_at`, `inject_expiration_time` | When it was created, and how long it waits for a result, in seconds |
| `inject_expectation_expected_security_platforms` | The types of security platform expected to answer, such as `EDR`. Empty: any |

### The signatures of an expectation

| Type | Value |
| --- | --- |
| `parent_process_name` | The name of the process that ran the attack on the endpoint: `oaev-implant-<inject id>-agent-<agent id>` for the OpenAEV implant, `<injector type><inject id>-agent-<agent id>` for an external injector. No other process has this name, so an alert whose process tree holds it belongs to this inject on this endpoint |
| `source_ipv4_address`, `source_ipv6_address` | The addresses of the endpoint that ran the attack |
| `target_ipv4_address`, `target_ipv6_address`, `target_hostname_address` | The address or host name the attack targeted, when the inject has one |
| `start_date`, `end_date` | When the implant started and ended on this agent, added as it reports |
| `ai_request_marker`, `ai_target_endpoint` | The marker of an AI inject's request and the AI endpoint it called |

An EDR collector looks for alerts on that host whose parent process is the implant's. A SIEM collector can match addresses and the time window instead.

## Writing the verdicts

One verdict, with `PUT .../injects/expectations/<expectation id>`, or many at once, with `PUT .../injects/expectations/bulk` and `{"inputs": {"<expectation id>": {...}, ...}}`. Each verdict is:

| Field | Required | What it does |
| --- | --- | --- |
| `collector_id` | Yes | The collector that answers. The bulk route reads the collector of the first verdict for all of them |
| `is_success` | Yes | `true`: detected, or prevented for a prevention expectation. `false`: not detected, or not prevented |
| `metadata` | No | Text values kept with the result, such as the alert's identifier |

The platform turns `is_success` into the result the interface shows and into a score against the expected score, writes it on the agent's expectation, then recomputes the endpoint's and the asset group's. Only technical expectations, detection, prevention and vulnerability, accept a collector's verdict; the bulk route skips the others and the identifiers it does not know.

The alerts go to `POST .../inject-expectations-traces/bulk`, as `{"expectation_traces": [...]}`, each with `inject_expectation_trace_expectation`, `inject_expectation_trace_source_id` (the collector), `inject_expectation_trace_alert_name`, `inject_expectation_trace_alert_link` and `inject_expectation_trace_date`. The link lets an analyst open the alert in the tool from the inject's result.

Last, `PUT /api/tenants/<tenant id>/collectors/<collector id>` with `{"collector_last_execution": "<ISO 8601 date>"}` stamps the run.

## What no collector answers

A collector does not need to write a negative verdict for an attack it never saw. The built-in **Expectations Expiration Manager** closes every expectation still without a result when its expiration time is over: 6 hours by default for detection and prevention, the `openaev.expectation.technical.expiration-time` key of the [`openaev.expectation` block](/reference/openaev/configuration/#openaev-expectation). The attack then reads **Not Detected** or **Not Prevented**. Write `is_success: false` only when the tool tells you something definite, such as an alert that shows the action ran and was not blocked.

## The user of a collector

| Call | Capability |
| --- | --- |
| Register and stamp the last execution | **Manage tenant settings**, which allows creating and writing collectors |
| Create its security platform | **Manage security platforms** |
| Read expectations, write verdicts and traces | **Access assessment** to read, **Manage assessment** to write |

Give each collector a user of its own; the capabilities are in the [OpenAEV capabilities reference](/reference/openaev/capabilities/). A call the user has no right to make answers 404, not 403.

## How collectors are deployed

A collector runs as a container or a Python process, next to the tool it reads if needed: it only opens outbound connections, to OpenAEV and to the tool. Deploy it with [XTM Composer](/deploy/integrations/composer/) from :menu[Integrations], from a Docker image and an entry in the catalog, or [by hand](/deploy/integrations/manual/). The Filigran collectors take `OPENAEV_URL`, `OPENAEV_TOKEN`, `OPENAEV_TENANT_ID`, `COLLECTOR_ID`, `COLLECTOR_NAME`, `COLLECTOR_PLATFORM` (the type of security platform), `COLLECTOR_PERIOD` (an ISO 8601 duration such as `PT1M`) and `COLLECTOR_LOG_LEVEL`, plus the settings of the tool.

## What's next

- [Build an OpenAEV collector](/develop/build/collector/), a collector step by step.
- [Injectors](/develop/integration-model/injectors/), what produces the expectations a collector answers.
- [Expectations](/openaev/use/expectations/), how the results show to the validation engineer.
