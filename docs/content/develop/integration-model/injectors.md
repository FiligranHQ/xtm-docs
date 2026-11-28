---
title: Injectors
description: How an external OpenAEV injector registers its contracts, receives each inject from its RabbitMQ queue and reports the result, as the platform implements it in 3.261029.0.
doc_type: explanation
products:
  - openaev
personas: [developer, platform-engineer]
owner: docs-develop
last_reviewed: 2026-11-09
migrated_from:
  - openaev:/integrations/injectors/injectors/
---

An injector is the OpenAEV integration that carries out an inject: it sends the message, calls the API, runs the scan. An external injector is a program of its own, in any language, that talks to the platform through its REST API and a RabbitMQ queue. This page explains that protocol as OpenAEV 3.261029.0 implements it: what the injector declares when it registers, what each inject message holds, how the injector reports a result, and what keeps it shown as running. To write one step by step, follow [Build an OpenAEV injector](/develop/build/injector/); to choose between the injectors that exist, read [How injectors deliver your injects](/openaev/use/injectors/).

::diagram{id=develop-integration-model-injectors}

## The life of an external injector

1. **It registers.** At start, it posts its identity and its contracts to the platform. The platform stores the injector and its contracts, declares the injector's RabbitMQ queue, and answers with the connection details of RabbitMQ and the name of that queue.
2. **It waits for injects.** When an inject of one of its contracts is due, in a simulation or an atomic test, the platform computes the inject's expectations, then publishes the inject to the injector's queue once the transaction commits.
3. **It acts and reports.** The injector consumes the message, tells the platform it received the inject, does the work, and posts the result. The last result closes the inject.
4. **It registers again, and again.** Each registration is an upsert and a heartbeat. An injector that stops registering shows as stopped after two minutes.

Filigran's `pyoaev` Python library implements this protocol for the injectors of the [OpenAEV-Platform/injectors](https://github.com/OpenAEV-Platform/injectors) repository. The sections below describe it from the platform side, so you can write an injector in another language or check what a library does.

## Registration

The injector posts a multipart request to `POST /api/tenants/<tenant id>/injectors`, with the token of its user in `Authorization: Bearer <token>`. The `input` part is a JSON document with the content type `application/json`; an optional `icon` part, a PNG image, becomes the injector's icon. The `/api/injectors` route without a tenant works only when the user reaches a single tenant: with several, the platform refuses the registration (400) because it cannot tell which tenant the injector belongs to.

