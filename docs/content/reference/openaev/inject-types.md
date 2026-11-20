---
title: OpenAEV inject types
description: The built-in injectors of this version of OpenAEV, the inject types each one registers, their fields, API keys and expectations, generated from the source code.
sidebar_label: Inject types
doc_type: reference
products:
  - openaev
personas: [developer, exercise-planner, validation-engineer]
generated: true
generator: reference:openaev:inject-types
---

::diagram{id=reference-openaev-inject-types}

An inject type is a contract that an injector registers in the platform: a form, the targets it reaches and the expectations it offers. When you add an inject to a scenario, you pick a contract and fill in its fields; through the API, you send the contract's identifier in `inject_injector_contract` and the field values under their keys. This page lists the injectors OpenAEV ships with and every field of their contracts. When to use each one is in [Built-in injectors and their injects](/openaev/use/injectors/built-in/). Injectors deployed beside the platform, such as those of the catalog, register their own contracts: their page in **Integrations** lists them.

::screenshot{id=openaev-injector-email-contracts alt="The Email injector's page in Integrations lists the contracts it registers, Send individual mails and Send multi-recipients mail, with the date each was last updated."}
