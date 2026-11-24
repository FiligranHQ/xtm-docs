---
title: OpenCRQ business resources
description: The business resources a primary asset holds in this version of OpenCRQ, and the loss
  drivers each attack outcome recommends for each of them, generated from the source code.
sidebar_label: Business resources
doc_type: reference
products:
  - opencrq
personas: [risk-manager]
generated: true
generator: reference:opencrq:business-resources
---

::diagram{id=reference-opencrq-business-resources}

A business resource says what a primary asset holds of value: personal data, payment card data, operations that must keep running. When a risk scenario's outcome hits a resource, OpenCRQ recommends the [loss drivers](/reference/opencrq/loss-drivers/) that pairing opens, and derives a range from the risk scope's financial profile for those drivers only. Any other driver can still be priced by hand.
