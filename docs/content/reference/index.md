---
title: Reference
description: Exact settings, fields, components and keys of each XTM product,
  generated from the source code of each release.
doc_type: landing
hide_table_of_contents: true
sidebar_label: Overview
products:
  - suite
  - opencti
  - openaev
  - composer
  - xtm-one
  - opencrq
sidebar_position: 0
---

:::product-hero{eyebrow="Generated from the source code" title="The exact name of every setting, capability and key" primary-label="Find what you look for" primary-to="#find-what-you-look-for" secondary-label="Browse by product" secondary-to="#browse-reference" diagram=reference-overview}
The guides explain how to do things; the reference lists what exists: every configuration key with its environment variable and its default, every capability of a role, every filter key, every API operation. Its pages are generated from the source code of each release, so they match the version you run. Each page says which code it comes from.
:::

## Find what you look for

A reference page answers one question: what is the exact name, and what does it accept? Start from what you look for, then pick your product.

| You look for | XTM One | OpenCTI | OpenAEV | OpenCRQ | XTM Composer |
| --- | --- | --- | --- | --- | --- |
| A setting, its environment variable and its default | [Configuration](/reference/xtm-one/configuration/) | [Configuration](/reference/opencti/configuration/), [integration settings](/reference/opencti/integration-settings/) | [Configuration](/reference/openaev/configuration/) | [Configuration](/reference/opencrq/configuration/) | [Configuration keys](/reference/composer/configuration/) |
| What a role may do | | [Capabilities](/reference/opencti/capabilities/) | [Capabilities](/reference/openaev/capabilities/) | [Capabilities](/reference/opencrq/capabilities/) | |
| An API operation or an endpoint | [API](/reference/xtm-one/api/) | [GraphQL schema](/reference/opencti/graphql-schema/), [HTTP endpoints](/reference/opencti/http-endpoints/) | [OpenAPI](/reference/openaev/openapi/) | [API](/reference/opencrq/api/) | |
| A tool an AI assistant may call | [MCP tools](/reference/xtm-one/mcp-tools/) | | | | |
| A key to filter on | | [Filter keys](/reference/opencti/filter-keys/) | [Filter keys](/reference/openaev/filter-keys/) | | |
| What the product ships with | | [Data model](/reference/opencti/data-model/), [vocabularies](/reference/opencti/vocabularies/), [inference rules](/reference/opencti/inference-rules/), [decay rules](/reference/opencti/decay-rules/), [playbook components](/reference/opencti/playbook-components/) | [Inject types](/reference/openaev/inject-types/) | [Attack outcomes](/reference/opencrq/attack-outcomes/), [business resources](/reference/opencrq/business-resources/), [controls](/reference/opencrq/controls/), [loss drivers](/reference/opencrq/loss-drivers/), [connectors](/reference/opencrq/connectors/) | |
| What runs in the background and what it reports | | [Managers](/reference/opencti/managers/), [metrics](/reference/opencti/metrics/), [audit events](/reference/opencti/audit-events/), [errors](/reference/opencti/errors/) | | [Background processes](/reference/opencrq/processes/) | |
| A feature still in development | | [Feature flags](/reference/opencti/feature-flags/) | | [Feature flags](/reference/opencrq/feature-flags/) | |

## How to use it

::::tabs
:::tab[Platform engineers]
You set a deployment up and need the exact key. Open the configuration page of the product, filter on a word, and copy the environment variable into your `.env`. The guides of [Deploy](/deploy/) link to the section of each key they mention.
:::
:::tab[Administrators]
You build a role and need to know what each capability opens. The capabilities page of the product lists them by the label the role editor shows.
:::
:::tab[Developers]
You write a query, a filter or an integration. The schema, the endpoints and the filter keys give the exact names and types; [Develop](/develop/) explains how to call them.
:::
:::tab[AI agents]
An agent that deploys, configures or queries a product reads these pages as Markdown: every page has a Markdown twin, and [llms.txt](/llms.txt) lists them all.
:::
::::

## Browse Reference

::section-overview{id=reference}
