---
title: Script OpenCTI in Python
description: Script the OpenCTI API in Python with pycti - connect, read with filters, create objects with markings and relationships, import a STIX bundle, and run your own GraphQL queries.
doc_type: how-to
products:
  - opencti
personas: [developer]
owner: docs-develop
last_reviewed: 2026-11-01
no_screenshot: The tasks run in Python against the API; the results show on the usual entity screens, linked from each step.
migrated_from:
  - opencti:/development/python/
---

`pycti` is the official Python library of OpenCTI. Its `OpenCTIApiClient` wraps the GraphQL API: one attribute per entity type (`client.report`, `client.indicator`, `client.intrusion_set`...) with `list`, `read` and `create` methods, plus raw queries and STIX bundle import. This page does the common operations with examples that ran against OpenCTI 7.261030.1. To build a connector, which sends bundles to the workers instead of calling the API for each object, follow [Build an OpenCTI connector](/develop/build/connector/).

::diagram{id=develop-build-python-client}

## Before you start

- Install the `pycti` version of your platform: `pip install pycti==7.261030.1`. The library and the platform are released together, with the same version number.
- Put the platform URL and an API token in `OPENCTI_URL` and `OPENCTI_TOKEN`. Every call runs as the token's user: their capabilities, markings and confidence level apply, as in the interface.

## Connect

`OpenCTIApiClient` checks the URL and the token when you create it, and raises an error when the platform is unreachable or refuses the token.

```python
import os

from pycti import OpenCTIApiClient

# Checks the URL and the token at once: raises an error if either is wrong
client = OpenCTIApiClient(
    url=os.environ["OPENCTI_URL"],
    token=os.environ["OPENCTI_TOKEN"],
    log_level="error",
)
me = client.query("query { me { name user_email } about { version } }")["data"]
print(f"Connected to OpenCTI {me['about']['version']} as {me['me']['name']}")
```

```text
Connected to OpenCTI 7.261030.1 as admin
```

The constructor also takes `ssl_verify` (`False` by default: pass `True`, or the path of a CA bundle, for an HTTPS platform), `proxies`, `custom_headers` and `requests_timeout`, in seconds (300 by default). `log_level="error"` keeps the client quiet; its default, `info`, logs each call.

## Read with filters

`list()` takes a filter group: a `mode` (`and` or `or`), a list of `filters`, each with a `key`, `values` and an `operator` (`eq` by default), and nested `filterGroups`. The keys are the ones of the interface: the [filter keys reference](/reference/opencti/filter-keys/) lists them per entity type. `read()` returns the first match, or `None`.

This example finds the intrusion set Velvet Harbor, then the reports that contain it, published since January 1, 2026:

```python
import os

from pycti import OpenCTIApiClient

client = OpenCTIApiClient(os.environ["OPENCTI_URL"], os.environ["OPENCTI_TOKEN"], log_level="error")

velvet_harbor = client.intrusion_set.read(
    filters={
        "mode": "and",
        "filters": [{"key": "name", "values": ["Velvet Harbor"]}],
        "filterGroups": [],
    }
)
print("Intrusion set:", velvet_harbor["name"], velvet_harbor["standard_id"])

# The reports that contain Velvet Harbor, published since January 1, 2026
reports = client.report.list(
    filters={
        "mode": "and",
        "filters": [
            {"key": "objects", "values": [velvet_harbor["id"]]},
            {"key": "published", "values": ["2026-01-01T00:00:00Z"], "operator": "gte"},
        ],
        "filterGroups": [],
    },
    orderBy="published",
    orderMode="desc",
    getAll=True,
)
for report in reports:
    author = (report["createdBy"] or {}).get("name", "no author")
    print(f"{report['published'][:10]}  {report['name']}  ({author})")
```

```text
Intrusion set: Velvet Harbor intrusion-set--4e78bb64-59c9-51e3-b1b9-ac8133e1c8e0
2026-01-15  Velvet Harbor reuses guest Wi-Fi portal infrastructure  (Example CERT)
2026-01-14  Velvet Harbor phishing kit mimics hotel booking confirmations  (Example CERT)
2026-01-12  Velvet Harbor targets hotel booking systems  (Example CERT)
2026-01-09  Night-time exports from a hotel booking platform  (Grand Meridian Hotels)
```

