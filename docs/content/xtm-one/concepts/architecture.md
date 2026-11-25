---
title: "How XTM One works: the platform, the worker and the modules"
description: The two services of XTM One, the stores they keep their data in, what happens when an agent answers, and how OpenCTI, OpenAEV and OpenCRQ connect to it.
doc_type: explanation
products: [xtm-one]
personas: [admin, platform-engineer, ai-builder]
owner: docs-xtm-one
last_reviewed: 2026-11-08
migrated_from:
  - xtm-one:/deployment/overview/
  - xtm-one:/deployment/data-flow/
---

XTM One is two services you run, the **platform** and the **worker**, on three stores: PostgreSQL, Redis and an S3-compatible object storage. People, schedules, webhooks and the other products of the XTM platform send it work. To do that work, it calls the AI providers you configure and the tools its AI agents act through. This page explains what each part does, where your data lives, what leaves the deployment when an agent answers, and how OpenCTI, OpenAEV and OpenCRQ connect. It is the model to keep in mind before you [install XTM One](/deploy/install/xtm-one/) or decide what an agent may reach.

::diagram{id=xtm-one-architecture-components}

## The two services

| Service | Image | What it does | How it scales |
| --- | --- | --- | --- |
| Platform | `xtmone/platform` | Serves the interface and the API on port 4000, the chat, the MCP endpoints and the webhooks. Runs the scheduler that starts assignments, polls integrations for new events, and applies database migrations at startup. Runs the agent's loop for chat answers. | Add CPU and memory and raise `WEB_WORKERS` first, then add replicas for availability |
| Worker | `xtmone/worker` | Runs the background work: assignment runs, background tasks delegated to other agents, the indexing of knowledge-base documents. Picks up a run where it stopped after a restart. | Add replicas when work waits in the queue; each runs up to `WORKER_CONCURRENCY` jobs at once (20 by default) |

Both read the same configuration. Set the same environment variables on both, such as `DATABASE_URL`, `REDIS_URL` and `ENTERPRISE_LICENSE`: see the [XTM One configuration](/reference/xtm-one/configuration/).

## Where data lives

| Data | Store |
| --- | --- |
| Agents, assignments, runs, conversations, users, groups, settings | PostgreSQL |
| Knowledge-base documents and their embeddings, agent memory | PostgreSQL, with the pgvector extension |
| Uploaded files, attachments, files agents generate | Object storage (S3-compatible) |
| Job queue, cache, short-term memory of a conversation | Redis |
| Traces of each AI interaction | PostgreSQL, or ClickHouse when configured |

PostgreSQL is the record: back it up. Redis holds nothing that cannot be rebuilt; a run in progress keeps its state in PostgreSQL, so a lost Redis does not lose a run. Integration and channel credentials are encrypted before they are written to the database.

## What happens when an agent answers

A chat message and an assignment run follow the same path:

1. The request reaches the platform, which authenticates the person or the trigger and loads the agent: its instructions, its model, the tools and knowledge it may use.
2. The platform builds a prompt: the instructions, the context of the request, and the list of tools the agent is allowed to call.
3. It sends the prompt to the agent's AI provider.
4. The model answers, or asks for a tool to be run. It never runs anything itself: it returns the name of a tool and its arguments.
5. The platform or the worker runs the tool: it searches a knowledge base, calls an integration or OpenCTI, hands work to another agent. It returns the result to the model, which may ask for another tool. This repeats until the model writes its answer.
6. The answer goes back to the person, or to the run's record for an assignment, and the interaction is recorded as a trace.

So the model sees the prompt, the tool descriptions and the tool results, never your credentials nor a whole data source. What is sent to a provider, and how to restrict it, is in [Govern how AI is used](/xtm-one/administer/ai-governance/). Before a tool runs, an agent can be set to ask a person for approval: see [How XTM One marks what to trust and what to review](/xtm-one/concepts/trust/).

## How the XTM platform connects

OpenCTI, OpenAEV and OpenCRQ **register** with XTM One. Each presents the registration token set in `PLATFORM_REGISTRATION_TOKEN` on XTM One, which each holds under its own variable: `XTM__XTM_ONE_TOKEN` in OpenCTI, `OPENAEV_XTM_ONE_TOKEN` in OpenAEV, `XTM_ONE__TOKEN` in OpenCRQ. Each registers again regularly: that registration is also its heartbeat. Once a platform is registered:

- Its assistant and its AI features send their requests to XTM One agents, which work with that platform as the person who asked. See [XTM One in OpenCTI](/xtm-one/use/in-opencti/), [XTM One in OpenAEV](/xtm-one/use/in-openaev/) and [XTM One in OpenCRQ](/xtm-one/use/in-opencrq/).
- Agents reach it through its integration tools, and AI clients through its MCP server, such as `/mcp/opencti`: see [Use XTM One's own MCP servers from other clients](/xtm-one/build/platform-mcp-servers/).
- A platform in Enterprise Edition gives XTM One its Enterprise Edition too: see [What each edition of XTM One brings](/xtm-one/administer/editions/).

How to register the platforms is in [Connect the modules to each other](/deploy/configure/connect-modules/).

## Who runs it

Filigran can host XTM One for you, or you run it on your own infrastructure, connected or air-gapped. Hosted by Filigran, you reach it by a URL, configure models, sign-in and agents from the screens, and Filigran handles the infrastructure, upgrades and backups. Self-hosted, you run the two services and the three stores, behind a reverse proxy that terminates HTTPS: see [Install XTM One](/deploy/install/xtm-one/).

:::example[The weekly Velvet Harbor brief, from trigger to Slack]
At Grand Meridian Hotels, XTM One runs at https://xtm-one.grand-meridian.example, with OpenCTI registered. Every Monday at 08:00, Europe/Paris, the assignment **Weekly Velvet Harbor brief** of the agent **Velvet Harbor watcher** comes due:

1. The platform's scheduler sees it is due and creates a run, which the worker picks up.
2. The worker loads the agent **Velvet Harbor watcher** and sends its prompt to the model Sophie Laurent chose for it.
3. The model asks for the new OpenCTI reports on Velvet Harbor. The worker calls OpenCTI with the agent's rights and returns the reports.
4. The model asks for the knowledge base **Grand Meridian security policies**, to word the brief as the crisis communication playbook requires. The worker searches it in PostgreSQL.
5. The model writes the brief. The worker records the run and its trace, and posts the brief to the CTI team.

Camille Roux reads it on Monday morning. If she wants to know which reports it used, the run's record lists each tool call, as [Review what agents did](/xtm-one/use/work-history/) shows.
:::

## What's next

- [How XTM One keeps what agents know and do](/xtm-one/concepts/data-model/): the objects behind the screens.
- [Install XTM One](/deploy/install/xtm-one/) and [Configure XTM One](/deploy/configure/xtm-one/).
- [XTM One configuration](/reference/xtm-one/configuration/): every environment variable.
