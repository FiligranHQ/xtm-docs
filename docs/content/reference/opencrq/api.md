---
title: OpenCRQ API
description: Every procedure of the OpenCRQ API in this version, its kind and its input, generated
  from the source code.
sidebar_label: API
doc_type: reference
products:
  - opencrq
personas: [developer]
generated: true
generator: reference:opencrq:api
---

::diagram{id=reference-opencrq-api}

The interface of OpenCRQ and the agents of XTM One call the same procedures: whatever a person can do in the interface, a script can do through the API with the rights of its account. A script authenticates as a service account, created by an administrator in its tenant, with the token it gets.