`list()` returns 100 objects by default (`first`). `getAll=True` follows the pages for you; `withPagination=True` returns the page information too, for your own paging with `after`.

## Create objects

Each `create()` returns the object with its `id`, the internal identifier, and its `standard_id`, the STIX identifier. OpenCTI computes the `standard_id` from the identifying properties of the object, such as the pattern of an indicator or the name of an intrusion set: create the same object twice and the second call updates the first, with the same identifiers.

This example creates an indicator authored by Example CERT, marked TLP:GREEN, and the relationship that says it indicates Velvet Harbor:

```python
import os

from pycti import OpenCTIApiClient
from stix2 import TLP_GREEN

client = OpenCTIApiClient(os.environ["OPENCTI_URL"], os.environ["OPENCTI_TOKEN"], log_level="error")

author = client.identity.create(type="Organization", name="Example CERT")
tlp_green = client.marking_definition.read(id=TLP_GREEN["id"])
velvet_harbor = client.intrusion_set.read(
    filters={"mode": "and", "filters": [{"key": "name", "values": ["Velvet Harbor"]}], "filterGroups": []}
)

indicator = client.indicator.create(
    name="checkin-velvet.example.net",
    description="Look-alike check-in page sent to hotel guests.",
    pattern_type="stix",
    pattern="[domain-name:value = 'checkin-velvet.example.net']",
    x_opencti_main_observable_type="Domain-Name",
    valid_from="2026-01-15T10:00:00Z",
    confidence=70,
    createdBy=author["id"],
    objectMarking=[tlp_green["id"]],
)
relationship = client.stix_core_relationship.create(
    fromId=indicator["id"],
    toId=velvet_harbor["id"],
    relationship_type="indicates",
    createdBy=author["id"],
    objectMarking=[tlp_green["id"]],
)
print("Indicator:", indicator["standard_id"])
print("Relationship:", relationship["standard_id"])
```

```text
Indicator: indicator--5bc47565-1eed-5022-9d7d-2892cceae172
Relationship: relationship--fb4d3042-a0dd-5079-9526-4d3ab0ed96e5
```

`x_opencti_main_observable_type` is required for an indicator: it is the observable type the pattern detects. The indicator then shows in :menu[Observations > Indicators], with Velvet Harbor in its relationships. The `confidence` you pass is capped at the user's maximum confidence level.

## Import a STIX bundle

`client.stix2.import_bundle_from_json()` writes a whole STIX 2.1 bundle through the API, object by object, referenced objects first. It returns the imported objects, and the objects left out because they have more references than its `objects_max_refs` argument allows (no limit by default). It runs in your process, as the token's user: use it for a few hundred objects. For more, or on a schedule, a connector sends the bundle to the workers instead.

This example adds a note to the report "Velvet Harbor targets hotel booking systems":

```python
import os

import stix2
from pycti import Identity, Note, OpenCTIApiClient

client = OpenCTIApiClient(os.environ["OPENCTI_URL"], os.environ["OPENCTI_TOKEN"], log_level="error")

author = stix2.Identity(
    id=Identity.generate_id("Example CERT", "organization"),
    name="Example CERT",
    identity_class="organization",
)
report = client.report.read(
    filters={
        "mode": "and",
        "filters": [{"key": "name", "values": ["Velvet Harbor targets hotel booking systems"]}],
        "filterGroups": [],
    }
)
note_text = "Three member hotels confirmed the phishing pages on January 15, 2026."
note = stix2.Note(
    id=Note.generate_id(created="2026-01-15T12:00:00Z", content=note_text),
    created="2026-01-15T12:00:00Z",
    content=note_text,
    object_refs=[report["standard_id"]],
    created_by_ref=author.id,
    object_marking_refs=[stix2.TLP_GREEN.id],
)
bundle = stix2.Bundle(objects=[author, stix2.TLP_GREEN, note], allow_custom=True)

# Written at once by the platform, as the token's user: no queue, no worker
imported, too_many_refs = client.stix2.import_bundle_from_json(bundle.serialize())
for obj in imported:
    print("Imported", obj["type"], obj["id"])
```

