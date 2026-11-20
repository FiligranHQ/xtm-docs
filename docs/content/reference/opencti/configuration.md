---
title: OpenCTI configuration
description: Every OpenCTI configuration key of this version, with its environment variable,
  default and description, generated from the OpenCTI source code.
sidebar_label: Configuration
doc_type: reference
products:
  - opencti
personas: [developer, platform-engineer, admin]
generated: true
generator: opencti-configuration
---

::diagram{id=reference-opencti-configuration}

Every key OpenCTI reads in this version: the keys of `config/default.json`, and the keys the code reads without declaring them there. Each has an environment variable, the way to set it in Docker: its name is the key in upper case, with `__` between the levels, so `app:base_url` is `APP__BASE_URL`. In the XTM stack, set these variables in [`docker-compose.override.yml`](/deploy/install/docker-compose/#change-the-stack-without-editing-docker-compose-yml); in an installation without containers, set the key in `config/production.json`.

A list goes in the variable as JSON: `APP__ADMIN__EXTERNALLY_MANAGED_ROLES=["Administrator"]`. A key marked **Not in default.json** works the same way; its default is the one the code applies.
