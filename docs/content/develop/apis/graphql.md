---
title: Query OpenCTI with GraphQL
description: Authenticate with an API token, then read and write OpenCTI knowledge with GraphQL queries and mutations, page through results and handle errors.
doc_type: how-to
products:
  - opencti
personas: [developer, platform-engineer]
owner: docs-platform
last_reviewed: 2026-11-01
sidebar_position: 1
migrated_from:
  - opencti:/reference/api/
  - opencti:/development/api-usage/
---

Everything the OpenCTI interface does goes through one GraphQL API, at `/graphql`, and your scripts and integrations can use it the same way. This page takes you from an API token to working requests on the demo data: list the reports that mention the Velvet Harbor intrusion set, create an indicator, page through long results and read the errors the API returns. Every request and response below was run against OpenCTI 7.261030, with only the long parts trimmed. To choose between this API, the live streams and the feeds, read [APIs and integration options](/develop/apis/) first.

## Before you start

- A user account whose role has the **Allow token usage** capability, under **API usage with authorization header**. Without it, the API refuses the token as if it were wrong.
- The capabilities of the operations you call: **Access knowledge** to read, **Create / Update knowledge** to create and change, **Delete knowledge** to delete. The [GraphQL schema reference](/reference/opencti/graphql-schema/) gives the capability of every query and mutation.
- `curl` and `jq` for the examples. Any HTTP client works: the API takes a JSON body over `POST`.

A request sees only what its user sees: the markings, organizations and confidence level of the user apply to every query and mutation. For an integration, create a dedicated user: see [Create accounts for integrations](/opencti/administer/service-accounts/) explains, and use its token.

## Get an API token

:::steps
1. Open your profile from the account icon at the top right of OpenCTI, then **Profile**.
2. In the **API access** card, click **Generate Token**.

   ::screenshot{id=opencti-profile-api-access alt="The API access card with the OpenCTI version, the Playground and Generate Token buttons, and a token list giving the name, the last four characters, the last use and the expiry date of each token."}

3. In **Generate a new token**, enter a **Name** that says where the token is used, such as `hotel-feed-importer`.
4. Choose a **Duration**: **30 days**, **60 days**, **90 days**, **1 year** or **Unlimited**. After its duration, the API refuses the token.

   ::screenshot{id=opencti-profile-token-generate alt="The Generate a new token panel with the Name field, the Duration list and the Generate button."}

5. Click **Generate**, then copy the token from **Token details**. OpenCTI keeps only a fingerprint of it: once the panel is closed, nobody can read it again.
:::

