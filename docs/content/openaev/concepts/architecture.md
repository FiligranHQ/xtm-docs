---
title: "How OpenAEV works: platform, agents, collectors and injectors"
description: What OpenAEV is made of, what its platform, agents, executors, injectors and collectors each do, and the path an inject takes from a simulation to a score.
doc_type: explanation
products:
  - openaev
personas: [validation-engineer, platform-engineer, admin]
owner: docs-platform
last_reviewed: 2026-10-31
migrated_from:
  - openaev:/deployment/platform/overview/
  - openaev:/deployment/managers/
---

OpenAEV is made of a platform, where you build scenarios and run simulations, and of integrations that act on your environment and measure its response. Agents and executors run actions on endpoints, injectors deliver injects to people and systems, and collectors read what your security tools saw. This page explains what each part does and the path an inject takes. With it, you know what to deploy and where to look when a result is missing. How OpenAEV fits in the XTM stack, next to XTM One and OpenCTI, is in [The XTM stack](/deploy/plan/architecture/).

## What OpenAEV is made of

::diagram{id=openaev-architecture-components}

| Part | What it is | Keeps data |
| --- | --- | --- |
| Platform | A Java application: the web interface, the API under `/api`, the scheduler and the built-in integrations | No |
| OpenAEV agents | Programs installed on your endpoints that run the actions of injects | No |
| Executors | The integrations that run work on endpoints: the OpenAEV agent, or a third-party agent driven through its vendor's API | No |
| Injectors | The integrations that deliver injects: built into the platform, or standalone Python processes | No |
| Collectors | Standalone Python processes that read data from your security tools and other sources | No |
| PostgreSQL | The main database, including user sessions, in the database `openaev` by default | Yes |
| Elasticsearch or OpenSearch | Search indices that the platform builds from PostgreSQL, under the prefix `openaev` by default | Yes, rebuilt from PostgreSQL |
| RabbitMQ | The queues between the platform and its injectors | Messages in transit |
| S3 storage (MinIO, or RustFS in xtm-docker) | Files uploaded to the platform, in the bucket `openaev` by default | Yes |

OpenAEV does not use Redis. Sessions are stored in PostgreSQL, so they survive a restart of the platform.

## The platform

The platform is the center of OpenAEV. You configure scenarios, simulations, atomic tests and integrations there, and it schedules each inject at its time. Agents, injectors and collectors connect to its API with a URL and a token, `OPENAEV_URL` and `OPENAEV_TOKEN` for the Python integrations.

The platform writes to PostgreSQL first. It then copies the data to Elasticsearch in the background, for search. A change can take a short while to appear in search results, and the indices can be rebuilt from PostgreSQL.

## Agents and executors

To act on an endpoint, OpenAEV needs an executor on it: a program that starts the actions of an inject as separate processes.

- **The OpenAEV agent** is the default executor, for Windows, Linux and macOS, on x64 and ARM64 processors. It enrolls the endpoint as an asset in the platform, fetches the jobs to run and starts an implant for each one. The implant runs the actions; the agent itself does nothing on the endpoint, so it stays neutral toward antivirus software. See [OpenAEV agents](/openaev/use/agents/).
- **A third-party agent** already on your endpoints can play the same role: CrowdStrike Falcon, SentinelOne, Tanium or Caldera. The platform asks the vendor's API to start the implant. Each is turned on in the platform configuration with its own key, such as `executor.crowdstrike.enable`.

Each implant works in its own folder, under `runtimes/` in the agent's folder, one per inject execution. Antivirus exclusions apply to `runtimes/` only, so the actions themselves can still be detected and blocked: that is what the simulation measures. An OpenAEV agent runs at most 5 jobs at a time by default (`executor.openaev.agent.max-simultaneous-jobs`).

Installing executors on endpoints is in [How integrations run](/deploy/integrations/how-they-run/).

## Injectors

Injectors deliver injects: they push the actions of a simulation to people and systems. Two kinds exist:

