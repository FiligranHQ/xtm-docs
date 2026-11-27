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

OpenCRQ reads an environment value made only of digits, or `true` or `false`, as a number or a boolean: a password or a token must hold other characters, or the platform refuses it at boot.

The `telemetry` block is stricter than the others: a `TELEMETRY__*` variable it does not know, such as a name of an earlier build, stops the platform too. The table shows the block as one key, `TELEMETRY`; its description lists the variables of its two groups, usage telemetry and observability.
