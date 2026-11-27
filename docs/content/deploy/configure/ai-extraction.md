---
title: Extract intelligence from documents on premises
description: Run the extraction web service of the Import Document AI connector in your own stack, so that OpenCTI extracts threat intelligence from documents on premises.
doc_type: how-to
products:
  - opencti
edition: ee
personas: [platform-engineer]
owner: docs-platform
last_reviewed: 2026-10-31
migrated_from:
  - opencti:/deployment/advanced/ai-import-document/
---

The Import Document AI connector of OpenCTI extracts entities, observables and relationships from PDF, text, HTML and Markdown files, and imports them as a STIX bundle. It sends each document to an extraction web service, the one Filigran hosts or one you run. This page runs that web service in your own stack, for an air-gapped network or to keep documents on your premises, and points the connector to it.

::diagram{id=deploy-configure-ai-extraction}

:::only{opencti=">=7.260811"}
The connector has no default address: without an XTM One agent, it needs the address of a web service and your license certificate, or every import fails with `No agent_slug provided and api_base_url/api_key is not configured`.
:::

:::only{opencti="<7.260811"}
Without `CONNECTOR_WEB_SERVICE_URL`, the connector calls `https://importdoc.ariane.testing.filigran.io`, a testing address of Filigran: set the address of your web service explicitly.
:::

::::only{opencti=">=7.260811"}
:::xtm-one[Extract with an XTM One agent]
When OpenCTI is [registered with XTM One](/deploy/configure/connect-modules/), the connector can hand the document to an XTM One AI agent instead of the web service. The import dialog of OpenCTI then offers the agents bound to the intent `cti.stix_harvester` (the connector setting `CONNECTOR_XTM_ONE_INTENT`), and the extraction runs in your XTM One with the model you configured there. No web service is needed for those imports.
:::
::::

## Before you start

- OpenCTI with an Enterprise Edition license, and the license certificate that Filigran gave you, in PEM format: the connector sends it to the web service.
- An access token to the private Docker repository of Filigran, from your Customer Success Manager or Account Manager.
- A host that fits the web service. These figures come from Filigran; extraction time depends on the size of the file.

  | | With a GPU | Without a GPU |
  | --- | --- | --- |
  | Computing | An NVIDIA GPU | CPU at 3 GHz or more |
  | RAM | 8 GB | 16 GB |
  | Disk | 16 GB | 16 GB |
  | Average extraction time | 5 to 10 seconds | 30 seconds to 2 minutes |

- The stack installed as [Install with Docker Compose](/deploy/install/docker-compose/) describes.

## Run the web service

:::steps
1. Sign in to the private repository of Filigran, and enter the token when Docker asks for the password.

   ```bash
   docker login --username filigrancustomers
   ```

   To run the service where Docker Hub is not reachable, pull `filigran/import-document-ai-webservice` here and push it to your own registry.

