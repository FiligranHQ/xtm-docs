---
title: OpenCRQ connectors
description: The connectors of this version of OpenCRQ, what each brings, how and when it reads, and
  the settings that turn it on for a tenant, generated from the source code.
sidebar_label: Connectors
doc_type: reference
products:
  - opencrq
personas: [admin, platform-engineer]
generated: true
generator: reference:opencrq:connectors
---

::diagram{id=reference-opencrq-connectors}

A connector brings one kind of fact into a tenant: threats from OpenCTI, test results from OpenAEV, assets and vulnerabilities from Microsoft Defender, CVE details from public sources, compliance evidence from Drata. Each runs for one tenant with that tenant's credentials, so two tenants can read two different OpenCTI platforms.
