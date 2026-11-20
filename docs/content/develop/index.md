---
title: Integrate and extend
description: Build connectors, injectors and collectors, use the GraphQL, REST
  and MCP APIs, and contribute to the XTM products.
doc_type: landing
hide_table_of_contents: true
sidebar_label: Overview
products:
  - suite
  - opencti
  - openaev
  - xtm-one
sidebar_position: 0
---

:::product-hero{eyebrow="For developers and integrators" title="Connect your tools and your code to the XTM products" primary-label="Start here" primary-to="#start-here" secondary-label="Choose an API" secondary-to="/develop/apis/" diagram=develop-overview}
Everything the interfaces do, your code can do: read and write knowledge in OpenCTI, script the simulations of OpenAEV, talk to the agents of XTM One. These pages explain the integration model the products share, how to build an integration, how to call each API, and how to contribute to the products.
:::

## Start here

Pick what you build: each tab gives your goal and the pages to read first, in order.

::::tabs
:::tab[A script or an app]
You read or change data from your own code, at the pace of your script.

1. [APIs and integration options](/develop/apis/): what each way in is for, how they authenticate, and which to choose.
2. [Query OpenCTI with GraphQL](/develop/apis/graphql/): a token, then queries, mutations, pages of results and errors.
3. [Script OpenCTI in Python](/develop/build/python-client/): the same with pycti, with helpers per entity type.
4. [Script OpenAEV and XTM One with their REST APIs](/develop/apis/rest/): a token of your user, the endpoints, the errors.
:::
:::tab[An AI agent or assistant]
You let an assistant or your own agent work with the products.

1. [MCP servers](/develop/apis/mcp/): the Model Context Protocol servers XTM One exposes for OpenCTI, OpenAEV, OpenCRQ and itself, and every tool they offer.
2. [XTM One chat API](/develop/apis/chat/): send a message to an agent over HTTP and read its answer as it comes.
3. [XTM One emergency stop API](/develop/apis/emergency-stop/): stop what the agents do, from your own tooling.
:::
:::tab[An integration]
You bring a source, a security tool or an attack technique to the platforms.

1. [Integration model](/develop/integration-model/): connectors, collectors, injectors and executors, and what every integration shares.
2. [Build an OpenCTI connector](/develop/build/connector/): from an empty folder to reports and indicators in OpenCTI, then a Docker image.
3. [Build an OpenAEV injector](/develop/build/injector/): from its contract to the result of each inject.
4. [Build an OpenAEV collector](/develop/build/collector/): match the expectations of your simulations with the alerts of your security tool.
5. [Integration manifest](/develop/build/manifest/): what the catalog shows and XTM Composer deploys from.
:::
:::tab[A contribution]
You change a product or its documentation, and open a pull request.

1. [Build and run from source](/develop/contribute/build-from-source/): OpenCTI or OpenAEV from a clone, with the checks of the CI.
2. [Add or change a translation in OpenCTI or OpenAEV](/develop/contribute/translations/): a new text of the interface, in every language.
3. [Develop XTM Composer](/develop/contribute/xtm-composer/): build it in Rust and run it against a local OpenCTI.
4. [Write a page of the XTM documentation](/develop/contribute/docs/): from a template to a page checked as the CI does.
:::
::::

## Pull, listen or push

::::split
:::column
The ways into OpenCTI differ in who drives the exchange. Your code pulls with the GraphQL API; a consumer listens to a live stream that sends every change, deletions included; a device downloads a feed on a schedule. Choosing the right one saves you polling, and saves your tools stale data.

:::cards{cols=2}
  ::card[What each way in is for, and how to pick one.]{icon=BookOpen title="APIs and integration options" href="/develop/apis/"}
  ::card[Every change of the knowledge base, as it happens.]{icon=Activity title="Data streaming" href="/develop/apis/streams/"}
:::
:::
:::column
::diagram{id=develop-apis}
:::
::::

:::xtm-one[Agents use the same doors]
The AI agents of XTM One reach OpenCTI, OpenAEV and OpenCRQ through the MCP servers XTM One exposes, with the rights of the user they work for. An assistant of your own can use the same servers. See [MCP servers](/develop/apis/mcp/).
:::

## Browse Develop

::section-overview{id=develop}
