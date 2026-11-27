---
title: OpenCRQ connectors
description: The connectors of this version of OpenCRQ, what each brings, how and when it reads, the
  settings it reads and what a tenant needs before it runs, generated from the source code.
sidebar_label: Connectors
doc_type: reference
products:
  - opencrq
personas: [admin, platform-engineer]
generated: true
generator: reference:opencrq:connectors
---

::diagram{id=reference-opencrq-connectors}

A connector brings one kind of fact into a tenant: threats from OpenCTI, test results from OpenAEV with the assets it tests and their vulnerabilities, assets and vulnerabilities from Microsoft Defender, CVE details from public sources, compliance evidence from Drata, and default threats derived from MITRE ATT&CK. Each runs for one tenant with that tenant's credentials and service account, so two tenants can read two different OpenCTI platforms.