| Field of `input` | Required | What it does |
| --- | --- | --- |
| `injector_id` | Yes | The identifier of this injector, a UUIDv4 you choose once and keep across restarts. It names the queue and is the key of the upsert |
| `injector_name` | Yes | The name the interface shows |
| `injector_type` | Yes | A stable technical name, such as `openaev_http_query`. It names the icon file and the prefix of the process-name signature of agent expectations (see [Collectors](/develop/integration-model/collectors/#the-signatures-of-an-expectation)) |
| `injector_contracts` | Yes, may be empty | The contracts, below |
| `injector_custom_contracts` | No, `false` | Whether users may create custom contracts on this injector |
| `injector_category` | No | Free text the platform stores |
| `injector_payloads` | No, `false` | Whether the payloads of the Threat Arsenal run through this injector, on endpoints (see [Injectors that run on endpoints](#injectors-that-run-on-endpoints)) |
| `injector_executor_commands` | No | The command that starts the injector on an endpoint, per platform and architecture |
| `injector_executor_clear_commands` | No | The command that cleans up after it, per platform and architecture |
| `injector_author` | No | The organization the contracts are attributed to. Without it, the platform attributes them to an organization named after the injector |

The answer is the injector's connection to RabbitMQ:

```json
{
  "connection": {"host": "rabbitmq", "vhost": "/", "use_ssl": false, "port": 5672, "user": "openaev", "pass": "..."},
  "listen": "openaev_injector_9f2d6c1e-4b7a-4e0c-8d5f-3a1b2c4d5e6f"
}
```

`connection` holds the RabbitMQ settings of the platform itself, the `openaev.rabbitmq.*` keys of its [configuration](/reference/openaev/configuration/#openaev-rabbitmq), credentials included: treat the answer as a secret and never log it. The injector must reach RabbitMQ under that host name and port, so set `openaev.rabbitmq.hostname` to a name every injector resolves, not `localhost`.

### What a new registration changes

A registration with a known `injector_id` updates the injector: its name, its type, its flags, its commands, and its contracts. The contracts follow the list you send:

- A contract of the list that exists is updated: labels, content, platforms, attack patterns, atomic testing.
- A contract of the list that does not exist is created.
- A contract that the injector had and that the list no longer holds is deleted, unless a user created it as a custom contract. **The injects built on it are deleted with it**, and so are the attack chaining step templates that use it.

So never drop a contract from the list to hide it: keep its `contract_id` stable for as long as scenarios may use it.

## Contracts

A contract is one kind of inject the injector offers: its label in the inject type list, the form the user fills in, and what the injector receives. Each element of `injector_contracts` is:

| Field | Required | What it does |
| --- | --- | --- |
| `contract_id` | Yes | A stable UUID. An inject refers to its contract by this identifier |
| `contract_labels` | No | The label, per language: `{"en": "...", "fr": "..."}` |
| `contract_content` | Yes | The contract itself, as a JSON document serialized in a string |
| `contract_manual` | No, `false` | A manual contract: a person carries it out, not the injector |
| `contract_attack_patterns_external_ids` | No | MITRE ATT&CK identifiers, such as `T1566.002`. The platform links the attack patterns of the tenant with these identifiers and ignores the ones it does not have |
| `contract_platforms` | No | The platforms the contract applies to: `Windows`, `Linux`, `MacOS`, `Service`, `Generic`, `Internal`... A contract that acts on a remote service, not on an endpoint, takes `Service` |
| `is_atomic_testing` | No, `true` | Whether the contract is offered in atomic tests |
| `contract_domains` | No | Security domains, each with `domain_id`, `domain_name` and `domain_color` |

`contract_content` is what the interface reads to draw the form. Its keys:

```json
{
  "contract_id": "5d0c3a8e-2f4b-4c39-9a51-7f2e8c1b6d40",
  "config": {"type": "openaev_http_query", "label": {"en": "HTTP query"}, "color_dark": "#7e57c2", "color_light": "#5e35b1"},
  "label": {"en": "Send a GET request"},
  "manual": false,
  "fields": [
    {"key": "uri", "label": "URL", "type": "text", "mandatory": true, "readOnly": false, "defaultValue": ""}
  ],
  "variables": [],
  "context": {}
}
```

Each field has a `key`, which becomes the key of its value in the inject, a `label`, a `type`, and `mandatory` and `readOnly`. The types are `text`, `number`, `checkbox`, `textarea`, `select`, `choice`, `dependency-select`, `attachment`, `team`, `expectation`, `asset`, `asset-group`, `ai-target`, `payload`, `targeted-asset`, `credential-reference`, `article` and `challenge`. Conditions between fields use `mandatoryGroups`, `mandatoryConditionFields` and `mandatoryConditionValues`, `visibleConditionFields` and `visibleConditionValues`. An `expectation` field lets the user choose the expectations of the inject, such as **Detection** and **Prevention**, which collectors then answer.

## The queue

The platform declares one durable queue per injector, at each registration:

| Object | Name, with the default prefix `openaev` |
| --- | --- |
| Exchange | `openaev_amqp.connector.exchange`, of type direct, shared by every injector |
| Queue | `openaev_injector_<injector_id>` |
| Routing key | `openaev_push_routing_<injector_id>` |

The prefix is `openaev.rabbitmq.prefix`, and the queue type is `openaev.rabbitmq.queue-type` (`classic` by default, or `quorum`). The injector only consumes: use the queue name the registration returns, never one you build.

Each message is a JSON document with four keys:

| Key | What it holds |
| --- | --- |
| `injection` | The inject: `inject_id`, `inject_title`, `inject_content` (the values of the contract's fields, by their `key`), `inject_injector_contract` (the contract, with its `injector_contract_id`), `inject_expectations`, `inject_teams`, `inject_assets`, `inject_asset_groups`... |
| `assets` | The endpoints the inject targets, among those the inject's markings let it reach |
| `assetGroups` | The asset groups the inject targets |
| `attachments` | When the inject uses credentials: their references and an authorization code to resolve them. Absent otherwise |

The platform publishes an inject only once. Acknowledge the message after you report the result: an injector that dies before acknowledging gets the message again from RabbitMQ when it reconnects.

## Reporting the result

The injector reports with two routes, both under `/api/tenants/<tenant id>/injects/execution/`:

1. `POST .../reception/<inject id>`, with `{"tracking_total_count": 1}`: the inject moves to **Pending**.
2. `POST .../callback/<inject id>`, once or more, with the result:

| Field | Required | What it does |
| --- | --- | --- |
| `execution_message` | Yes | The message of the trace, shown in the inject's execution details |
| `execution_status` | Yes | `SUCCESS` (stored as `EXECUTED`), `WARNING`, `ERROR`, `TIMEOUT`, `INFO`... A success or an error counts in the inject's status; `INFO` does not |
| `execution_action` | No | `complete` closes the inject. `data` adds a trace and leaves it open, for an injector that reports in several steps |
| `execution_duration` | No | Duration of the execution, in milliseconds |
| `execution_output_structured` | No | A JSON document the platform parses into findings, for the contract's outputs |
| `execution_output_raw` | No | The raw output |
| `execution_context_identifiers` | No | The identifiers of the assets or AI targets the trace is about, up to 1,000: the trace then shows in the per-target view instead of the inject's timeline |

The platform accepts `complete` only on an inject in **Pending**, so post the reception first. An inject still in **Pending** 10 minutes after it was sent, the default of `inject.execution.threshold.minutes` in the [`inject` block](/reference/openaev/configuration/#inject), is closed as a timeout, except in a running attack chaining workflow. A callback that arrives after the inject is closed is kept as information and changes nothing. The callbacks go through a queue of the platform and are written in batches, so a result can take a few seconds to show.

## Staying shown as running

The interface shows an external injector as running while its last registration is less than two minutes old, and as stopped after that. Register again every 40 seconds or so: the platform recognizes a registration that changes nothing and does not write it to the audit log. A stopped injector keeps its contracts and its injects; the injects published meanwhile wait in its queue until it comes back. To remove an injector for good, stop it, then delete it from :menu[Integrations].

The platform also refuses to run an inject when the injector was deployed by XTM Composer and its deployment is not started, with the trace `No started connector instance found for injector type`. An injector you run yourself has no deployment, and the platform runs its injects.

## Injectors that run on endpoints

Some injectors do their work on the targeted endpoints rather than from where they run. They register with `injector_payloads: true`: the platform then attaches the payloads of the Threat Arsenal to them, as contracts that need an executor. When such an inject runs, the platform does not use the queue alone: it asks the executor of each targeted agent, the OpenAEV agent or an endpoint detection and response (EDR) tool, to run the injector's command on the endpoint.

`injector_executor_commands` gives that command, keyed by platform and architecture: `Windows.x86_64`, `Windows.arm64`, `Linux.x86_64`, `Linux.arm64`, `MacOS.x86_64`, `MacOS.arm64`. The executor replaces placeholders in it before running it, among them `#{inject}`, `#{agent}`, `#{tenant}`, `#{token}` and `#{location}`. `injector_executor_clear_commands` has the same keys and cleans up. [Executors](/develop/integration-model/executors/) explains how an executor runs these commands.

## The user of an injector

An injector acts with the rights of the user whose token it holds. Give it a user of its own, with a role that grants:

| Call | Capability |
| --- | --- |
| Register | **Manage tenant settings**, which allows creating and writing injectors |
| Post the reception and the results | **Manage assessment**, or a grant on each scenario, simulation or atomic test the injector serves: the platform checks the right on the inject's parent |

The capabilities are in the [OpenAEV capabilities reference](/reference/openaev/capabilities/). A call the user has no right to make answers 404, not 403, so as not to reveal what exists.

## How injectors are deployed

An external injector runs as a container or a Python process that only opens outbound connections: to the OpenAEV API and to RabbitMQ.

| Way | What the injector must provide |
| --- | --- |
| [XTM Composer](/deploy/integrations/composer/), from :menu[Integrations] | A Docker image and an entry in the catalog of the platform, with the settings the form asks for. The settings of the Filigran injectors are `OPENAEV_URL`, `OPENAEV_TOKEN`, `OPENAEV_TENANT_ID`, `INJECTOR_ID`, `INJECTOR_NAME` and `INJECTOR_LOG_LEVEL`, plus their own |
| [By hand](/deploy/integrations/manual/) | A Docker image or the sources, and a `README.md` listing its settings |

## What's next

- [Build an OpenAEV injector](/develop/build/injector/), an injector step by step.
- [Collectors](/develop/integration-model/collectors/), to read what the security tools saw of the injects.
- [Executors](/develop/integration-model/executors/), to understand how injects reach endpoints.
- [Integration model](/develop/integration-model/), what every integration shares.