A token starts with `flgrn_octi_tkn_`. To follow its last use, revoke it or renew it, see [Create and manage API tokens](/opencti/use/profile/#create-and-manage-api-tokens). For a service account, an administrator generates the token from its user account: see [API tokens](/opencti/administer/users/#api-tokens).

Keep the token in a secret store, and pass it to your scripts in an environment variable. The examples below read it from `OPENCTI_TOKEN` and the platform address from `OPENCTI_URL`. In a shell, `read -rs` reads the token without showing it or writing it in the history:

```bash
export OPENCTI_URL=https://opencti.example.com
read -rs OPENCTI_TOKEN && export OPENCTI_TOKEN
```

## Send a first request

A GraphQL request is a `POST` to `/graphql` with a JSON body: `query` holds the operation, `variables` its arguments. The token goes in the `Authorization` header with the `Bearer` scheme.

```bash
curl -s "$OPENCTI_URL/graphql" \
  -H "Authorization: Bearer $OPENCTI_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"query": "{ about { version } me { name user_email } }"}'
```

:::verify[The platform answers with its version and your user]
```json
{"data":{"about":{"version":"7.261030.1"},"me":{"name":"admin","user_email":"docs-admin@example.com"}}}
```
:::

The API accepts only `POST`. A `GET` answers `405` with `{"name":"MethodNotAllowedError","message":"Method Not Allowed. Use POST for GraphQL requests."}`.

Other ways to authenticate exist for other cases:

| Credential | Header | When |
| --- | --- | --- |
| API token | `Authorization: Bearer` followed by the token | Scripts, connectors, integrations. Needs **Allow token usage** |
| Login and password | `Authorization: Basic` followed by `login:password` in Base64 | Tools that only speak basic authentication. Needs **Allow basic auth usage**. An account that logs in through single sign-on has no local password |
| Browser session | The session cookie of the interface | The playground, and GraphQL subscriptions over WebSocket on `/graphql` |

## Query the reports on Velvet Harbor

Most entity types have a list query named after the type in the plural (`reports`, `indicators`, `intrusionSets`) and a single-item query (`report`, `indicator`, `intrusionSet`). Lists are connections: the items are in `edges[].node`, and `pageInfo` describes the page.

:::steps
1. Find the internal ID of the intrusion set. `search` matches close names too, so read the names that come back:

   ```bash
   curl -s "$OPENCTI_URL/graphql" \
     -H "Authorization: Bearer $OPENCTI_TOKEN" -H "Content-Type: application/json" \
     -d '{"query": "{ intrusionSets(search: \"Velvet Harbor\", first: 5) { edges { node { id standard_id name } } } }"}'
   ```

   ```json
   {"data":{"intrusionSets":{"edges":[
     {"node":{"id":"cd92e87a-28a6-4817-81a1-b3305df36c8c","standard_id":"intrusion-set--4e78bb64-59c9-51e3-b1b9-ac8133e1c8e0","name":"Velvet Harbor"}},
     {"node":{"id":"1dfb9b04-ac79-45a9-baba-68777009378f","standard_id":"intrusion-set--0dc7beb8-23f1-5220-ae5b-6d520f761ade","name":"Velvet Harbor Group"}},
     {"node":{"id":"a4763e42-83a3-4dca-9b8d-7870c0bdae3d","standard_id":"intrusion-set--f4c1499d-23a6-5c75-94d5-dd19b39bf509","name":"Velvet Harbour"}}]}}}
   ```

   `id` is the internal ID of OpenCTI; `standard_id` is the STIX ID, computed from the properties that identify the object, so it is the same on every platform. The `id` argument of a single-object query, such as `intrusionSet(id:)`, accepts either. Filter values take the internal ID.

2. List the reports that contain it, newest first. The `objects` filter key matches containers by the objects they hold; [Filters](/develop/apis/filters/) explains the syntax.

   ```bash
   cat > reports-on-velvet-harbor.json <<'EOF'
   {
     "query": "query ReportsOnVelvetHarbor($filters: FilterGroup, $first: Int, $after: ID) { reports(filters: $filters, first: $first, after: $after, orderBy: published, orderMode: desc) { pageInfo { hasNextPage endCursor globalCount } edges { node { id name published report_types confidence createdBy { name } objectMarking { definition } } } } }",
     "variables": {
       "first": 2,
       "filters": {"mode": "and", "filters": [{"key": "objects", "values": ["cd92e87a-28a6-4817-81a1-b3305df36c8c"]}], "filterGroups": []}
     }
   }
   EOF
   curl -s "$OPENCTI_URL/graphql" \
     -H "Authorization: Bearer $OPENCTI_TOKEN" -H "Content-Type: application/json" \
     -d @reports-on-velvet-harbor.json | jq
   ```

   ```json
   {
     "data": {
       "reports": {
         "pageInfo": {
           "hasNextPage": true,
           "endCursor": "WzE3NjgzNzc2MDAwMDAsInJlcG9ydC0tMzdhNmEyOTEtYjhjYi01YTAxLThiMTgtNzc3Y2ZhYzgxMWI1Il0=",
           "globalCount": 10
         },
         "edges": [
           {"node": {"id": "abe88acd-916d-49ba-a01c-3e564ba5e10f", "name": "Velvet Harbor reuses guest Wi-Fi portal infrastructure", "published": "2026-01-15T07:30:00.000Z", "report_types": ["threat-report"], "confidence": 60, "createdBy": {"name": "Example CERT"}, "objectMarking": [{"definition": "TLP:GREEN"}]}},
           {"node": {"id": "0156c67a-c04b-4a9c-a9a2-86b5e5e1aea5", "name": "Velvet Harbor phishing kit mimics hotel booking confirmations", "published": "2026-01-14T08:00:00.000Z", "report_types": ["threat-report"], "confidence": 70, "createdBy": {"name": "Example CERT"}, "objectMarking": [{"definition": "TLP:GREEN"}]}}
         ]
       }
     }
   }
   ```
:::

The list queries share these arguments:

| Argument | Effect |
| --- | --- |
| `first` | Number of items in the page. Without it, a page holds 500 items (`elasticsearch:default_pagination_result`). A larger value than 5,000 is silently cut to 5,000 (`elasticsearch:max_pagination_result`) |
| `after` | The `endCursor` of the previous page |
| `orderBy`, `orderMode` | Sort field, from the ordering enumeration of the type (`ReportsOrdering` for reports), and `asc` or `desc` |
| `search` | Full-text search on the main text fields |
| `filters` | A filter group, described in [Filters](/develop/apis/filters/) |

Ask only for the fields you use: nested fields, such as `createdBy` or `objectMarking`, load other objects. To get an object as STIX 2.1, ask for its `toStix` field, a JSON string. The [GraphQL schema reference](/reference/opencti/graphql-schema/) lists the fields of every type. To generate a typed client, download the schema in Schema Definition Language (SDL) from `/schema`, which any authenticated user can read:

```bash
curl -s "$OPENCTI_URL/schema" -H "Authorization: Bearer $OPENCTI_TOKEN" | jq -r .schema > opencti.graphql
```

## Page through long results

Pass the `endCursor` of a page as `after` to get the next one, until `hasNextPage` is `false`. `globalCount` gives the total number of matches from the first page. This loop prints every report on Velvet Harbor, two per page:

```bash
QUERY='query ReportsOnVelvetHarbor($filters: FilterGroup, $first: Int, $after: ID) {
  reports(filters: $filters, first: $first, after: $after, orderBy: published, orderMode: desc) {
    pageInfo { hasNextPage endCursor globalCount }
    edges { node { name published } }
  }
}'
VELVET_HARBOR_ID=cd92e87a-28a6-4817-81a1-b3305df36c8c
AFTER=null
while : ; do
  BODY=$(jq -n --arg q "$QUERY" --arg id "$VELVET_HARBOR_ID" --argjson after "$AFTER" \
    '{query: $q, variables: {first: 2, after: $after,
      filters: {mode: "and", filters: [{key: "objects", values: [$id]}], filterGroups: []}}}')
  PAGE=$(curl -s "$OPENCTI_URL/graphql" \
    -H "Authorization: Bearer $OPENCTI_TOKEN" -H "Content-Type: application/json" -d "$BODY")
  echo "$PAGE" | jq -r '.data.reports.edges[].node | "\(.published[:10])  \(.name)"'
  [ "$(echo "$PAGE" | jq '.data.reports.pageInfo.hasNextPage')" = "true" ] || break
  AFTER=$(echo "$PAGE" | jq '.data.reports.pageInfo.endCursor')
done
```

```text
2026-01-15  Velvet Harbor reuses guest Wi-Fi portal infrastructure
2026-01-14  Velvet Harbor phishing kit mimics hotel booking confirmations
2026-01-12  Velvet Harbor targets hotel booking systems
2026-01-09  Night-time exports from a hotel booking platform
2025-12-23  Velvet Harbor infrastructure moves to new hosting
2025-12-16  Look-alike booking domains registered in bulk
2025-12-02  Edge gateways exploited before the holiday season
2025-11-25  HarborGate backdoor on booking servers
2025-10-28  Velvet Loader delivered through booking attachments
2025-10-21  Fake booking requests reach hotel front desks
```

Keep the same `orderBy` and `filters` on every page: the cursor holds the sort values of the last item, and only means something for the same query. To follow changes over time rather than read a full list, listen to a [live stream](/develop/apis/streams/) instead of polling.

## Create an indicator

Mutations take an `input` object. Relationships to other objects, such as the author, the markings and the labels, take the IDs of those objects.

:::example[Record a Velvet Harbor phishing domain]
The CTI team at Example CERT finds `guest-wifi-velvet.example`, a look-alike guest Wi-Fi portal that Velvet Harbor uses against hotels. A script records it as an indicator, authored by Example CERT, marked TLP:GREEN and labelled `hotel-sector`, so that the SIEM live stream of the SOC picks it up.

First, the script reads the IDs it needs:

```bash
curl -s "$OPENCTI_URL/graphql" \
  -H "Authorization: Bearer $OPENCTI_TOKEN" -H "Content-Type: application/json" \
  -d '{"query": "{ labels(search: \"hotel-sector\") { edges { node { id value } } } identities(search: \"Example CERT\", first: 1) { edges { node { id name } } } markingDefinitions(search: \"TLP:GREEN\") { edges { node { id definition } } } }"}'
```

```json
{"data":{"labels":{"edges":[{"node":{"id":"8a316f32-14e5-49ab-a685-62f7718a1a2d","value":"hotel-sector"}}]},"identities":{"edges":[{"node":{"id":"13a8c72c-08b2-4262-bfac-82ecd17bad4c","name":"Example CERT"}}]},"markingDefinitions":{"edges":[{"node":{"id":"50e381ba-e609-449b-a2ed-c69881cd28d2","definition":"TLP:GREEN"}}]}}}
```

Then it sends the mutation, with the IDs it read:

```bash
cat > create-indicator.json <<'EOF'
{
  "query": "mutation CreateIndicator($input: IndicatorAddInput!) { indicatorAdd(input: $input) { id standard_id name pattern x_opencti_score valid_from valid_until objectLabel { value } createdBy { name } objectMarking { definition } } }",
  "variables": {
    "input": {
      "name": "guest-wifi-velvet.example",
      "description": "Look-alike guest Wi-Fi portal used by Velvet Harbor.",
      "pattern": "[domain-name:value = 'guest-wifi-velvet.example']",
      "pattern_type": "stix",
      "x_opencti_main_observable_type": "Domain-Name",
      "x_opencti_score": 75,
      "createdBy": "13a8c72c-08b2-4262-bfac-82ecd17bad4c",
      "objectMarking": ["50e381ba-e609-449b-a2ed-c69881cd28d2"],
      "objectLabel": ["8a316f32-14e5-49ab-a685-62f7718a1a2d"]
    }
  }
}
EOF
curl -s "$OPENCTI_URL/graphql" \
  -H "Authorization: Bearer $OPENCTI_TOKEN" -H "Content-Type: application/json" \
  -d @create-indicator.json | jq
```

```json
{
  "data": {
    "indicatorAdd": {
      "id": "3ed349ba-f227-4fb2-8203-e9683b328a9d",
      "standard_id": "indicator--a6b80844-c4b6-5dc7-aee8-3a0af2933801",
      "name": "guest-wifi-velvet.example",
      "pattern": "[domain-name:value = 'guest-wifi-velvet.example']",
      "x_opencti_score": 75,
      "valid_from": "2026-11-01T22:45:58.340Z",
      "valid_until": "2027-04-07T08:31:02.884Z",
      "objectLabel": [{"value": "hotel-sector"}],
      "createdBy": {"name": "Example CERT"},
      "objectMarking": [{"definition": "TLP:GREEN"}]
    }
  }
}
```

The script did not send `valid_from` nor `valid_until`: OpenCTI set them, the end of validity from the decay rule that applies to the indicator. The indicator matches the filters of the SIEM stream, so its consumers receive it within seconds.
:::

Creation is idempotent on the STIX ID. The STIX ID of an indicator is computed from its pattern: sending the same pattern again updates the existing indicator, with the same `id`, instead of creating a second one. The values the second call sends, such as a new `x_opencti_score`, replace the stored ones as far as the confidence level of the user allows.

To change fields of an existing object, send the field name and its new values:

```bash
curl -s "$OPENCTI_URL/graphql" \
  -H "Authorization: Bearer $OPENCTI_TOKEN" -H "Content-Type: application/json" \
  -d '{"query": "mutation UpdateIndicator($id: ID!, $input: [EditInput!]!) { indicatorFieldPatch(id: $id, input: $input) { id x_opencti_score } }", "variables": {"id": "3ed349ba-f227-4fb2-8203-e9683b328a9d", "input": [{"key": "x_opencti_score", "value": ["65"]}]}}'
```

```json
{"data":{"indicatorFieldPatch":{"id":"3ed349ba-f227-4fb2-8203-e9683b328a9d","x_opencti_score":65}}}
```

`value` is always a list, even for a single value. `operation` is `replace` when you leave it out; `add` and `remove` change multiple-value fields. `indicatorDelete(id:)` deletes the indicator: with the trash turned on, the default, it goes to the trash, from which an administrator can restore it.

Some request headers change how a mutation runs:

| Header | Effect |
| --- | --- |
| `opencti-draft-id` | Runs the request inside this draft workspace instead of the live knowledge base. See [Prepare changes in a draft](/opencti/use/ingest/drafts/) |
| `opencti-applicant-id` | Recorded in the origin of the change, as the user the request acts for. Connectors and playbooks set it |

## Read the errors

The API answers HTTP `200` even when a request fails. Read the `errors` array of the body: `message` says what happened, `extensions.code` gives the error type, and `extensions.data` the details, with the HTTP status the error stands for in `http_status`. When the failure is in one field, `data` holds `null` for that field and the rest of the answer is still there.

Without a valid token:

```json
{"errors":[{"message":"You must be logged in to do this.","locations":[{"line":1,"column":3}],"path":["reports"],"extensions":{"code":"AUTH_REQUIRED","data":{"http_status":401,"genre":"TECHNICAL"}},"name":"AUTH_REQUIRED"}],"data":{"reports":null}}
```

With a filter key that does not exist, `object-label` instead of `objectLabel`:

```json
{"errors":[{"message":"Incorrect filter keys not existing in any schema definition","locations":[{"line":1,"column":3}],"path":["reports"],"extensions":{"code":"BAD_USER_INPUT_ERROR","data":{"http_status":400,"genre":"BUSINESS","keys":["object-label"]}},"name":"BAD_USER_INPUT_ERROR"}],"data":{"reports":null}}
```

The errors you meet first:

| `extensions.code` | Cause | What to do |
| --- | --- | --- |
| `AUTH_REQUIRED` | No token, a wrong or expired token, or a user without **Allow token usage** | Check the header, the expiry of the token, and the role of its user |
| `FORBIDDEN_ACCESS` | The user lacks the capability of the operation, or cannot access the object | Give the user a role with the capability, or check the markings of the object |
| `GRAPHQL_VALIDATION_FAILED` | The query does not match the schema, such as `Cannot query field "title" on type "Report"`, or calls one query too many times through aliases | Check the field names in the schema reference. A request can call the same query twice and the same mutation once (`app:graphql:batching_protection:query_default`, `app:graphql:batching_protection:mutation_default`) |
| `BAD_USER_INPUT_ERROR` | An argument is wrong, such as an unknown filter key, listed in `extensions.data.keys` | Fix the key, using the [filter keys reference](/reference/opencti/filter-keys/) |
| `VALIDATION_ERROR` | A value breaks a rule, such as `The score should be an integer between 0 and 100`; `extensions.data.field` names the field | Fix the value |
| `RESOURCE_NOT_FOUND` | An introspection query (`__schema`) while introspection is turned off | Read the schema from `/schema` or from the schema reference |

The platform also limits each client, by address and user agent, to 10,000 requests a second by default (`app:rate_protection:max_requests`, `app:rate_protection:time_window`). Beyond it, the answer is HTTP `429` with `{"message":"Too many requests, please try again later."}`. The [errors reference](/reference/opencti/errors/#common-errors-and-what-to-do) lists every message and what to do.

:::only{opencti=">=7.261119"}
A platform with the [secure HTTP defaults](/deploy/operate/harden/#secure-http-defaults) on allows 30,000 requests per 60 seconds instead, unless its platform engineer sets these two keys. It also refuses a request body over 10 MB unless `app:max_payload_body_size` is set: split a large STIX bundle sent to `stixBundlePush` into smaller ones.
:::

## Use the playground

The playground is a GraphiQL editor at `/public/graphql`, also opened by **Playground** in the **API access** card of your profile. It sends requests with the session of your browser, so it runs with your own rights; to try another user's token, add an `Authorization` header in its **Headers** panel. It opens with two example operations, one of them with a filter in its variables.

Two settings of the platform decide what the playground can do. A platform engineer sets them in the configuration, as [Configuration](/reference/opencti/configuration/#technical-customization) describes:

| Key | Environment variable | Default | Effect |
| --- | --- | --- | --- |
| `app:graphql:playground:enabled` | `APP__GRAPHQL__PLAYGROUND__ENABLED` | `true` | Serves the playground and shows its button in the profile |
| `app:graphql:playground:force_disabled_introspection` | `APP__GRAPHQL__PLAYGROUND__FORCE_DISABLED_INTROSPECTION` | `true` | Refuses introspection queries. The documentation explorer and the autocompletion of the playground need introspection, so they stay empty while it is `true` |

:::only{opencti=">=7.261109"}
When `app:graphql:playground:enabled` is not set, it follows `app:graphql:secure_by_default` (`APP__GRAPHQL__SECURE_BY_DEFAULT`): a platform that turns the secure GraphQL defaults on serves no playground unless the platform engineer sets `APP__GRAPHQL__PLAYGROUND__ENABLED=true`. The same setting makes the WebSocket endpoint accept subscriptions only: send your queries and mutations over HTTP. See [Secure GraphQL defaults](/deploy/operate/harden/#secure-graphql-defaults).
:::

To browse the schema in the playground on a development platform, turn introspection on for authenticated users:

```bash
APP__GRAPHQL__PLAYGROUND__FORCE_DISABLED_INTROSPECTION=false
```

On a production platform, leave it refused and read the schema from `/schema` or from the [GraphQL schema reference](/reference/opencti/graphql-schema/). [Harden the deployment](/deploy/operate/harden/#graphql-playground) discusses turning the playground off.

## What's next

- [Filters](/develop/apis/filters/): the filter group syntax every list query, feed and stream uses.
- [Data streaming](/develop/apis/streams/): receive changes as they happen instead of polling.
- [Python client](/develop/build/python-client/): the same API from Python, with helpers for each entity type.
- [GraphQL schema](/reference/opencti/graphql-schema/) and [Errors](/reference/opencti/errors/): every operation, field, capability and error message of this version.
