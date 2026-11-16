---
title: OpenAEV capabilities
description: The capabilities a role grants in this version of OpenAEV, in a tenant and on the platform,
  generated from the source code.
sidebar_label: Capabilities
doc_type: reference
products:
  - openaev
personas: [admin]
generated: true
generator: reference:openaev:capabilities
---

::diagram{id=reference-openaev-capabilities}

A role grants capabilities, a group holds roles, and a user gets the capabilities of the roles of their groups. Most capabilities apply in a tenant: scenarios, simulations, targets, components and the tenant's settings. A few apply to the whole platform: its settings, its tenants, and **Bypass**, which gives every right.