```text
Imported identity identity--ac6f7b3a-7aa5-558d-a1bd-4a4f52840f9e
Imported marking-definition marking-definition--34098fce-860f-48ae-8e50-ebd3cc5e41da
Imported note note--2d4b50aa-e314-56b9-b221-92e4ceadcf41
```

The `generate_id` functions of `pycti` (`Note.generate_id`, `Identity.generate_id`, `Indicator.generate_id`...) compute the identifier OpenCTI would give the object, so a bundle sent twice merges instead of duplicating.

## Run your own GraphQL query

`client.query(query, variables)` sends any GraphQL query or mutation and returns the JSON response. Use it for a field or an operation the entity classes do not cover. The [GraphQL schema reference](/reference/opencti/graphql-schema/) lists the types and fields.

This example pages through the indicators whose name contains "velvet", two at a time, with the cursor of each page:

```python
import os

from pycti import OpenCTIApiClient

client = OpenCTIApiClient(os.environ["OPENCTI_URL"], os.environ["OPENCTI_TOKEN"], log_level="error")

query = """
query Indicators($filters: FilterGroup, $first: Int, $after: ID) {
  indicators(filters: $filters, first: $first, after: $after, orderBy: name, orderMode: asc) {
    pageInfo { globalCount hasNextPage endCursor }
    edges { node { name x_opencti_score } }
  }
}
"""
# The indicators whose name contains "velvet", two per page
filters = {
    "mode": "and",
    "filters": [{"key": "name", "values": ["velvet"], "operator": "contains"}],
    "filterGroups": [],
}
after = None
while True:
    page = client.query(query, {"filters": filters, "first": 2, "after": after})
    indicators = page["data"]["indicators"]
    for edge in indicators["edges"]:
        print(edge["node"]["name"], edge["node"]["x_opencti_score"])
    if not indicators["pageInfo"]["hasNextPage"]:
        break
    after = indicators["pageInfo"]["endCursor"]
print("Total:", indicators["pageInfo"]["globalCount"])
```

```text
booking-confirm-velvet.example.net 20
booking-velvetharbor.example 90
checkin-velvet.example 85
checkin-velvet.example.net 20
https://booking-velvetharbor.example/login 100
https://wifi-portal-velvet.example.net/login 50
reservations-velvet.example 70
Total: 7
```

## Check that it works

Each example prints what the platform returned. In the interface, the objects appear where analysts look for them: the reports in :menu[Analyses > Reports], the indicator in :menu[Observations > Indicators], the note under **Notes about this entity** on its report. Their history shows the token's user as creator.

:::example[Add an indicator to the Velvet Harbor case]
An Example CERT analyst receives a look-alike check-in domain from a member hotel and records it from a script, as in [Create objects](#create-objects): `name` and `pattern` with `checkin-velvet.example.net`, `x_opencti_main_observable_type` `Domain-Name`, `confidence` 70, author `Example CERT`, marking TLP:GREEN, and an `indicates` relationship to the intrusion set Velvet Harbor. The script reads its connection from the environment, where the analyst's token is already in `OPENCTI_TOKEN`:

```bash
export OPENCTI_URL=https://cti.example.com
python create_indicator.py
```

The indicator appears in :menu[Observations > Indicators] and in the knowledge of Velvet Harbor. Running the script again changes nothing: the indicator and the relationship keep their identifiers.
:::

## What's next

- [Build an OpenCTI connector](/develop/build/connector/), to import a feed on a schedule.
- [Connectors](/develop/integration-model/connectors/), the types of connectors and the users they write as.
- [GraphQL schema reference](/reference/opencti/graphql-schema/), every type and field of the API.
- More examples are in the [client-python/examples](https://github.com/OpenCTI-Platform/opencti/tree/master/client-python/examples) folder of the OpenCTI repository.
