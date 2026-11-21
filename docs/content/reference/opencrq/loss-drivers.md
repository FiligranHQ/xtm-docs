---
title: OpenCRQ loss drivers
description: The loss drivers and loss categories a risk scenario prices in this version of OpenCRQ,
  what each covers and what its amount scales with, generated from the source code.
sidebar_label: Loss drivers
doc_type: reference
products:
  - opencrq
personas: [risk-manager, executive]
generated: true
generator: reference:opencrq:loss-drivers
---

::diagram{id=reference-opencrq-loss-drivers}

A loss driver is one line of what an incident costs: the forensics firm, the downtime, the regulator's fine. You give each driver a range (minimum, most likely, maximum); OpenCRQ draws from it in each simulated year and reports the result by loss category. Which drivers a scenario should price depends on its outcome and on what its primary assets hold: see the [business resources](/reference/opencrq/business-resources/).
