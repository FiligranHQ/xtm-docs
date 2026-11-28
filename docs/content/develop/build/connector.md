---
title: Build an OpenCTI connector
description: Write an external import connector with pycti, from an empty folder to a run that creates reports, intrusion sets and indicators in OpenCTI, then package it with Docker.
doc_type: tutorial
products:
  - opencti
personas: [developer, platform-engineer]
owner: docs-develop
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/development/connectors/
---

In this tutorial, you write the connector that imports the hospitality threat feed of Example CERT into OpenCTI: each feed item becomes a report, with the intrusion set it names, its indicators, and the relationships between them. It takes about 45 minutes. At the end, you know how a connector registers, builds STIX 2.1 objects that never duplicate, reports its runs as works, saves its state, and runs on a schedule in Docker. The connector is an `EXTERNAL_IMPORT` connector: read [Connectors](/develop/integration-model/connectors/) for the other types.

::diagram{id=develop-build-connector}

## What you need

- An OpenCTI 7.261127 platform, such as the [XTM stack](/start/quick-start/) at `http://localhost:8080`.
- The API token of the user the connector will write as, in the `OPENCTI_TOKEN` variable of your shell. For a test, an administrator's token works; in production, give the connector its own user in the `Connectors` group ([Run an integration without XTM Composer](/deploy/integrations/manual/#create-a-token)).
- Python 3.10 or later, and Docker for the last step.

The connector uses `pycti`, the Python library of OpenCTI, which holds the connector helper (`OpenCTIConnectorHelper`) and the API client. Use the `pycti` version of your platform: both are released together.

## Step 1: Lay out the project

Create this tree. The connector code lives in `src/`; the catalog metadata in `__metadata__/` comes at the end.

```text
hospitality-feed/
├── __metadata__/
│   ├── connector_manifest.json
│   └── connector_config_schema.json
├── sample-feed/
│   └── feed.json
├── src/
│   ├── client.py
│   ├── converter.py
│   ├── main.py
│   ├── config.yml.sample
│   └── requirements.txt
├── Dockerfile
└── docker-compose.yml
```

Pin the libraries in `src/requirements.txt`, then install them in a virtual environment:

`src/requirements.txt`:

```text
pycti==7.261127.0
requests~=2.33
```

```bash
cd hospitality-feed
python3 -m venv .venv
. .venv/bin/activate
pip install -r src/requirements.txt
```

## Step 2: Read the feed

The feed returns JSON items. Each item names an intrusion set and lists indicators. Save this sample as `sample-feed/feed.json`, to test without the real feed:

`sample-feed/feed.json`:

```json
{
  "generated": "2026-01-15T10:00:00Z",
  "items": [
    {
      "id": "HTF-2026-0007",
      "title": "Velvet Harbor phishing kit mimics hotel booking confirmations",
      "published": "2026-01-14T08:00:00Z",
      "summary": "Phishing emails sent to front-desk staff link to look-alike booking pages that collect property-management credentials.",
      "actor": "Velvet Harbor",
      "confidence": 70,
      "indicators": [
        {"type": "domain", "value": "booking-confirm-velvet.example.net"},
        {"type": "ipv4", "value": "203.0.113.24"}
      ]
    },
    {
      "id": "HTF-2026-0008",
      "title": "Velvet Harbor reuses guest Wi-Fi portal infrastructure",
      "published": "2026-01-15T07:30:00Z",
      "summary": "A guest Wi-Fi captive portal clone hosted on a new server redirects guests to a credential harvesting page.",
      "actor": "Velvet Harbor",
      "confidence": 60,
      "indicators": [
        {"type": "url", "value": "https://wifi-portal-velvet.example.net/login"},
        {"type": "ipv4", "value": "198.51.100.77"}
      ]
    }
  ]
}
```

`client.py` fetches the items and keeps the ones published after a date, oldest first. It knows nothing about OpenCTI:

`src/client.py`:

```python
"""Reads the hospitality threat feed."""

import requests


class FeedClient:
    def __init__(self, url: str, api_key: str):
        self.url = url
        self.session = requests.Session()
        self.session.headers["Authorization"] = f"Bearer {api_key}"

    def items_since(self, since: str | None) -> list[dict]:
        """Return the feed items published after `since` (ISO 8601), oldest first."""
        response = self.session.get(self.url, timeout=60)
        response.raise_for_status()
        items = response.json()["items"]
        if since:
            items = [item for item in items if item["published"] > since]
        return sorted(items, key=lambda item: item["published"])
```

