---
title: Script OpenAEV and XTM One with their REST APIs
sidebar_label: REST API
description: Authenticate to the REST APIs of OpenAEV and XTM One with a token of your user, find every endpoint in their API reference, page through results and read the errors.
doc_type: how-to
products:
  - openaev
  - xtm-one
personas: [developer, platform-engineer]
owner: docs-platform
last_reviewed: 2026-11-09
sidebar_position: 2
migrated_from:
  - openaev:/reference/apis/rest-api/
---

Two modules of the XTM platform have a REST API: OpenAEV, under `/api`, and XTM One, under `/api/v1`. Their interfaces call the same endpoints, so a script can do what a person does on screen, with the rights of the user whose token it sends. This page gets a token for each, sends a first request, and covers what both APIs share: the API reference, pagination, tenants and errors. OpenCTI has a GraphQL API instead: see [Query OpenCTI with GraphQL](/develop/apis/graphql/). OpenCRQ exposes its procedures through its own API: see the [OpenCRQ API reference](/reference/opencrq/api/).

::diagram{id=develop-apis-rest}

| Module | Base path | Credential | API reference |
| --- | --- | --- | --- |
| OpenAEV | `/api`, and `/api/tenants/{tenantId}` for one tenant | The API token of a user, from **Profile** > **API access** | Swagger UI at `/swagger-ui/index.html`, OpenAPI document at `/api-docs` |
| XTM One | `/api/v1` | A personal API key (`fcp-...`), from **My Profile** > **API Keys** | `/api/reference`, OpenAPI document at `/api/openapi.json` |

## Before you start

- A user account for the script, with the rights of the operations it calls. A token never does more than its user: give an integration its own account, not a person's.
- `curl` and `jq` for the examples. Any HTTP client works: both APIs take and return JSON.
- The examples read the addresses and tokens from environment variables. In a shell, `read -rs` reads a token without showing it or writing it in the history:

  ```bash
  export OPENAEV_URL=https://openaev.example.com
  export XTM_ONE_URL=https://xtm-one.example.com
  read -rs OPENAEV_TOKEN && export OPENAEV_TOKEN
  read -rs XTM_ONE_API_KEY && export XTM_ONE_API_KEY
  ```

## Call the OpenAEV API

### Get a token

