---
title: OpenCRQ configuration
description: Every OpenCRQ configuration key of this version, with its environment variable, type,
  default and whether its block needs it, generated from the source code.
sidebar_label: Configuration
doc_type: reference
products:
  - opencrq
personas: [platform-engineer, developer]
generated: true
generator: reference:opencrq:configuration
---

::diagram{id=reference-opencrq-configuration}

OpenCRQ checks its whole configuration when it starts, against its configuration schema: a key with a wrong type, or a required key left blank, stops the platform with a message that names it. XTM One is required: `XTM_ONE__URL`, `XTM_ONE__TOKEN` and `XTM_ONE__PLATFORM_URL` must be set, and the platform registers with XTM One, which also grants its edition.

The technical names keep the product's former name, OpenGRC: the default bucket `opengrc`, the telemetry service `opengrc-platform`. Give them as they are.