2. Add the service to `docker-compose.override.yml`, under `services:`, as [Change the stack without editing docker-compose.yml](/deploy/install/docker-compose/#change-the-stack-without-editing-docker-compose-yml) describes. It listens on port 8000 inside the stack; the connector reaches it there, so it needs no published port. Keep the `deploy` block only on a host with an NVIDIA GPU.

   ```yaml
     import-document-ai-webservice:
       image: filigran/import-document-ai-webservice
       restart: always
       deploy:
         resources:
           reservations:
             devices:
               - driver: nvidia
                 count: 1
                 capabilities: [gpu]
   ```

3. Start it.

   ```bash
   docker compose pull import-document-ai-webservice
   docker compose up -d import-document-ai-webservice
   ```

   :::verify
   ```bash
   docker compose ps import-document-ai-webservice
   ```
   The service is `running`.
   :::
:::

## Point the connector to it

The connector is not part of xtm-docker. Add it to `docker-compose.override.yml` with the address of your web service.

:::steps
1. Give the connector an identifier in `.env`.

   ```bash
   echo "CONNECTOR_IMPORT_DOCUMENT_AI_ID=$(python3 -c 'import uuid; print(uuid.uuid4())')" >> .env
   ```

2. Add the connector under `services:`, with your license certificate in place of the three certificate lines: the whole PEM text, from `-----BEGIN CERTIFICATE-----` to `-----END CERTIFICATE-----`, indented as shown.

   :::only{opencti=">=7.260811"}
   ```yaml
     connector-import-document-ai:
       image: opencti/connector-import-document-ai:latest
       environment:
         OPENCTI_URL: http://opencti:8080
         OPENCTI_TOKEN: ${OPENCTI_ADMIN_TOKEN}
         CONNECTOR_ID: ${CONNECTOR_IMPORT_DOCUMENT_AI_ID}
         CONNECTOR_NAME: ImportDocumentAI
         CONNECTOR_SCOPE: application/pdf,text/plain,text/html,text/markdown
         CONNECTOR_AUTO: "false"
         CONNECTOR_LOG_LEVEL: info
         IMPORT_DOCUMENT_AI_API_BASE_URL: http://import-document-ai-webservice:8000
         IMPORT_DOCUMENT_AI_API_KEY: |
           -----BEGIN CERTIFICATE-----
           Your license certificate
           -----END CERTIFICATE-----
       depends_on:
         opencti:
           condition: service_healthy
       restart: always
   ```

   The connector calls `/stix` under `IMPORT_DOCUMENT_AI_API_BASE_URL`.
   :::

   :::only{opencti="<7.260811"}
   ```yaml
     connector-import-document-ai:
       image: opencti/connector-import-document-ai:latest
       environment:
         OPENCTI_URL: http://opencti:8080
         OPENCTI_TOKEN: ${OPENCTI_ADMIN_TOKEN}
         CONNECTOR_ID: ${CONNECTOR_IMPORT_DOCUMENT_AI_ID}
         CONNECTOR_NAME: ImportDocumentAI
         CONNECTOR_SCOPE: application/pdf,text/plain,text/html,text/markdown
         CONNECTOR_AUTO: "false"
         CONNECTOR_LOG_LEVEL: info
         CONNECTOR_WEB_SERVICE_URL: http://import-document-ai-webservice:8000
         CONNECTOR_LICENCE_KEY_PEM: |
           -----BEGIN CERTIFICATE-----
           Your license certificate
           -----END CERTIFICATE-----
       depends_on:
         opencti:
           condition: service_healthy
       restart: always
   ```

   The connector calls `/extract_entities_relations` under `CONNECTOR_WEB_SERVICE_URL`.
   :::

   Use the image tag of your OpenCTI version in place of `latest`, as for the other connectors.

3. Start the connector.

   ```bash
   docker compose up -d connector-import-document-ai
   ```

4. In OpenCTI, go to :menu[Data > Import], upload a PDF file, and launch an import with the connector `ImportDocumentAI`.

   :::verify
   The import completes and its entities appear in OpenCTI.

   :::only{opencti=">=7.260811"}
   The connector log names your web service:

   ```bash
   docker compose logs connector-import-document-ai | grep "HTTP Post Request"
   ```
   The line holds `http://import-document-ai-webservice:8000/stix`.
   :::
   :::
:::

If the import fails with `ImportDocumentAI webservice seems to be unreachable`, the connector cannot reach the address you set: check the service name and the port.

## Connect OpenCTI to your own language model

The connector above handles documents only. The other AI features of OpenCTI, such as the summaries of an entity, the search in natural language and the text generation in the editors, call the language model set in the `ai` keys of the platform. To keep those prompts on your premises too, point these keys to a model you host.

| Key | Default | What it does |
| --- | --- | --- |
| `ai:enabled` | not set | Set to `true` to connect the model. Unset or `false`, the AI features have no model |
| `ai:type` | `mistralai` | The API of the model: `mistralai`, `openai` or `azureopenai`. With `ai:enabled` and `ai:token` set, any other value stops the platform at start |
| `ai:endpoint` | empty | URL of the model API. Empty: the public cloud of the provider. For the Mistral AI cloud, set `https://api.mistral.ai`: with `mistralai` and any other URL, OpenCTI calls an OpenAI-compatible API at the URL followed by `/v1`, as a model served by vLLM exposes |
| `ai:token` | empty | The API key. Without it, the AI features have no model |
| `ai:model` | empty | The model name, as the API knows it |
| `ai:max_tokens` | `30000` | Length of the prompt sent to the model. Despite its name, OpenCTI cuts the prompt to this number of characters, not tokens |
| `app:ai:timeout` | `60000` | Time, in milliseconds, a search in natural language waits for the model before it fails. Raise it for a slow model |

For Azure OpenAI, set `ai:type` to `azureopenai`, with `ai:version`, the API version of your deployment, `ai:ai_azure_instance`, the instance name, such as `example-cert` for `https://example-cert.openai.azure.com`, and `ai:ai_azure_deployment`, the deployment name.

Two keys of `config/default.json` have no effect: `ai:timeout`, since the timeout is read from `app:ai:timeout` (`APP__AI__TIMEOUT`), and `ai:model_images`, which no code reads. The platform also needs the **Generative AI (AI Insight, NLQ)** switch on, in :menu[Settings > Filigran Experience]. Every other AI key is in the [AI Service section](/reference/opencti/configuration/#ai-service) of the configuration reference.

:::example[Use the model of Example CERT's GPU cluster]
Example CERT serves a Mistral model with vLLM at `https://llm.example.com`, behind its own API key, kept in `.env` as `OPENCTI_AI_TOKEN`. It adds to `docker-compose.override.yml`:

```yaml
services:
  opencti:
    environment:
      - AI__ENABLED=true
      - AI__TYPE=mistralai
      - AI__ENDPOINT=https://llm.example.com
      - AI__TOKEN=${OPENCTI_AI_TOKEN}
      - AI__MODEL=mistralai/Mistral-Small-3.2-24B-Instruct-2506
      - APP__AI__TIMEOUT=120000
```

After `docker compose up -d opencti`, the summaries and the search in natural language of OpenCTI use this model, and no prompt leaves the network.
:::

## What's next

- [Run an integration without XTM Composer](/deploy/integrations/manual/), for the other settings of a connector run this way.
- [Install offline or with FIPS 140-3](/deploy/install/air-gapped/).
- [Connect the modules to each other](/deploy/configure/connect-modules/), to extract with an XTM One agent instead.