- **Built-in injectors** run inside the platform and are configured in its configuration: email, SMS, media pressure, challenges, manual action reminders and Caldera, among others.
- **External injectors** are standalone Python processes, such as the HTTP query injector. Each one registers with the platform, which gives it the connection details of RabbitMQ. The injector then takes its injects from its own queue. It must be able to reach RabbitMQ under the host name the platform knows it by.

Some injectors run actions on endpoints. When they register, they tell the executors how to start them on Windows, Linux and macOS, and the executors do the rest. See [Injectors](/openaev/use/injectors/).

## Collectors

Collectors are standalone Python processes that pull data from other systems, for two purposes:

- **Measure the response to injects.** Collectors for security tools look for the alerts and logs an inject caused. They fill the detection and prevention expectations. An endpoint detection and response (EDR) collector matches on the host name and on the name of the parent process, the implant, which contains the inject's ID. A security information and event management (SIEM) collector relies on the EDR: it confirms what the EDR collector has already matched.
- **Feed the platform.** Other collectors import assets, identities, and threat intelligence such as attack patterns and kill chains, used to build scenarios.

See [Collectors](/openaev/use/collectors/).

## The path of an inject

::diagram{id=openaev-architecture-inject-path}

Take a simulation that runs an attack technique on a group of endpoints:

1. At the time set in the simulation, the platform starts the inject and creates its expectations: for example, that the attack is detected and prevented.
2. For an action on endpoints, the platform hands a job to the executor. An OpenAEV agent fetches the job from the platform. For a third-party agent, the platform calls the vendor's API.
3. On each endpoint, the agent starts an implant, which runs the actions. For an inject to people or systems, an injector delivers it instead: an email, an HTTP request, a message.
4. Your security tools see the actions, or not, and raise alerts.
5. Collectors read the alerts and logs of those tools and match them to the inject.
6. Each match fills an expectation. An expectation that nothing has filled when it expires stays unmet: the attack counts as not detected or not prevented.

Expectations expire after a delay set in the platform configuration, in seconds. By default, detection and prevention expectations expire after 6 hours (`openaev.expectation.technical.expiration-time`), and expectations of people after 24 hours (`openaev.expectation.human.expiration-time`).

## Background work

Background components of the platform, its managers, keep some features working without anyone asking. The telemetry manager counts how the platform is used every hour and sends that [usage telemetry](/openaev/administer/telemetry/) to Filigran every 6 hours. The platform runs other background work, set by configuration keys:

| Work | What it does | When it runs |
| --- | --- | --- |
| Indexing | Copies the data of PostgreSQL to the Elasticsearch indices | Continuously |
| Agent inactivity monitor | Checks which agents have stopped reporting | Every 5 minutes (`openaev.cron.config.agent.inactivity.monitor.interval`, in minutes) |
| Implant cleanup | Asks each agent to remove the folders of implants and payloads older than 24 hours | Every 8 hours (`clean-implant-interval` of each executor, in hours) |
| Health probes | Checks the connection to PostgreSQL, RabbitMQ and S3 storage, for the health endpoint | Every 10 seconds (`openaev.healthcheck.connectivity-probe-interval`) |

Every key is in the [OpenAEV configuration reference](/reference/openaev/configuration/).

:::xtm-one[Registration with XTM One]
OpenAEV registers with XTM One when both `openaev.xtm.one.url` and `openaev.xtm.one.token` are set. How to connect OpenAEV to XTM One and to OpenCTI is in [Connect the modules](/deploy/configure/connect-modules/).
:::

## What's next

- [The XTM stack](/deploy/plan/architecture/): OpenAEV next to XTM One, OpenCTI and XTM Composer, and the services they share.
- [Sizing](/deploy/plan/sizing/): the machines the platform and its services need.
- [How integrations run](/deploy/integrations/how-they-run/): deploy executors, injectors and collectors.
- [Scenarios and simulations](/openaev/concepts/scenarios-simulations/) and [Injects and expectations](/openaev/concepts/injects-expectations/): what you build on this architecture.