## Step 3: Turn items into STIX 2.1

`converter.py` builds the STIX objects with the `stix2` library, which `pycti` installs. Three rules make a connector a good citizen:

- **Deterministic identifiers.** Each object gets the identifier OpenCTI would compute itself, from `pycti`'s `generate_id` functions: `IntrusionSet.generate_id(name)`, `Indicator.generate_id(pattern)`, `Report.generate_id(name, published)`. Sent twice, the same object updates the existing one instead of creating a duplicate, and the `Hospitality` sector merges with the one the platform already has.
- **An author and a marking on every object.** `created_by_ref` points to the `Example CERT` organization, and `object_marking_refs` to TLP:GREEN. Analysts can then filter by source, and markings restrict who sees the data.
- **A confidence on every object.** The feed rates each item; when the worker writes it, the platform caps it at the maximum confidence level of the connector's user.

Two OpenCTI properties extend the STIX indicator: `x_opencti_main_observable_type`, the observable type the pattern detects, and `x_opencti_create_observables`, which makes the worker also create the observable.

`src/converter.py`:

```python
"""Turns feed items into STIX 2.1 objects that OpenCTI understands."""

import stix2
from pycti import (
    Identity,
    Indicator,
    IntrusionSet,
    Report,
    StixCoreRelationship,
)

# How each indicator type of the feed becomes a STIX pattern and an OpenCTI observable type
PATTERNS = {
    "domain": ("[domain-name:value = '{}']", "Domain-Name"),
    "ipv4": ("[ipv4-addr:value = '{}']", "IPv4-Addr"),
    "url": ("[url:value = '{}']", "Url"),
}


class Converter:
    def __init__(self, author_name: str):
        # Every object carries its author and a TLP marking
        self.author = stix2.Identity(
            id=Identity.generate_id(author_name, "organization"),
            name=author_name,
            identity_class="organization",
        )
        self.marking = stix2.TLP_GREEN
        self.sector = stix2.Identity(
            id=Identity.generate_id("Hospitality", "class"),
            name="Hospitality",
            identity_class="class",
            created_by_ref=self.author.id,
            object_marking_refs=[self.marking.id],
        )

    def _common(self, confidence: int) -> dict:
        return {
            "created_by_ref": self.author.id,
            "object_marking_refs": [self.marking.id],
            "confidence": confidence,
        }

    def to_stix(self, item: dict) -> list:
        common = self._common(item["confidence"])
        intrusion_set = stix2.IntrusionSet(
            id=IntrusionSet.generate_id(item["actor"]),
            name=item["actor"],
            **common,
        )
        objects = [intrusion_set]
        targets = stix2.Relationship(
            id=StixCoreRelationship.generate_id(
                "targets", intrusion_set.id, self.sector.id
            ),
            relationship_type="targets",
            source_ref=intrusion_set.id,
            target_ref=self.sector.id,
            **common,
        )
        objects.append(targets)
        for entry in item["indicators"]:
            template, observable_type = PATTERNS[entry["type"]]
            pattern = template.format(entry["value"])
            indicator = stix2.Indicator(
                id=Indicator.generate_id(pattern),
                name=entry["value"],
                pattern=pattern,
                pattern_type="stix",
                valid_from=item["published"],
                custom_properties={
                    "x_opencti_main_observable_type": observable_type,
                    "x_opencti_create_observables": True,
                },
                **common,
            )
            indicates = stix2.Relationship(
                id=StixCoreRelationship.generate_id(
                    "indicates", indicator.id, intrusion_set.id
                ),
                relationship_type="indicates",
                source_ref=indicator.id,
                target_ref=intrusion_set.id,
                **common,
            )
            objects += [indicator, indicates]
        report = stix2.Report(
            id=Report.generate_id(item["title"], item["published"]),
            name=item["title"],
            description=item["summary"],
            published=item["published"],
            report_types=["threat-report"],
            object_refs=[obj.id for obj in objects] + [self.sector.id],
            external_references=[
                {"source_name": "Hospitality threat feed", "external_id": item["id"]}
            ],
            **common,
        )
        return objects + [report]

    def bundle(self, items: list[dict]) -> str:
        objects = [self.author, self.marking, self.sector]
        for item in items:
            objects += self.to_stix(item)
        return stix2.Bundle(objects=objects, allow_custom=True).serialize()
```