Each OpenAEV user has one API token. Open the profile from the account menu at the top right, then **Profile**: the **API access** section shows the **Token key**, and **API specifications** opens the Swagger UI. **RENEW** deletes the token and creates a new one at once: every script that used the old one stops working until you give it the new one. See [Set up your profile](/openaev/use/profile/#use-your-api-token).

::screenshot{id=openaev-profile-api-access alt="The API access section of the OpenAEV profile: the Token key, the RENEW button, an example request to /api/exercises that passes the token in its Authorization header, and API specifications."}

### Send a first request to OpenAEV

Send the token in the `Authorization` header, with the `Bearer` scheme. A request with a token is stateless: OpenAEV keeps no session for it.

```bash
curl -s "$OPENAEV_URL/api/exercises" \
  -H "Authorization: Bearer $OPENAEV_TOKEN" \
  -H "Content-Type: application/json" | jq '.[0] | {exercise_id, exercise_name, exercise_status}'
```

Send only the `Authorization` header from a script. A request that also carries the session cookie of the interface is checked like a browser request, and its `POST`, `PUT` and `DELETE` need the cross-site request forgery (CSRF) token the interface sends.

### Work in one tenant

A platform can hold several tenants, each with its scenarios, simulations, assets and teams. Most resource endpoints exist twice: under `/api/...`, and under `/api/tenants/{tenantId}/...` for one tenant. The tenant identifier is the segment after the host in the address of the tenant in your browser. A user who does not belong to the tenant gets `403`. See [Give each organization its own tenant](/openaev/administer/multi-tenancy/).

### Search and page through results

Search endpoints take a `POST` with the page, the sort and the filters in the body, such as `POST /api/scenarios/search`:

| Field | Effect |
| --- | --- |
| `page` | Page number, from `0`. Default `0` |
| `size` | Items per page. Default `20`, at most `1000` |
| `textSearch` | Free text matched on the searchable attributes |
| `sorts` | A list of `{"property": ..., "direction": "asc"}`; `asc` when the direction is left out |
| `filterGroup` | A filter group: see [Filters](/develop/apis/filters/) |

The answer is a page: the items in `content`, the total in `totalElements`, and `totalPages`, `number` and `size`.

:::example[List the Velvet Harbor scenarios of Grand Meridian Hotels]
For the weekly report to Hélène Marchetti, the CISO, a script of the validation team lists the scenarios of the Grand Meridian Hotels tenant that mention Velvet Harbor, by name. The tenant's identifier, read from its address, is in `GRAND_MERIDIAN_TENANT_ID`:

```bash
curl -s "$OPENAEV_URL/api/tenants/$GRAND_MERIDIAN_TENANT_ID/scenarios/search" \
  -H "Authorization: Bearer $OPENAEV_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"page": 0, "size": 20, "textSearch": "Velvet Harbor",
       "sorts": [{"property": "scenario_name", "direction": "asc"}]}' \
  | jq '{total: .totalElements, scenarios: [.content[] | {scenario_name, scenario_severity, scenario_recurrence}]}'
```

The same script creates the scenario for the next replay. Only `scenario_name` is required; `scenario_severity` takes `low`, `medium`, `high` or `critical`:

```bash
curl -s "$OPENAEV_URL/api/tenants/$GRAND_MERIDIAN_TENANT_ID/scenarios" \
  -H "Authorization: Bearer $OPENAEV_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"scenario_name": "Velvet Harbor booking lure, May 2026",
       "scenario_description": "Replays the fake booking request of the European Hospitality Summit 2026 on the front desks.",
       "scenario_severity": "high"}' | jq '{scenario_id, scenario_name}'
```

The scenario appears in the tenant's list, owned by the user of the token, ready for its injects.
:::

### Read the errors of OpenAEV

| Code | Meaning |
| --- | --- |
| `400` | The input is invalid: a validation error lists the fields at fault, other errors carry a `message` |
| `401` | No token, or an unknown one |
| `403` | The user lacks the capability or the grant, or does not belong to the tenant |
| `404` | The resource does not exist, or the user cannot see it |
| `409` | The change conflicts with the data, such as a duplicate |

## Call the XTM One API

### Get an API key

In XTM One, open **My Profile** from the user menu, then the **API Keys** tab, and click **Create Key**. Keep **Platform API** in **Surfaces**, choose **Read-only** for a script that only reads, and an **Expiry**. Copy the key when it shows: it is never shown again. See [Set up your profile and API keys](/xtm-one/use/profile/#create-an-api-key).

::screenshot{id=xtm-one-profile-api-key alt="The top of the New API key dialog: the Name of the key, and its two Surfaces, Platform API for the REST API under /api/v1 and MCP endpoints, both on."}

A read-only key sends only `GET`, `HEAD` and `OPTIONS` requests: any other method is refused, even a `POST` that only searches. A key limited to agents reaches only the routes of those agents and their conversations. Its own **Requests per minute** and **Quota** count on top of the quotas of its user.

### Send a first request to XTM One

```bash
curl -s "$XTM_ONE_URL/api/v1/agents" \
  -H "Authorization: Bearer $XTM_ONE_API_KEY" | jq '.[] | {name, slug, enabled}'
```

The answer lists the agents the key's user can see: their own, those shared with them or their groups, and those of the company. The [XTM One API reference](/reference/xtm-one/api/) lists every route; your platform serves them with their fields at `/api/reference`, generated from the code of your version. To talk to an agent with an API key, create a conversation with `POST /api/v1/chat/conversations`, then send each message with `POST /api/v1/chat/conversations/{conversation_id}/messages`, which answers once the agent has replied. The routes under `/api/v1/web/chat`, which stream the answer as server-sent events to the chat screen, take a signed-in session only and refuse an API key with `403`: see [XTM One chat API](/develop/apis/chat/).

### Read the errors of XTM One

XTM One answers an error with a JSON body whose `detail` says why.

| Code | Meaning |
| --- | --- |
| `401` | No key, or a revoked or expired one, or a deactivated user |
| `403` | The key does not cover this surface or method, or the user lacks the permission |
| `429` | The key or its user reached a rate limit or a quota |

## What's next

- [APIs and integration options](/develop/apis/), to choose between the APIs, the streams and the MCP servers.
- [MCP servers](/develop/apis/mcp/), to let an AI client call the modules.
- [Filters](/develop/apis/filters/), the filter group format of OpenAEV searches.
