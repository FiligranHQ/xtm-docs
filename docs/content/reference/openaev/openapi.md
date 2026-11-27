---
title: OpenAEV OpenAPI
description: Where OpenAEV serves the OpenAPI description of its REST API and its Swagger UI, the settings that change them, what the description covers, and how to authenticate the calls you try from it.
sidebar_label: OpenAPI
doc_type: reference
products:
  - openaev
personas: [developer, platform-engineer]
owner: docs-openaev
last_reviewed: 2026-11-09
---

OpenAEV describes its REST API in an OpenAPI document that the platform builds from its own code each time it starts, with springdoc-openapi. The description always matches the version that runs: every endpoint of the API, its parameters, its request and response bodies, and the role it requires. Read it to write a client, generate one, or try an endpoint from the browser in Swagger UI. How to authenticate, paginate and filter is in [REST API](/develop/apis/rest/).

::diagram{id=reference-openaev-openapi}

## Where to find it

| What | Path | Notes |
| --- | --- | --- |
| OpenAPI description, in JSON | `/api-docs` | Set by `springdoc.api-docs.path` |
| Swagger UI | `/swagger-ui/index.html` | Reads the description above and lets you call the endpoints |

The paths are relative to the address of the platform, such as `https://openaev.example.com/api-docs`. When OpenAEV runs under a sub-path, set with `server.servlet.context-path`, both paths are under it too.

In the interface, open your **Profile**: the **API access** section shows your token and an example request, and its **API specifications** button opens Swagger UI. The button points to `/swagger-ui/index.html` at the root of the host: under a sub-path, type the address with the sub-path yourself.

## Settings

OpenAEV ships these springdoc keys in its `application.properties`. Set them as any key of the platform, with the environment variable in the second column.

| Key | Environment variable | Value shipped | Effect |
| --- | --- | --- | --- |
| `springdoc.api-docs.path` | `SPRINGDOC_API-DOCS_PATH` | `/api-docs` | Path of the OpenAPI description |
| `springdoc.swagger-ui.validatorUrl` | `SPRINGDOC_SWAGGER-UI_VALIDATORURL` | `none` | Swagger UI does not send the description to an external validator |
| `springdoc.override-with-generic-response` | `SPRINGDOC_OVERRIDE-WITH-GENERIC-RESPONSE` | `false` | Does not add the responses of the platform's global error handlers to every operation: a workaround for an incompatibility between Spring Boot 3.5.7 and later and springdoc 2.8.15 and earlier |

The [configuration reference](/reference/openaev/configuration/#springdoc) lists them with every other key.

## What the description covers

- **The REST API of the platform**: every endpoint of its controllers, under `/api/` and, for the tenant-scoped ones, `/api/tenants/{tenantId}/`. The context of a request that the platform resolves itself from the path or the headers is left out of the parameters.
- **Its identity**: the title `OpenAEV API`, the version of the platform that serves it, the Apache 2.0 license, and a link to the documentation.
- **The role each endpoint requires**: the description of an operation ends with **Required role**, and with **Special Requirement** when the endpoint also checks a right on the object, such as being a planner or an observer of the simulation.
- **One security scheme**: `JSESSIONID`, the session cookie of the browser. The description does not declare the API key, which the API accepts as well: see below.

The interface of OpenAEV uses the same description: the TypeScript types of its API calls are generated from it.

## Try the API from Swagger UI

Sign in to OpenAEV in the same browser, then open Swagger UI: the calls you try carry your session cookie, and run with your rights. A `POST`, `PUT` or `DELETE` also needs the CSRF token the interface sends; for those, and for scripts, use your API key with a client instead:

```bash
curl -fsS https://openaev.example.com/api/scenarios \
  -H "Authorization: Bearer $OPENAEV_TOKEN"
```

`OPENAEV_TOKEN` is the **Token key** of the **API access** section of your profile.

## Download the description

Save the description of the version you run, for a client generator or to compare two versions:

```bash
curl -fsS https://openaev.example.com/api-docs -o openaev-api.json
jq -r '.info.version, (.paths | length)' openaev-api.json
```

The first line printed is the version of the platform, such as `3.261029.0`; the second, the number of paths it describes. Download it again after each upgrade: the description changes with the API.

## What's next

- [REST API](/develop/apis/rest/): authentication, pagination, errors and the multi-tenant paths.
- [Filters](/develop/apis/filters/#openaev-filters) and [OpenAEV filter keys](/reference/openaev/filter-keys/): the `filterGroup` of the search endpoints.
- [OpenAEV inject types](/reference/openaev/inject-types/): the contracts and fields an inject takes.