## Step 4: Write the connector

`main.py` ties the pieces together with the connector helper:

- `OpenCTIConnectorHelper(config)` reads every `OPENCTI_*` and `CONNECTOR_*` setting, environment variable first, then `config.yml`, then default. It registers the connector and starts its heartbeat.
- `get_config_variable()` reads the connector's own settings the same way.
- `helper.get_state()` and `helper.set_state()` read and save a dictionary on the platform. The connector keeps the date of the last item it imported, so a restart does not import everything again.
- `helper.api.work.initiate_work()` opens a work, the record of one run that the **Works** tab of the connector shows; `send_stix2_bundle(bundle, work_id=...)` attaches the bundle to it; `to_processed()` closes it with a message.
- `helper.schedule_iso()` runs `process()` at once, then every `CONNECTOR_DURATION_PERIOD`, an ISO 8601 duration. Before a run, it checks the connector's queue: when more than `CONNECTOR_QUEUE_THRESHOLD` megabytes (500) wait for the workers, it skips the run. The first run of a connector that keeps running starts without this check.

`src/main.py`:

```python
"""Hospitality threat feed connector for OpenCTI."""

import os
import sys
from datetime import datetime, timezone

import yaml
from pycti import OpenCTIConnectorHelper, get_config_variable

from client import FeedClient
from converter import Converter


def load_config() -> dict:
    path = os.path.join(os.path.dirname(os.path.abspath(__file__)), "config.yml")
    if not os.path.isfile(path):
        return {}
    with open(path, encoding="utf-8") as file:
        return yaml.safe_load(file)


class HospitalityFeedConnector:
    def __init__(self):
        config = load_config()
        # Reads every OPENCTI_* and CONNECTOR_* setting and registers the connector
        self.helper = OpenCTIConnectorHelper(config)

        # Settings of this connector: environment variable, then config.yml, then default
        def setting(name, key, **kwargs):
            return get_config_variable(name, ["hospitality_feed", key], config, **kwargs)

        self.feed = FeedClient(
            url=setting("HOSPITALITY_FEED_URL", "url", required=True),
            api_key=setting("HOSPITALITY_FEED_API_KEY", "api_key", required=True),
        )
        self.converter = Converter(
            setting("HOSPITALITY_FEED_AUTHOR", "author", default="Example CERT")
        )
        self.duration_period = get_config_variable(
            "CONNECTOR_DURATION_PERIOD",
            ["connector", "duration_period"],
            config,
            default="PT1H",
        )

    def process(self):
        state = self.helper.get_state() or {}
        since = state.get("last_published")
        items = self.feed.items_since(since)
        if not items:
            self.helper.connector_logger.info("No new item", {"since": since})
            return
        # A work groups what one run sends: the Works tab of the connector shows it
        now = datetime.now(timezone.utc).strftime("%Y-%m-%d %H:%M:%S")
        work_id = self.helper.api.work.initiate_work(
            self.helper.connect_id, f"Hospitality threat feed run @ {now}"
        )
        try:
            bundle = self.converter.bundle(items)
            sent = self.helper.send_stix2_bundle(bundle, work_id=work_id)
            message = f"{len(items)} feed items sent in {len(sent)} bundles"
            self.helper.api.work.to_processed(work_id, message)
            self.helper.connector_logger.info(message)
            # Saved on the platform: the next run starts from here, even after a restart
            self.helper.set_state({"last_published": items[-1]["published"]})
        except Exception as error:
            self.helper.api.work.to_processed(work_id, str(error), in_error=True)
            raise

    def run(self):
        # Runs process() now, then every CONNECTOR_DURATION_PERIOD
        self.helper.schedule_iso(
            message_callback=self.process, duration_period=self.duration_period
        )


if __name__ == "__main__":
    try:
        HospitalityFeedConnector().run()
    except Exception as error:
        print(error, file=sys.stderr)
        sys.exit(1)
```

