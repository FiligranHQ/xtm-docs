---
title: OpenCRQ control catalogue
description: The controls of the defense catalogue in this version of OpenCRQ, the ATT&CK techniques
  each prevents and detects, what it maps to, and what each strength level means, generated from the
  source code.
sidebar_label: Control catalogue
doc_type: reference
products:
  - opencrq
personas: [risk-manager]
generated: true
generator: reference:opencrq:controls
---

::diagram{id=reference-opencrq-controls}

A control is a device of the catalogue that prevents or detects at least one ATT&CK technique. You declare the controls you have, each with the strength level it reaches and the share of your estate it covers; OpenAEV's test results then show how they held, and lower a declaration the tests contradict. OpenCRQ prices how each attack meets them on its path to your primary assets.
