---
title: Community resources
description: Where the integrations of the XTM suite live - catalogs, XTM Composer, connector repositories, libraries and images - and where to learn and ask questions.
doc_type: reference
products:
  - opencti
personas: [developer, platform-engineer]
owner: docs-develop
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/deployment/resources/
---

Filigran and the community publish hundreds of integrations for OpenCTI and OpenAEV. This page lists where each piece of that ecosystem lives: the catalogs where you find an integration, XTM Composer that runs it, the repositories that hold its code, the libraries you build with, and the places to learn and ask. Each entry says what you go there for.

::screenshot{id=opencti-ingest-integrations-available alt="The Available tab of Integrations lists the connectors of the catalog as cards, with filters by use case and solution category on the side."}

## Catalogs

| Resource | What you find there |
| --- | --- |
| The catalog of the platform: :menu[Integrations], **Available** tab in OpenCTI (:menu[Data > Ingestion > Connectors catalog] before 7.260811); **Integrations** > **Catalog** in OpenAEV | The integrations your platform version can run, with their description, their settings and, with XTM Composer, a **Deploy** button. OpenCTI builds it from the catalog embedded in each release and from your own catalogs ([Integration manifest](/develop/build/manifest/)) |
| [XTM Hub integrations library](https://hub.filigran.io/cybersecurity-solutions/open-cti-integrations) | The same integrations, to browse without a platform, with the use cases they serve |

## XTM Composer

| Resource | What you find there |
| --- | --- |
| [FiligranHQ/xtm-composer](https://github.com/FiligranHQ/xtm-composer) | The source of XTM Composer, the component that deploys integrations from the catalog on Docker, Podman or Kubernetes |
| [XTM Composer](/deploy/integrations/composer/) and [Install XTM Composer](/deploy/integrations/install-composer/) | What it does, what it needs, how to install and configure it |

## Repositories

| Repository | Content |
| --- | --- |
| [OpenCTI-Platform/connectors](https://github.com/OpenCTI-Platform/connectors) | The connectors of OpenCTI, one folder each under `external-import/`, `internal-enrichment/`, `internal-import-file/`, `internal-export-file/` and `stream/`. Also `templates/` (a starting point per connector type), `connectors-sdk/`, `shared/tools/composer/` (the scripts that build the manifests and the catalog) and `docs/` (the implementation guidelines a contribution must follow) |
| [OpenCTI-Platform/opencti](https://github.com/OpenCTI-Platform/opencti) | The platform, the worker (`opencti-worker/`) and, since version 7, the Python library (`client-python/`) with its `examples/` |
| [OpenAEV-Platform/collectors](https://github.com/OpenAEV-Platform/collectors) | The collectors of OpenAEV |
| [OpenAEV-Platform/injectors](https://github.com/OpenAEV-Platform/injectors) | The injectors of OpenAEV |
| [FiligranHQ/xtm-docker](https://github.com/FiligranHQ/xtm-docker) | The Docker Compose deployment of the suite, with its default integrations |

Each connector folder has the same layout: `src/` for the code, `__metadata__/` for the manifest and the configuration schema, a `Dockerfile`, a `docker-compose.yml`, and a `README.md` that lists its settings.

## Libraries

| Library | Language | Use it to | Install |
| --- | --- | --- | --- |
| `pycti` | Python | Call the OpenCTI API, and write connectors with its `OpenCTIConnectorHelper`: [Script OpenCTI in Python](/develop/build/python-client/) | From [PyPI](https://pypi.org/project/pycti/), at the version of your platform: `pip install pycti==7.261030.1` for OpenCTI 7.261030.1 |
| `connectors-sdk` | Python | Write connectors with typed settings, STIX models and a ready-made run loop, on top of `pycti`. The connectors repository templates use it | From the `connectors-sdk/` folder of the connectors repository, with `pip install "connectors-sdk @ git+https://github.com/OpenCTI-Platform/connectors.git@master#subdirectory=connectors-sdk"` |
| `pyoaev` | Python | Call the OpenAEV API, and write collectors and injectors | From [PyPI](https://pypi.org/project/pyoaev/) |

## Images

The integrations built by Filigran are published on Docker Hub, one image per integration: `opencti/connector-*` for OpenCTI, such as [opencti/connector-export-file-stix](https://hub.docker.com/r/opencti/connector-export-file-stix/tags), and `openaev/collector-*` and `openaev/injector-*` for OpenAEV. Their tags follow the platform versions; XTM Composer pulls the tag of the catalog version.

## Learn and ask

| Resource | What you find there |
| --- | --- |
| [Filigran Community on Slack](https://community.filigran.io) | Questions to the community and the Filigran teams |
| [Filigran Academy](https://academy.filigran.io) | Training courses for analysts and administrators, with certificates |
| [YouTube channel](https://www.youtube.com/@Filigran) | Demonstrations, use cases, webinars |
| [Filigran blog](https://blog.filigran.io) | Articles by Filigran and the community on features and use cases |
| [Content library](https://filigran.io/content-library/) | Briefs, reports, white papers and recorded webinars |
| [Customer stories](https://filigran.io/customer-story-library/) | How organizations use the suite |

## What's next

- [Integration model](/develop/integration-model/), the kinds of integration and what they share.
- [Build an OpenCTI connector](/develop/build/connector/), your first connector.
