---
title: Executors
description: How an OpenAEV executor starts the implant on each targeted endpoint, the parts an executor has in the platform code, and what a contributor writes to add one, as of 3.261029.0.
doc_type: explanation
products:
  - openaev
personas: [developer, platform-engineer]
owner: docs-develop
last_reviewed: 2026-11-09
migrated_from:
  - openaev:/integrations/executors/executors/
---

An executor is the OpenAEV integration that runs attack actions on endpoints: for each targeted agent, it starts the implant, and the implant runs the payload and reports to the platform. Unlike injectors and collectors, executors are not separate programs: they live in the code of the platform and run inside it, whether you deploy them from the catalog or not. This page explains, as OpenAEV 3.261029.0 implements it, how an inject reaches an executor, what an executor is made of, and what you write to add one to the [OpenAEV repository](https://github.com/OpenAEV-Platform/openaev). To choose and set up an executor, read [Choose between the OpenAEV agent and your EDR](/openaev/use/executors/).

::diagram{id=develop-integration-model-executors}

## The executors of the platform

| Executor | Type | How it starts the implant |
| --- | --- | --- |
| OpenAEV agent | `openaev_agent` | It saves a job; the OpenAEV agent of the endpoint fetches its jobs from the API and runs them |
| Caldera | `openaev_caldera_executor` | Through the Caldera agents of the endpoints |
| CrowdStrike Falcon | `openaev_crowdstrike_executor` | A Real Time Response script, in batches |
| Microsoft Defender for Endpoint | `openaev_mde_executor` | A Live Response script |
| Palo Alto Cortex | `openaev_paloaltocortex_executor` | A script of the agent script library, in batches |
| SentinelOne | `openaev_sentinelone_executor` | A Remote Ops script, in batches |
| Tanium | `openaev_tanium` | A Tanium package, in batches |

The code of each one is in `openaev-api/src/main/java/io/openaev/executors/<name>/`, and its integration in `openaev-api/src/main/java/io/openaev/integration/impl/executors/<name>/`.

## How an inject reaches an executor

A contract that acts on endpoints needs an executor: the payloads of the Threat Arsenal, and the contracts of injectors registered with `injector_payloads`. When such an inject runs, the platform:

1. Resolves the agents of the targeted endpoints, and leaves out, each with a trace on the inject, the agents that are inactive (unseen for one hour), that have no executor, that are overloaded, or whose endpoint the inject's markings do not reach.
2. Groups the remaining agents by executor.
3. Asks each executor to launch the inject on its agents, first in one batch, then one by one for the agents the batch did not handle.
4. Records an error trace on each agent of an executor that fails, so no agent waits for a result that will not come.

The executor builds, for each agent, the command that downloads and starts the implant: the injector's command for the endpoint's platform and architecture, from its `injector_executor_commands` (`Windows.x86_64`, `Linux.arm64`, `MacOS.arm64`...). Before running it, the executor replaces its placeholders:

| Placeholder | Replaced by |
| --- | --- |
| `#{inject}`, `#{agent}`, `#{tenant}` | The identifiers of the inject, the agent and the tenant |
| `#{token}` | A token the implant uses to call the API |
| `#{baseUrl}` | The address of OpenAEV the endpoint reaches, `openaev.agent-url` or `openaev.base-url` |
| `#{location}`, `#{payload_location}` | The folder of the implant and of the payloads on the endpoint |
| `#{maxSize}`, `#{unsecuredCertificate}`, `#{withProxy}` | Settings of the implant |

The implant reports to `POST /api/tenants/<tenant id>/injects/execution/<agent id>/callback/<inject id>`; the inject is closed when every agent has completed or timed out.

## What an executor is made of

An executor is a set of Spring classes of the platform:

| Part | Base class | What it does |
| --- | --- | --- |
| Integration factory | `IntegrationFactory` | A `@Service`. Creates the executor's entry in the catalog, with its description, logo and form (`insertCatalogEntry`), migrates its old configuration keys (`runMigrations`), and creates an integration for each deployment (`spawn`) |
| Integration | `Integration` | One per deployment. `innerStart` registers the executor with `ExecutorService.register` (identifier, type, name, documentation link, color, icon, banner, platforms) and schedules its tasks; `innerStop` cancels them; `refresh` reads its configuration |
| Configuration | `BaseIntegrationConfiguration` | One field per setting, each with `@IntegrationConfigKey`: its key in the deployment form (`EXECUTOR_ID`, `EXECUTOR_NAME`, `EXECUTOR_<NAME>_API_URL`...), its description, its type, whether it is required. `@ConfigurationProperties(prefix = "executor.<name>")` also reads the keys of the platform configuration, the first time |
| Executor context | `ExecutorContextService` | Launches the implant: `launchBatchExecutorSubprocess` for many agents at once, which returns the agents it handled, and `launchExecutorSubprocess` for one agent |
| Endpoint import | A `Runnable` the integration schedules | Reads the hosts and groups of the vendor and turns them into endpoints, agents of the executor and asset groups (`EndpointService.syncAgentsEndpoints`, `AssetGroupService`) |
| Cleanup | A `Runnable` the integration schedules | Deletes the old implant and payload folders on the endpoints |

The CrowdStrike executor shows every part: `CrowdStrikeExecutorIntegrationFactory`, `CrowdStrikeExecutorIntegration`, `CrowdStrikeExecutorConfig`, `CrowdStrikeExecutorContextService`, `CrowdStrikeExecutorService` and `CrowdStrikeGarbageCollectorService`. Its integration registers the executor for Windows, Linux and macOS, imports the endpoints every `EXECUTOR_CROWDSTRIKE_API_REGISTER_INTERVAL` seconds (1,200 by default) and cleans them every `EXECUTOR_CROWDSTRIKE_CLEAN_IMPLANT_INTERVAL` hours (8 by default).

The EDR executors check the Enterprise Edition license before they launch anything, and record a `LICENSE RESTRICTION` trace without it. Deploying one requires the **Manage tenant settings** capability.

## The executor registration route

`POST /api/tenants/<tenant id>/executors` registers an executor from outside, with `executor_id`, `executor_name`, `executor_type` and `executor_platforms`, and the platform refuses a second executor of the same type with another identifier. But the platform only dispatches injects to executors it has code for: it looks up the executor context of an executor by its name, or by its deployment, inside its own process, and records an error trace on every agent when it finds none. So a new executor is a contribution to the platform, built and tested like the rest of it: see [Build and run from source](/develop/contribute/build-from-source/).

## What's next

- [Injectors](/develop/integration-model/injectors/), whose commands the executors run.
- [Collectors](/develop/integration-model/collectors/), to read what the endpoint's security tools saw.
- [Write an OpenAEV database migration](/develop/contribute/database-migrations/), when your executor stores something new.