The connector never creates objects through the API. `send_stix2_bundle()` sends the bundle to the connector's queue: split into one bundle per object over RabbitMQ, or whole through the platform API, where the worker splits it. The workers write it in the background, in the right order, with retries: a connector can send thousands of objects without waiting.

:::only{opencti=">=7.261127"}
### Report a failed run

When `process()` raises, the helper catches the exception. It logs `Connector run raised an exception` with the traceback, sends the exception to the platform with its next ping, as an error of the run, and schedules the next run as usual; with `CONNECTOR_RUN_AND_TERMINATE=true`, it sends the error, then exits. In this connector, a feed that answers with an HTTP error makes `items_since()` raise before any work opens; an error while sending makes `process()` close the work in error, then raise.

An error has a `code`, a `message` and a `criticity`. For an exception, the message is the name of the exception and its text, and the code and the criticity come from its `code` and `criticity` attributes. Without them, the error gets the code `other.unknown` and the criticity `50`. With Python 3.11 or later, each exception of an `ExceptionGroup` gives its own error. To name the failures of your feed, raise an exception of your own from `client.py`:

```python
class FeedUnavailable(Exception):
    code = "hospitality_feed.unavailable"  # a non-empty string
    criticity = 40                         # an integer
```

A run can also report errors without raising: `process()` returns a list of dictionaries with `code`, `message` and `criticity`, or calls `self.helper.connector_info.add_error(code, message, criticity)`. The helper logs and drops an error that is not valid, such as a criticity that is not an integer.

An error of criticity 40 or more that `process()` returns or raises fails the run: the **Last run** date of the connector stays at the last run without such an error. An error added with `add_error()` is reported, but does not change the **Last run** date. The errors of a run replace those of the run before. The platform keeps the 20 most critical, one per code, and drops them 10 minutes after the last ping that sent them. The connector's page does not show them: read them through the API, with the **Access connectors** capability:

```graphql
query {
  connector(id: "0e5a7f8c-2b4d-4c1e-9f3a-6d8b1c2e4f70") {
    name
    connector_info {
      last_run_datetime
      errors { code message criticity }
    }
  }
}
```
:::

## Step 5: Run it once

Copy `src/config.yml.sample` to `src/config.yml`. It holds the settings that are not secret; the token and the feed key come from the environment.

`src/config.yml.sample`:

```yaml
# Secrets (OPENCTI_TOKEN, HOSPITALITY_FEED_API_KEY) come from the environment
opencti:
  url: 'http://localhost:8080'

connector:
  id: '0e5a7f8c-2b4d-4c1e-9f3a-6d8b1c2e4f70'
  type: 'EXTERNAL_IMPORT'
  name: 'Hospitality threat feed'
  scope: 'report,indicator,intrusion-set'
  log_level: 'info'
  duration_period: 'PT1H'

hospitality_feed:
  url: 'https://feeds.example.com/hospitality/v1/items.json'
  author: 'Example CERT'
```

`CONNECTOR_ID` must be a UUIDv4 of its own: generate one with `python3 -c 'import uuid; print(uuid.uuid4())'` and keep it. The platform stores the connector's state and works under it.

:::steps
1. Serve the sample feed in a second terminal:

   ```bash
   cd hospitality-feed/sample-feed
   python3 -m http.server 8791 --bind 127.0.0.1
   ```

2. Run the connector once, with the token of your user already in `OPENCTI_TOKEN`:

   ```bash
   cd hospitality-feed/src
   export HOSPITALITY_FEED_URL=http://127.0.0.1:8791/feed.json
   export HOSPITALITY_FEED_API_KEY=local-test-key
   export CONNECTOR_RUN_AND_TERMINATE=true
   export CONNECTOR_QUEUE_PROTOCOL=api
   export OPENCTI_JSON_LOGGING=false
   python main.py
   ```

   `CONNECTOR_RUN_AND_TERMINATE=true` runs `process()` once and exits. `CONNECTOR_QUEUE_PROTOCOL=api` sends the bundles through the platform API, because RabbitMQ is usually not reachable from your workstation. `OPENCTI_JSON_LOGGING=false` prints plain logs instead of JSON lines.
:::

