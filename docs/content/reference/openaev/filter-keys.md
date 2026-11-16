---
title: OpenAEV filter keys
description: Every filter key OpenAEV accepts in this version, in its lists, its search endpoints and its dashboard widgets, with values and operators, generated from the source code.
sidebar_label: Filter keys
doc_type: reference
products:
  - openaev
personas: [developer, validation-engineer]
generated: true
generator: reference:openaev:filter-keys
---

::diagram{id=reference-openaev-filter-keys}

An OpenAEV list filters on the properties its objects declare as filterable, and so do the search endpoints of the API, such as `POST /api/scenarios/search`: a key the object does not declare is refused. The platform describes the keys of each object at `POST /api/schemas/<object>`, which the filter menus read to offer them. Custom dashboards filter the documents of the search engine, whose keys are listed in the last sections. The format of a filter group is in [Filters](/develop/apis/filters/#openaev-filters).
