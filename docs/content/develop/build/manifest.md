---
title: Integration manifest
description: The connector manifest and configuration schema that the OpenCTI catalog shows and XTM Composer deploys from, field by field, and how to build a catalog of your own connectors.
doc_type: reference
products:
  - opencti
  - composer
personas: [developer, platform-engineer]
owner: docs-develop
last_reviewed: 2026-11-01
---

The manifest describes a connector to the OpenCTI catalog: its name and description on the catalog card, the Docker image XTM Composer deploys, and the settings the deployment form asks for. A connector ships it as two files in its `__metadata__/` folder; a catalog gathers the manifests of many connectors in one JSON file that OpenCTI loads. This page lists every field as OpenCTI 7.261030 reads it, then builds the catalog of the [hospitality threat feed connector](/develop/build/connector/).

::screenshot{id=opencti-ingest-connectors-catalog alt="The connector catalog shows one card per manifest, with its logo, title, short description, use cases and a Deploy button for the connectors XTM Composer can run."}

## The files

| File | Holds | Read by |
| --- | --- | --- |
| `__metadata__/connector_manifest.json` | What the catalog shows and what XTM Composer deploys | The catalog build; pycti, which reports `container_version` and `slug` when the connector registers |
| `__metadata__/connector_config_schema.json` | The connector's settings, as a JSON Schema | The catalog build, which copies it into the manifest as `config_schema` |
| `__metadata__/logo.png` | The logo of the card (any `logo.*` file) | The catalog build, which embeds it as a `data:` URL in `logo` |
| A catalog, such as `manifest.json` | `id`, `name`, `description`, `version`, and the `contracts`: one manifest per connector | OpenCTI, at start and every `catalog_manager:interval` |

The [OpenCTI-Platform/connectors](https://github.com/OpenCTI-Platform/connectors) repository builds its catalog, `manifest.json`, from the `__metadata__/` folders of its connectors, and each OpenCTI release embeds it. Your own catalogs come on top of it: see [Load your catalog](#load-your-catalog).

## connector_manifest.json

| Field | Type | What OpenCTI does with it |
| --- | --- | --- |
| `title` | String | The name on the card and the default display name of a deployment |
| `slug` | String, lowercase words joined by `-` | The identifier of the connector across versions. A deployed connector follows the latest compatible version of its slug |
| `description` | String | The description on the connector's page in the catalog |
| `short_description` | String | The text of the card |
| `logo` | `data:` URL, URL or `null` | The logo of the card. The catalog build fills it from `logo.*` |
| `use_cases` | Strings | Facets of the catalog, such as `Market Vertical & Mission-Specific Intelligence` or `Vulnerability & Exploit Awareness` |
| `solution_categories` | Strings | Facets of the catalog, such as `Threat Intelligence Feed` or `Enrichment & Reputation` |
| `license_type` | `Free`, `Commercial` or `null` | Shown on the card |
| `contact` | String | Who supports the connector |
| `verified`, `last_verified_date` | Boolean, date or `null` | Whether Filigran verified the connector, and when. Deploying a connector that is not verified asks the administrator to acknowledge that Filigran does not support it |
| `playbook_supported` | Boolean | Whether the connector can run as a playbook step. Match it with `playbook_compatible=True` in the connector code |
| `max_confidence_level` | Integer, 0 to 100 | The confidence level the deployment form proposes for the service account it creates. 100 when absent |
| `support_version` | Semantic version range, such as `>=7.261030.0` | The platform versions the connector works with. The catalog marks the connector incompatible on other versions |
| `subscription_link` | URL or `null` | Where to subscribe to the data source |
| `source_code` | URL | Where the code lives |
| `manager_supported` | Boolean | `true`: the card has a **Deploy** button and XTM Composer can run the connector. With a `config_schema`, `container_image` and `container_type` are required, or OpenCTI refuses the whole catalog; without one, OpenCTI logs a warning |
| `container_image` | String, without tag | The Docker image XTM Composer pulls |
| `container_version` | String | The image tag, `rolling` or a version. XTM Composer runs `container_image:container_version`; pycti reports it as the connector's version |
| `container_type` | Connector type | `EXTERNAL_IMPORT`, `INTERNAL_ENRICHMENT`, `INTERNAL_IMPORT_FILE`, `INTERNAL_EXPORT_FILE`, `INTERNAL_ANALYSIS` or `STREAM` |
| `config_schema` | JSON Schema | Filled from `connector_config_schema.json` by the catalog build |

The manifest of the hospitality threat feed connector:

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

## connector_config_schema.json

A JSON Schema (draft 2020-12) of type `object`. Each property is a setting, named after its environment variable, because XTM Composer passes the values to the container as environment variables.

| Keyword | Effect in the deployment form and in XTM Composer |
| --- | --- |
| `type` | `string`, `integer`, `boolean`, `array` (of strings) or `dict` |
| `description` | The help text of the field |
| `default` | The value the form proposes. A field left empty takes it |
| `enum` | A choice list |
| `format: password` | A secret: the form hides it, OpenCTI stores it encrypted with the public key of XTM Composer, never returns it, and keeps the stored value when an update leaves the field empty. Mark every key and password this way |
| `required` | The fields the form asks for first; the others are under the optional settings |
| `deprecated: true` | Hidden from the form |
| `additionalProperties` | Keep `true`: a deployment can carry settings the schema does not list |

OpenCTI validates every deployment against the schema and shows the errors in the form. A few settings never appear in the form, because the platform and XTM Composer set them: `OPENCTI_URL`, `OPENCTI_TOKEN`, `CONNECTOR_TYPE` and `CONNECTOR_RUN_AND_TERMINATE`. `CONNECTOR_ID` comes from the deployment, and the default of `CONNECTOR_NAME` becomes the default display name. `OPENCTI_TOKEN` is a token of the deployment's service account, which OpenCTI creates and encrypts for XTM Composer.

The schema of the hospitality threat feed connector:

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://example.com/connectors/hospitality-feed_config.schema.json",
  "type": "object",
  "properties": {
    "OPENCTI_URL": {
      "description": "The base URL of the OpenCTI instance.",
      "format": "uri",
      "type": "string"
    },
    "OPENCTI_TOKEN": {
      "description": "The API token to connect to OpenCTI.",
      "format": "password",
      "type": "string",
      "writeOnly": true
    },
    "CONNECTOR_NAME": {
      "default": "Hospitality threat feed",
      "description": "The name of the connector.",
      "type": "string"
    },
    "CONNECTOR_SCOPE": {
      "default": ["report", "indicator", "intrusion-set"],
      "description": "The types of objects the connector imports.",
      "items": {"type": "string"},
      "type": "array"
    },
    "CONNECTOR_LOG_LEVEL": {
      "default": "error",
      "description": "The minimum level of logs to display.",
      "enum": ["debug", "info", "warn", "warning", "error"],
      "type": "string"
    },
    "CONNECTOR_DURATION_PERIOD": {
      "default": "PT1H",
      "description": "Time between two runs, as an ISO 8601 duration.",
      "format": "duration",
      "type": "string"
    },
    "HOSPITALITY_FEED_URL": {
      "default": "https://feeds.example.com/hospitality/v1/items.json",
      "description": "URL of the feed.",
      "format": "uri",
      "type": "string"
    },
    "HOSPITALITY_FEED_API_KEY": {
      "description": "API key of the feed.",
      "format": "password",
      "type": "string",
      "writeOnly": true
    },
    "HOSPITALITY_FEED_AUTHOR": {
      "default": "Example CERT",
      "description": "Organization set as the author of every imported object.",
      "type": "string"
    }
  },
  "required": ["OPENCTI_URL", "OPENCTI_TOKEN", "HOSPITALITY_FEED_API_KEY"],
  "additionalProperties": true
}
```

## Build a catalog

A catalog is a JSON object with an `id`, a `name`, a `description`, a `version` and the list of manifests in `contracts`. This script builds one from every `__metadata__/` folder under the current directory, and runs the checks OpenCTI runs when it loads the catalog. It is stricter on one point: OpenCTI only logs a warning for a `manager_supported` connector without `config_schema`, while the script stops, because the deployment form needs the schema:

```python
"""Builds a custom OpenCTI catalog from the __metadata__ folder of each connector."""