:::verify[The connector registers, sends, and exits]
```text
INFO:Hospitality threat feed:Connector registered with ID
INFO:Hospitality threat feed:Starting run and terminate
INFO:api:Initiate work
INFO:api:Pushing a bundle to queue through API
INFO:api:Reporting work update_processed
INFO:Hospitality threat feed:2 feed items sent in 15 bundles
INFO:Hospitality threat feed:Closing run and terminate
```
:::

The helper splits the bundle into one bundle per object, in the order the workers need: 15 distinct objects here, because the intrusion set and its `targets` relationship appear in both items.

## Step 6: Check what OpenCTI created

In OpenCTI, open :menu[Integrations] and the **Hospitality threat feed** connector in the **Deployed** tab. Its **Works** tab shows the run, its 15 operations, and any object the platform refused.

::screenshot{id=opencti-ingest-connectors-works alt="The Works tab of the Hospitality threat feed connector lists a completed work with its operations completed out of the total and a button counting its errors."}

The reports are in :menu[Analyses > Reports]. To check from code, list the reports published after January 13, 2026. The script also reads the URL of the platform from the environment: run `export OPENCTI_URL=http://localhost:8080` first.

`verify.py`:

```python
import os
from pycti import OpenCTIApiClient

client = OpenCTIApiClient(os.environ["OPENCTI_URL"], os.environ["OPENCTI_TOKEN"], log_level="error")
reports = client.report.list(
    filters={
        "mode": "and",
        "filters": [{"key": "published", "values": ["2026-01-13T00:00:00Z"], "operator": "gt"}],
        "filterGroups": [],
    },
    orderBy="published",
    orderMode="asc",
)
for report in reports:
    print(
        report["published"][:10],
        report["name"],
        "| author:", report["createdBy"]["name"],
        "| confidence:", report["confidence"],
        "| objects:", len(report["objectsIds"]),
    )
```

```text
2026-01-14 Velvet Harbor phishing kit mimics hotel booking confirmations | author: Example CERT | confidence: 70 | objects: 7
2026-01-15 Velvet Harbor reuses guest Wi-Fi portal infrastructure | author: Example CERT | confidence: 60 | objects: 7
```

Each report keeps the confidence of its feed item, because the administrator's maximum confidence level is 100. With a connector user capped at 50, both reports would show 50.

When the **Works** tab counts errors, open them. A common one on a first run is `INDICATOR_PATTERN_EXCLUDED`: an indicator matches an exclusion list of :menu[Settings > Customization > Exclusion lists], and the platform refuses it, then refuses the relationships that point to it with `ELEMENT_NOT_FOUND`. The fix is in the data or in the list, not in the connector.

Run `python main.py` again: the connector logs `No new item`, because its state holds the date of the last item. To import again, reset the connector from its page: **Reset the connector state** forgets the state and purges its queue.

## Step 7: Describe it for the catalog

XTM Composer deploys connectors from the catalog of the platform. To be listed there, a connector ships two files in `__metadata__/`: `connector_manifest.json`, which describes it, and `connector_config_schema.json`, which lists its settings and becomes the form an administrator fills. pycti also reads the manifest at start: the connector reports its `container_version` and `slug` when it registers.

`__metadata__/connector_manifest.json`:

```json
{
  "title": "Hospitality threat feed",
  "slug": "hospitality-feed",
  "description": "Imports the reports of the Example CERT hospitality threat feed: the intrusion sets that target hotel chains, their indicators, and the report that ties them together.",
  "short_description": "Reports, intrusion sets and indicators on threats to the hospitality sector.",
  "logo": null,
  "use_cases": ["Market Vertical & Mission-Specific Intelligence"],
  "solution_categories": ["Threat Intelligence Feed"],
  "license_type": "Free",
  "contact": "cti@example.com",
  "verified": false,
  "last_verified_date": null,
  "playbook_supported": false,
  "max_confidence_level": 70,
  "support_version": ">=7.261030",
  "subscription_link": "https://feeds.example.com/hospitality",
  "source_code": "https://git.example.com/example-cert/connector-hospitality-feed",
  "manager_supported": true,
  "container_version": "1.0.0",
  "container_image": "registry.example.com/example-cert/connector-hospitality-feed",
  "container_type": "EXTERNAL_IMPORT"
}
```

