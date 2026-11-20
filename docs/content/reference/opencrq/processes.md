---
title: OpenCRQ background processes
description: The jobs and engines OpenCRQ runs in the background in this version, what each does and
  when it runs, generated from the source code.
sidebar_label: Background processes
doc_type: reference
products:
  - opencrq
personas: [admin, platform-engineer]
generated: true
generator: reference:opencrq:processes
---

::diagram{id=reference-opencrq-processes}

Most of what OpenCRQ computes happens after a change, not when you open a page: a scenario saved, a test result imported, a new day. The first change of a burst books a run of each job that read the changed data, a little later, and the changes that arrive until then fold into that run: however fast an import writes, a job runs at most once per delay (`recompute.debounce_*` in the [configuration](/reference/opencrq/configuration/#recompute)).