import json
import sys
from pathlib import Path

from jsonschema import Draft202012Validator

contracts = []
for metadata in sorted(Path(".").glob("**/__metadata__")):
    contract = json.loads((metadata / "connector_manifest.json").read_text())
    schema_file = metadata / "connector_config_schema.json"
    if schema_file.is_file():
        contract["config_schema"] = json.loads(schema_file.read_text())
    # What OpenCTI checks when it loads the catalog (it only warns when config_schema is missing)
    if contract["manager_supported"]:
        for field in ("container_image", "container_type", "config_schema"):
            if not contract.get(field):
                sys.exit(f"{contract['slug']}: {field} is required for XTM Composer")
        Draft202012Validator.check_schema(contract["config_schema"])
    contracts.append(contract)
    print(f"{contract['slug']} {contract['container_version']}: ok")

catalog = {
    "id": "example-cert-catalog",
    "name": "Example CERT connectors",
    "description": "Connectors built by Example CERT for its member hotel chains",
    "version": "7.261030.1",
    "contracts": contracts,
}
Path("example-cert-catalog.json").write_text(json.dumps(catalog, indent=2))
```

It needs `jsonschema` (`pip install jsonschema`). Run it from the folder that holds your connectors:

```bash
python build_catalog.py
```

```text
hospitality-feed 1.0.0: ok
```

The result, `example-cert-catalog.json`, starts like this:

```json
{
  "id": "example-cert-catalog",
  "name": "Example CERT connectors",
  "description": "Connectors built by Example CERT for its member hotel chains",
  "version": "7.261030.1",
  "contracts": [
    {
      "title": "Hospitality threat feed",
      "slug": "hospitality-feed",
      ...
      "config_schema": {
        "$schema": "https://json-schema.org/draft/2020-12/schema",
        ...
```

OpenCTI also reads a newer catalog format, marked with `"manifest_schema_version": "1"`, where each contract carries `version`, `image_name`, `image_type`, `min_version` and `max_version`, and puts `playbook_supported` and `max_confidence_level` under `additional_properties`. Write the format above unless you generate catalogs with the tools of the connectors repository.

## Load your catalog

List your catalogs in `app:custom_catalogs` (`APP__CUSTOM_CATALOGS`) of the platform, as a JSON array of file paths the platform can read or `http(s)://` URLs. The catalog manager loads them next to the built-in catalog; their connectors appear in :menu[Integrations], **Available** tab, and XTM Composer deploys the ones with `manager_supported: true`. [XTM Composer](/deploy/integrations/composer/#add-your-own-connectors-to-the-opencti-catalog) shows the Docker Compose block. Filigran may remove this setting in a later version.

Push the image to a registry XTM Composer can pull from: [Pull integrations from a private registry](/deploy/integrations/private-registry/) covers a registry that needs credentials.

## What's next

- [Build an OpenCTI connector](/develop/build/connector/), the connector this manifest describes.
- [XTM Composer](/deploy/integrations/composer/), what it does with the catalog.
- [Community resources](/develop/integration-model/resources/), where the catalogs and repositories live.
