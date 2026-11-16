---
title: How injectors deliver your injects
description: What an OpenAEV injector does, the built-in, catalog and external injectors, how they get their injects, and where to see each injector and its inject types.
doc_type: explanation
products:
  - openaev
personas: [validation-engineer, platform-engineer, exercise-planner]
owner: docs-openaev
last_reviewed: 2026-11-07
migrated_from:
  - openaev:/usage/injectors/
---

An injector is the integration that delivers an inject: it sends the email to the players, publishes the article, calls the API, runs the scan, or hands an attack action to the endpoints. Each injector brings its contracts, the inject types you pick when you add an inject to a scenario, a simulation or an atomic test. This page explains the kinds of injectors, how each one gets its injects, and where to see them, so you know which ones your scenarios can use and what to deploy for the others. Where injectors sit among agents, executors and collectors is in [How OpenAEV works](/openaev/concepts/architecture/#injectors).

::diagram{id=openaev-injectors-kinds}

## Three kinds of injectors

| Kind | Where it runs | Examples | What you do |
| --- | --- | --- | --- |
| Built in | Inside the platform, always present | Email, Media pressure, Challenges, Manual, Phishing, OpenAEV Implant | Nothing, besides the mail server the email and phishing injectors use. See [Built-in injectors and their injects](/openaev/use/injectors/built-in/) |
| From the catalog, in the platform | Inside the platform, once deployed | OpenCTI, OVHCloud SMS Platform | Deploy it from :menu[Integrations], with its settings. It requires an Enterprise Edition license |
| External | As its own container or Python program | Nmap, Nuclei, HTTP query, Shodan, NetExec | Run it with XTM Composer from :menu[Integrations], or yourself |

The catalog, in :menu[Integrations], **Available** tab, lists the external injectors the platform knows: reconnaissance (Censys, Nmap, Shodan), vulnerability scanning (Nuclei), attack simulation (HTTP query, Amazon Web Services, NetExec, Stratus Red Team), communication (Microsoft Teams, Slack, email through Microsoft 365, Google Workspace or SMTP) and AI security (AI Red Team).

## How an injector gets its injects

The platform decides when each inject runs. Then:

- A **built-in** injector runs the inject in the platform: it sends the email through the platform's mail server, or publishes the article on the player interface.
- An **external** injector registers with the platform when it starts, with its URL, token and identifier. The platform answers with the connection details of RabbitMQ, and puts each inject in the injector's own queue. So the injector must reach RabbitMQ under the host name and port the platform gives it. Deployed from the catalog with XTM Composer, it registers with the token of the tenant's **Catalog connectors** service account, which every integration of the catalog shares: see [Run a collector](/openaev/use/collectors/#run-a-collector). How an external integration reaches its platform, with XTM Composer or without it, is in [How integrations run](/deploy/integrations/how-they-run/).

## Injectors that act on endpoints

Some contracts run on endpoints: the attack actions of the Threat Arsenal, run by the **OpenAEV Implant** injector, and the contracts of external injectors built for endpoints. An injector of this kind does not reach the endpoint itself. When it registers, it tells the executors how to start it on Windows, Linux and macOS, and an executor starts it on each target: the OpenAEV agent or your EDR. So these injects need an executor on their targets. See [Choose between the OpenAEV agent and your EDR](/openaev/use/executors/).

## See your injectors and their inject types

Open :menu[Integrations], **Deployed** tab. Each injector shows whether it is started and when the platform last heard from it: a built-in injector shows **Runs inside the platform**. Open an injector to see its contracts, with their **Kill chains**, **Payload domains** and **Attack patterns**. An injector deployed by XTM Composer also has a **Logs** tab.

Some injectors accept contracts of your own, built from one of theirs: their contract list then has a button to **Create a new injector contract**, in two steps, **Select the template** and **Create the injector contract**.

:::example[What Grand Meridian's Velvet Harbor scenario uses]
The scenario that replays Velvet Harbor against Grand Meridian Hotels uses four injectors:

| Inject | Injector | Kind |
| --- | --- | --- |
| The fake booking request for the European Hospitality Summit 2026, sent to the front desks | Phishing | Built in |
| The loader's PowerShell actions on the front-desk workstations | OpenAEV Implant, through the OpenAEV agent or Meridian EDR | Built in, needs an executor |
| A scan of the booking database server's exposed services | Nmap | External, run by XTM Composer |
| The incident case for the CERT, in OpenCTI | OpenCTI | From the catalog |

Only Nmap needs a container: the platform engineer deploys it from the **Available** tab, and it shows as started in the **Deployed** tab before the simulation starts.
:::

::screenshot{id=openaev-integrations-injectors-deployed alt="The Deployed tab of Integrations filtered on injectors: the built-in Challenges, Email, Manual, Media pressure, OpenAEV Implant and Phishing injectors, all started."}

## What's next

- [Built-in injectors and their injects](/openaev/use/injectors/built-in/)
- [Install the OpenAEV agent on your endpoints](/openaev/use/agents/), for the injects that run on endpoints.
- [How integrations run](/deploy/integrations/how-they-run/) and [XTM Composer](/deploy/integrations/composer/), to deploy external injectors.
- [Build an injector](/develop/build/injector/), to write your own.
