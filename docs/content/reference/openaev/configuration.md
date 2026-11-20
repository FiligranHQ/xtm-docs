---
title: OpenAEV configuration
description: Every OpenAEV configuration key of this version, with its environment variable, default and
  what it does, generated from the source code.
sidebar_label: Configuration
doc_type: reference
products:
  - openaev
personas: [platform-engineer, developer]
generated: true
generator: reference:openaev:configuration
---

::diagram{id=reference-openaev-configuration}

OpenAEV is a Spring Boot application: an environment variable overrides the key the release sets in its `application.properties`, and a key neither sets takes the default its code holds. In Docker, set the environment variable: the key in upper case with each dot replaced by an underscore, dashes kept, such as `OPENAEV_BASE-URL` for `openaev.base-url`.

The first administrator comes from `OPENAEV_ADMIN_EMAIL`, `OPENAEV_ADMIN_PASSWORD` and `OPENAEV_ADMIN_TOKEN`, which the platform checks at startup: the email must be a valid address, the token a UUID. Where the code still reads a key under the former name `openbas.*`, that name wins over its `openaev.*` twin.
