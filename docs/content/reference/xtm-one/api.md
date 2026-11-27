---
title: XTM One API
description: Every route of the XTM One REST API in this version, its method, its path and what it does,
  generated from the source code.
sidebar_label: API
doc_type: reference
products:
  - xtm-one
personas: [developer]
generated: true
generator: reference:xtm-one:api
---

::diagram{id=reference-xtm-one-api}

The interface of XTM One calls these routes, so a script can do what a person does on screen, with the rights of the user whose API key it sends. A user creates a key in **My Profile** > **API Keys**, with the **Platform API** surface. A read-only key sends only `GET`, `HEAD` and `OPTIONS` requests: any other method is refused, even a `POST` that only searches.

[Script OpenAEV and XTM One with their REST APIs](/develop/apis/rest/) shows a first request and how to read the errors. To talk to an agent from your code, see [XTM One chat API](/develop/apis/chat/).
