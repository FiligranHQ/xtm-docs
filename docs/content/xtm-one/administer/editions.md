---
title: What each edition of XTM One brings
description: What the Community Edition and the Enterprise Edition of XTM One give, the two ways to activate the Enterprise Edition, what a license carries, and what happens when the Enterprise Edition ends.
doc_type: explanation
products: [xtm-one]
personas: [admin, platform-engineer]
owner: docs-xtm-one
last_reviewed: 2026-11-08
migrated_from:
  - xtm-one:/administration/editions-and-licensing/
---

XTM One is one platform with two editions, and every deployment contains all of it. The edition decides one thing: whether AI is licensed. In the **Community Edition**, the platform runs without AI: sign-in, users, settings, the connections to OpenCTI, OpenAEV and OpenCRQ and their MCP servers work, while agents, chat and automations are listed but locked. In the **Enterprise Edition**, every capability is available. A deployment is in Enterprise Edition as soon as an OpenCTI or OpenAEV platform in Enterprise Edition is connected to it, or an XTM One license is installed; no redeployment is needed. OpenCRQ is never a source: it takes its own edition from the XTM One license. Whether AI actually runs is a separate switch an administrator holds: see [Run XTM One with AI switched off](/xtm-one/administer/running-without-ai/).

::diagram{id=xtm-one-editions-activation}

## What each edition gives

