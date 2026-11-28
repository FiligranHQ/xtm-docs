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
| External | As its own container or Python program | Nmap, Nuclei, HTTP query, Shodan, NetExec | Run it with XTM Composer from :menu[Integrations], which requires an Enterprise Edition license, or run it yourself |

The catalog, in :menu[Integrations], **Available** tab, lists the external injectors the platform knows: reconnaissance (Censys, DNS Enumeration, Nmap, Shodan), vulnerability scanning (Nuclei), attack simulation (HTTP query, Amazon Web Services, NetExec, Stratus Red Team), communication (Microsoft Teams, Slack, email through Microsoft 365, Google Workspace or SMTP) and AI security (AI Red Team).

## How an injector gets its injects

The platform decides when each inject runs. Then:

- A **built-in** injector runs the inject in the platform: it sends the email through the platform's mail server, or publishes the article on the player interface.
- An **external** injector registers with the platform when it starts, with its URL, token and identifier. The platform answers with the connection details of RabbitMQ, and puts each inject in the injector's own queue. So the injector must reach RabbitMQ under the host name and port the platform gives it. Deployed from the catalog with XTM Composer, it registers with the token of the tenant's **Catalog connectors** service account, which every integration of the catalog shares: see [Run a collector](/openaev/use/collectors/#run-a-collector). How an external integration reaches its platform, with XTM Composer or without it, is in [How integrations run](/deploy/integrations/how-they-run/).

The inject picker of scenarios, simulations and atomic tests offers only the contracts of an injector registered in the tenant. A contract whose injector has not registered would fail at launch, so it stays hidden until its injector registers.

## Injectors that act on endpoints

Some contracts run on endpoints: the attack actions of the Threat Arsenal, run by the **OpenAEV Implant** injector, and the contracts of external injectors built for endpoints. An injector of this kind does not reach the endpoint itself. When it registers, it tells the executors how to start it on Windows, Linux and macOS, and an executor starts it on each target: the OpenAEV agent or your EDR. So these injects need an executor on their targets. See [Choose between the OpenAEV agent and your EDR](/openaev/use/executors/).

## See your injectors and their inject types

Open :menu[Integrations], **Active** tab. Each injector card shows its status and, under **Last Seen**, when the platform last heard from it:

- **Running**: the injector runs inside the platform, or it is external and the platform heard from it in the last two minutes. An injector that runs inside the platform shows **Always on** under **Last Seen**.
- **Not responding**: XTM Composer started the external injector, but the platform has not heard from it for more than two minutes. Check its container and its **Logs** tab.
- **Stopped**: the injector was stopped from its page, whether XTM Composer or the platform runs it, or the external injector you run yourself has not called the platform for more than two minutes.

Open an injector to see its contracts, with their **Kill chains**, **Domains** and **Attack patterns**. An injector from the catalog also shows, in its **Overview**, the **Commit hash** of the build it reported when it registered and, when XTM Composer runs it, the **Image digest** of its container: the digest tells which build runs behind a moving tag such as `latest`. An injector deployed by XTM Composer also has a **Logs** tab.

Some injectors accept contracts of your own, built from one of theirs: their contract list then has a **Create** button that opens **Create a new action**, in two steps, **Select the template** and **Create the action**.

:::example[What Grand Meridian's Velvet Harbor scenario uses]
The scenario that replays Velvet Harbor against Grand Meridian Hotels uses four injectors:

| Inject | Injector | Kind |
| --- | --- | --- |
| The fake booking request for the European Hospitality Summit 2026, sent to the front desks | Phishing | Built in |
| The loader's PowerShell actions on the front-desk workstations | OpenAEV Implant, through the OpenAEV agent or Meridian EDR | Built in, needs an executor |
| A scan of the booking database server's exposed services | Nmap | External, run by XTM Composer |
| The incident case for the CERT, in OpenCTI | OpenCTI | From the catalog |

Only Nmap needs a container: the platform engineer deploys it from the **Available** tab, and it shows as **Running** in the **Active** tab. Once it has registered, its scan contracts appear in the inject picker of the scenario.
:::

::screenshot{id=openaev-integrations-injectors-deployed alt="The Active tab of Integrations filtered on injectors: the built-in Challenges, Email, Manual, Media pressure, OpenAEV Implant and Phishing injectors, all Running and Always on."}

## What's next

- [Built-in injectors and their injects](/openaev/use/injectors/built-in/)
- [Install the OpenAEV agent on your endpoints](/openaev/use/agents/), for the injects that run on endpoints.
- [How integrations run](/deploy/integrations/how-they-run/) and [XTM Composer](/deploy/integrations/composer/), to deploy external injectors.
- [Build an injector](/develop/build/injector/), to write your own.