[Integration manifest](/develop/build/manifest/) explains every field, writes the configuration schema of this connector, and builds a catalog that OpenCTI can load.

## Step 8: Run it in Docker on a schedule

Package the connector as an image. The `Dockerfile` installs the dependencies, copies the code, and copies the manifest of step 7, from which pycti reads the version and the slug the connector reports when it registers:

`Dockerfile`:

```dockerfile
FROM python:3.12-slim
ENV PYTHONUNBUFFERED=1
WORKDIR /opt/connector-hospitality-feed
COPY src/requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY src .
# pycti reports the version and slug of the manifest when the connector registers
COPY __metadata__/connector_manifest.json __metadata__/
ENTRYPOINT ["python", "main.py"]
```

Add a `.dockerignore` with `src/config.yml`, so that your local configuration stays out of the image. In Docker, every setting is an environment variable. The service joins the network of the XTM stack, `xtm_default`, where OpenCTI is `http://opencti:8080` and RabbitMQ is reachable, so the connector uses RabbitMQ, the default.

:::example[Import the Example CERT hospitality feed every hour]
Example CERT runs the connector next to its XTM stack. The feed is at `https://feeds.example.com/hospitality/v1/items.json`; its key and the connector's token are in the `.env` file of the deployment, as `HOSPITALITY_FEED_API_KEY` and `OPENCTI_TOKEN`.

`docker-compose.yml`:

```yaml
services:
  connector-hospitality-feed:
    build: .
    image: example-cert/connector-hospitality-feed:1.0.0
    environment:
      - OPENCTI_URL=http://opencti:8080
      - OPENCTI_TOKEN=${OPENCTI_TOKEN}
      - CONNECTOR_ID=0e5a7f8c-2b4d-4c1e-9f3a-6d8b1c2e4f70
      - CONNECTOR_TYPE=EXTERNAL_IMPORT
      - CONNECTOR_NAME=Hospitality threat feed
      - CONNECTOR_SCOPE=report,indicator,intrusion-set
      - CONNECTOR_LOG_LEVEL=info
      - CONNECTOR_DURATION_PERIOD=PT1H
      - HOSPITALITY_FEED_URL=https://feeds.example.com/hospitality/v1/items.json
      - HOSPITALITY_FEED_API_KEY=${HOSPITALITY_FEED_API_KEY}
    restart: always
    networks:
      - xtm

networks:
  xtm:
    external: true
    name: xtm_default
```

```bash
docker compose up -d --build
docker compose logs connector-hospitality-feed
```

In Docker, the logs are JSON lines. After the first run, the connector waits for the next one:

```text
{"timestamp": "2026-11-01T22:57:41.169417Z", "level": "INFO", "name": "Hospitality threat feed", "message": "Connector registered with ID", "attributes": {"id": "0e5a7f8c-2b4d-4c1e-9f3a-6d8b1c2e4f70"}}
{"timestamp": "2026-11-01T22:57:41.175210Z", "level": "INFO", "name": "Hospitality threat feed", "message": "No new item", "attributes": {"since": "2026-01-15T07:30:00Z"}}
{"timestamp": "2026-11-01T22:57:41.192690Z", "level": "INFO", "name": "Hospitality threat feed", "message": "Schedule next run of connector", "attributes": {"next_run_datetime": "2026-11-01T23:57:41Z"}}
```

Every hour, the connector imports the items published since its last run. Analysts find them in :menu[Analyses > Reports], authored by `Example CERT`, and the **Works** tab of the connector records each run.
:::

## Share it

To publish the connector to the community, contribute it to [OpenCTI-Platform/connectors](https://github.com/OpenCTI-Platform/connectors). Its templates and guidelines build on `connectors-sdk`, a layer over the same helper that adds typed settings and a ready-made scheduling loop.

## What you learned

- A connector registers with its token and writes as that token's user, with its confidence level: [Connectors](/develop/integration-model/connectors/).
- Deterministic identifiers, authors, markings and confidence make imported data merge cleanly: [Script OpenCTI in Python](/develop/build/python-client/) shows the same identifiers from the API side.
- Works and state make each run visible and resumable, and a manifest makes the connector deployable from the interface: [Integration manifest](/develop/build/manifest/).