| Area | Community Edition | Enterprise Edition |
| --- | --- | --- |
| Sign-in, single sign-on, users, groups, invitations, API keys | Yes | Yes |
| :menu[Administration > Settings], branding, :menu[Administration > Audit logs], the security audit, secret vaults | Yes | Yes |
| Registering OpenCTI, OpenAEV and OpenCRQ, and the exposure figures of the CTEM dashboards | Yes | Yes |
| The MCP servers of the connected platforms, such as `/mcp/opencti`, for AI clients | Yes | Yes |
| Integrations already connected, relays, browser profiles: kept and editable | Yes | Yes |
| Preparing AI providers in :menu[Administration > Settings > AI and models > Models], for use once the Enterprise Edition is active | Yes | Yes |
| :menu[Administration > Observability]: what AI activity already happened | Yes, read only | Yes |
| Chat, in XTM One, in messaging apps and in the assistants of OpenCTI, OpenAEV and OpenCRQ (in OpenCRQ, only when the license names it as a Sub-License) | No | Yes |
| Agents and their assignments, background tasks, workspaces | No | Yes |
| :menu[Orchestration > Fleets]: [teams of agents](/xtm-one/build/fleets/) that work in one workspace | No | Yes |
| :menu[Work > Value]: [what the agents' work is worth](/xtm-one/use/value/), in hours saved and net value | No | Yes |
| :menu[Agent Studio]: knowledge bases, prompts, templates, skills | No | Yes |
| Connecting a new integration, MCP server or custom tool; creating variables | No | Yes |
| The agent MCP endpoints and Agent-to-Agent interoperability | No | Yes |
| The exposure briefing and the AI assessments of each CTEM stage | No | Yes |
| Agentic capacity | None | The included allowance, or the license's capacity |

There is nothing in between: the Enterprise Edition from a connected platform and the Enterprise Edition from a license unlock the same capabilities. They differ only in their capacity, described below.

In the Community Edition, the locked areas stay in the menu with a lock, and hovering one says why. A creation button opens **Activate Enterprise Edition to unlock AI**, which lists the two ways to activate it with **Start free trial** and **Contact sales**. **Fleets** opens on a notice that fleets need the agentic AI license: the fleets are kept, and listed again once the Enterprise Edition is back. **Value** opens on "Value is part of Enterprise Edition." with the same two buttons. Both stay open when the Enterprise Edition is active and an administrator switches AI off. Everything else that exists stays readable, and nothing you prepare, such as users, groups, AI providers or platform connections, is lost when the edition changes.

## Activate the Enterprise Edition

Two ways, neither needs a redeployment:

- **Connect a platform in Enterprise Edition.** Register an OpenCTI or OpenAEV platform that holds its own Enterprise Edition license: XTM One is in Enterprise Edition as soon as the registration is accepted. OpenCRQ sends no license when it registers, so registering it does not change the edition of XTM One. See [Connect the modules to each other](/deploy/configure/connect-modules/).
- **Install an XTM One license.** In :menu[Administration > Settings > Platform], the card **Enterprise License** shows the **Instance ID** that Filigran needs to issue the license. Click **Install License**, paste the license's PEM certificate and confirm. You can set it in `ENTERPRISE_LICENSE` instead, on the platform and on the worker, then restart: the variable wins over a license pasted in the interface. See the [XTM One configuration](/reference/xtm-one/configuration/#platform-mode).

A license is refused when it was not issued by Filigran for XTM One, for another instance, or has ended. After its end date, a standard license keeps working for a 90-day grace period, and the card warns **License expires within 3 months** beforehand; a trial license stops at its end date.

## What a license carries

A license carries two things, and never switches a capability on or off:

- **Agentic capacity**: how many agentic executions the deployment may run per day, per month and per year. One agentic execution is one complete request: a chat turn, an assignment run, a background task, however many agents and tools it involves. The card shows it under **Agentic capacity**; a period the license sets no ceiling for reads **Unlimited**.
- **Sub-Licenses**: the OpenCTI, OpenAEV and OpenCRQ platforms the license also puts in Enterprise Edition, by instance or **Global** for every platform of that product. Each gets its Enterprise Edition from XTM One when it registers.

::screenshot{id=xtm-one-editions-license-card alt="The Enterprise License card with its Enterprise Edition badge: the organization Filigran, a Standard license that expires on Dec 1, 2026, read from an environment variable, an agentic capacity of 50k a year, and OpenCTI, OpenAEV and OpenCRQ as Global sub-licenses."}

### The included allowance

When the Enterprise Edition comes from a connected platform and no XTM One license is installed, the deployment runs under an included allowance of 30 agentic executions per day, 100 per month and 300 per year, shared by all its users. It is sized to discover and try the agents, not for production. Installing a license replaces it with the license's capacity.

:menu[Administration > Settings > Usage and quotas > Quotas] shows which ceilings apply and where they come from, and lets you give users a default share and set overrides per group or per user within those ceilings.

## When the Enterprise Edition ends

A deployment returns to the Community Edition when its last source goes: the connected platform reports, when it registers again, that it lost its Enterprise Edition, or it is unregistered; or the license is removed or ends. Within a few seconds, AI stops starting, in the interface and in the automations:

- Schedules, integration events, webhooks and messaging bots start no AI work. A person writing to a bot is told that the Enterprise Edition is required; a webhook sender receives a `403` with the same explanation.
- A run that comes due is **Held** and starts by itself once the Enterprise Edition is back, or fails with the cause **AI not licensed** after `AI_UNAVAILABLE_HOLD_MAX_SECONDS` (one week by default).
- Nothing is deleted, paused or charged. When the Enterprise Edition is back, everything resumes on its own; a schedule that missed several occurrences runs once, not once per occurrence.

Work already running finishes under its own time limit.

:::example[From a connected OpenCTI to a license of its own]
Grand Meridian Hotels runs OpenCTI in Enterprise Edition. Marc Dupuis installs XTM One at https://xtm-one.grand-meridian.example, and Hélène Marchetti registers OpenCTI with it.

- As soon as OpenCTI registers, the **Enterprise License** card reads that the Enterprise Edition is active through the connected platform, under the included allowance of 30 executions per day. Sophie Laurent builds **Velvet Harbor watcher** and the SOC tries **Night shift triage**.
- After two weeks, the SOC uses the allowance by mid-morning. Hélène sends the **Instance ID** to Filigran and receives a license with a daily capacity sized for the three teams, and OpenAEV and OpenCRQ as **Sub-Licenses**.
- Marc keeps the license in the deployment's configuration rather than in the database, and gives it to both services:

  ```yaml
  # docker-compose.override.yml
  services:
    platform:
      environment:
        ENTERPRISE_LICENSE: ${ENTERPRISE_LICENSE}
    worker:
      environment:
        ENTERPRISE_LICENSE: ${ENTERPRISE_LICENSE}
  ```

  ```bash
  export ENTERPRISE_LICENSE="$(cat /etc/xtm-one/grand-meridian-xtm-one.pem)"
  docker compose up -d
  ```

  After the restart, **Agentic capacity** shows the license's figures, and the card's **Source** reads **Environment variable**.
:::

## What's next

- [Run XTM One with AI switched off](/xtm-one/administer/running-without-ai/)
- [Set up the platform in Settings](/xtm-one/administer/settings/)
- [Govern how AI is used](/xtm-one/administer/ai-governance/)
