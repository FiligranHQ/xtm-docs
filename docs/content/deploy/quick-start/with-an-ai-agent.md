---
title: Deploy with an AI agent
description: Let Claude Code or another coding agent deploy the XTM platform on your
  machine, from a prompt and steps it can check.
doc_type: how-to
products:
  - suite
personas: [platform-engineer, evaluator, developer]
owner: docs-platform
last_reviewed: 2026-10-31
---

A coding agent such as Claude Code can deploy the XTM platform on your machine by following [Run the XTM platform locally](/deploy/quick-start/). That page is written for it as much as for you: every step is a command, and every step ends with a check whose expected result is written down.

::diagram{id=deploy-quick-start-with-an-ai-agent}

## Before you start

- Run the agent on a machine you can reset, or a virtual machine: it runs `docker` and `git` with your rights.
- Have Docker Compose 2.20.2 or later and 16 GB of RAM on it.

## Give it this prompt

The deployment starts the whole stack: XTM One, OpenCTI, OpenAEV, XTM Composer and the default integrations of each platform. The prompt has no module to choose.

```text
Deploy the XTM platform on this machine with Docker, following
https://filigran.io/docs/deploy/quick-start/index.md step by step.
It starts the whole stack: do not try to leave a module out.

Before you start, check Docker Compose 2.20.2 or later, the free RAM and the ports the page lists,
and tell me what is missing instead of working around it.
Create .env with the page's command. It prints the admin passwords: never repeat a password, key or token.
After each step, run its check and stop if it does not give the expected result.
At the end, give me the URL of each platform and tell me the credentials are in .env.
```

:::verify[What a good run ends with]
The agent reports each check it ran, OpenCTI, OpenAEV, XTM One and OpenCRQ say `healthy`, and it gives you the addresses, such as `http://localhost:8080` for OpenCTI, without any secret in its answer.
:::

## What the agent works from

| Source | What it gives |
| --- | --- |
| [The quick start in Markdown](/deploy/quick-start/index.md) | Every command, every check, and the table of modules and the services each one starts |
| [stack.json](/deploy/stack.json) | The same stack as data: modules, services, dependencies, ports, health checks |
| [llms.txt](/llms.txt) | The map of the whole documentation |

The module table and `stack.json` are generated from the `docker-compose.yml` of FiligranHQ/xtm-docker, so they match what Docker Compose will start.

## What's next

- [Run the XTM platform locally](/deploy/quick-start/), the steps the agent follows.
