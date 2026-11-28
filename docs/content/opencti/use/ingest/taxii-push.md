---
title: Let partners push intelligence to you
description: Expose a TAXII 2.1 collection on OpenCTI so that partners and tools push STIX objects into your platform, and choose who may push.
sidebar_position: 4
doc_type: how-to
products:
  - opencti
personas: [cti-lead, cti-analyst, admin]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/import/taxii-push/
---

TAXII Push turns the usual direction around: instead of OpenCTI polling a server, OpenCTI exposes a Trusted Automated Exchange of Intelligence Information (TAXII) 2.1 collection, and partners or tools send STIX 2.1 objects to it as soon as they have them. You decide who may push, and under which account their data is written. It follows the "Add Objects" endpoint of the [TAXII 2.1 specification](https://docs.oasis-open.org/cti/taxii/v2.1/os/taxii-v2.1-os.html). For how imported data reaches the knowledge base, read [How intelligence gets into OpenCTI](/opencti/use/ingest/).

## Before you start

- Your role has the **Manage ingestion** capability.
- The users who will push exist on your platform, each with an API token and the **Access data sharing** capability: without it, the platform refuses their requests. A service account per partner keeps their data apart: see [Create accounts for integrations](/opencti/administer/service-accounts/).

## Create the collection

Open the creation form:

::::only{opencti=">=7.260811"}
Open :menu[Integrations], select the **Available** tab, and click **Create** on the **TAXII Push** card.
::::

::::only{opencti="<7.260811"}
Open :menu[Data > Ingestion > TAXII push] and click the create button.
::::

Then fill it in:

:::steps

1. Enter a **Name** and a **Description**.
2. Choose the account that writes the data in **User responsible for data creation (empty = System)**. Leave it empty only if the data should be written with the full rights of the system.
3. In **Accessible for**, add the users, groups or organizations allowed to push to this collection. Users with the **Manage ingestion** capability can always push.
4. Turn on **Copy confidence level to OpenCTI scores for indicators** to set the score of each pushed indicator from its STIX confidence.
5. Click **Create**.
6. Open the actions menu of the new collection and choose **Start**. A stopped collection answers that it does not exist.

:::

::screenshot{id=opencti-ingest-taxii-push-create alt="The TAXII Push creation form has only a name, a description, the user responsible for data creation, the Accessible for field and the confidence-to-score switch."}

## Tell the partner how to push

Give the partner three things: the address of the collection, the API token of the account you allowed in **Accessible for**, and the format to use. Their developers, or the tool they configure, then send objects to the collection with an HTTP `POST`:

```text
POST https://<platform>/taxii2/root/collections/<collection-id>/objects/
Authorization: Bearer <token of a user allowed to push>
Content-Type: application/taxii+json;version=2.1
```

The body is a TAXII envelope with an `objects` array of STIX 2.1 objects. `application/vnd.oasis.stix+json;version=2.1` is accepted too. A request body larger than 50 MB is refused: a partner with more to send splits its objects into several requests, or a platform engineer raises `app:max_payload_body_size` (`APP__MAX_PAYLOAD_BODY_SIZE`, see [Network and security](/reference/opencti/configuration/#network-and-security)).

:::only{opencti=">=7.261119"}
On a platform that runs with the [secure HTTP defaults](/deploy/operate/harden/#secure-http-defaults) and leaves `app:max_payload_body_size` unset, the limit is 10 MB instead. Ask your platform engineer which limit applies before you tell the partner. Their TAXII client can read it too: the API root, `https://<platform>/taxii2/root/`, gives it in bytes as `max_content_length`.
:::

::::only{opencti=">=7.260811"}
The collection ID is the last part of the address of the collection's page, which you open from the **Deployed** tab of :menu[Integrations].
::::

::::only{opencti="<7.260811"}
The list in :menu[Data > Ingestion > TAXII push] shows the address of each collection, with a button to copy it.
::::

The answer is a TAXII status with an `id`. Clients follow the processing at `https://<platform>/taxii2/root/status/<id>/`, which counts the objects pending, processed and failed.

Before you give the address to the partner, a platform engineer or the partner can check it with one test object:

:::steps

1. Send a test bundle with one object:

   ```bash
   curl -X POST "https://<platform>/taxii2/root/collections/<collection-id>/objects/" \
     -H "Authorization: Bearer $TOKEN" \
     -H "Content-Type: application/taxii+json;version=2.1" \
     -d '{"objects":[{"type":"identity","spec_version":"2.1","id":"identity--2c5c6a2e-6a1f-4e7c-9d3b-0b0f7c6f1a11","created":"2026-01-15T10:00:00.000Z","modified":"2026-01-15T10:00:00.000Z","name":"Fictional Test Partner","identity_class":"organization"}]}'
   ```

2. Check the status the answer points to.

:::

:::verify[The push was accepted]
The first answer has `"status": "pending"` and a `total_count` of 1. The status endpoint then shows `"status": "complete"` with a `success_count` of 1, and the organization appears in OpenCTI.
:::

:::example[Let a member hotel push the indicators it finds]
Grand Meridian Hotels, a fictional hotel chain, runs a sandbox that detects the phishing emails sent to its front desks. Its security team wants to push each new indicator to your platform as soon as it is found.

1. Create the service account `Grand Meridian Hotels push`, author of its data, and give its token to the hotel's security team.
2. Create the collection:
   - **Name**: `Grand Meridian Hotels indicators`
   - **Description**: `Indicators pushed by the sandbox of Grand Meridian Hotels`
   - **User responsible for data creation (empty = System)**: `Grand Meridian Hotels push`
   - **Accessible for**: `Grand Meridian Hotels push`, and nobody else
   - **Copy confidence level to OpenCTI scores for indicators**: on, as the sandbox sets a confidence on each indicator
3. **Start** the collection and send its ID, here `4d1e9b52-7a3c-4f08-b6e2-91c5a0d37f14`, to the hotel.

The sandbox then pushes each indicator, with the token in `GMH_PUSH_TOKEN`:

```bash
curl -X POST "https://opencti.example.com/taxii2/root/collections/4d1e9b52-7a3c-4f08-b6e2-91c5a0d37f14/objects/" \
  -H "Authorization: Bearer ${GMH_PUSH_TOKEN}" \
  -H "Content-Type: application/taxii+json;version=2.1" \
  -d '{"objects":[{"type":"indicator","spec_version":"2.1","id":"indicator--6b0f3c1e-2d8a-4e57-9f40-3a1c7e5b9d26","created":"2026-01-15T09:12:00.000Z","modified":"2026-01-15T09:12:00.000Z","name":"frontdesk-booking-confirm.example","pattern":"[domain-name:value = '"'"'frontdesk-booking-confirm.example'"'"']","pattern_type":"stix","valid_from":"2026-01-15T09:12:00.000Z","confidence":85}]}'
```

The indicator appears in :menu[Observations > Indicators] with a score of 85, created by `Grand Meridian Hotels push`. A push with the token of another account, which is not in **Accessible for** and lacks the **Manage ingestion** capability, gets the answer that the collection does not exist.
:::

The platform refuses an empty `objects` array and a missing or wrong `Content-Type`. It answers that the collection does not exist to a user who is neither in **Accessible for** nor has the **Manage ingestion** capability.

::::only{opencti=">=7.261127"}
The collection and status endpoints answer a refusal with a TAXII error, which holds an `http_status` and a `title` that says what to fix:

| `http_status` | `title` | Cause |
| --- | --- | --- |
| 401 | `You must be logged in to do this.` | The request has no valid token |
| 403 | `You are not allowed to do this.` | The user lacks the **Access data sharing** capability |
| 400 | `Content-Type in request is missing or invalid` | The `Content-Type` names another media type, or lacks `version=2.1` |
| 400 | `Objects required` | The `objects` array is empty or missing |
| 404 | `Collection not found` | The collection does not exist, is stopped, or the user may not push to it |
| 404 | `Work not found` | The status `id` is unknown to the user who asks |

Every error also carries an `error_id`, the value of the `X-Request-ID` header of the answer. When the platform itself fails, the `title` is only `An unexpected error occurred`, and the error goes to the platform log as `TAXII request failed`, with that ID as its `request_id`. The partner sends you the `error_id`, and your platform engineer finds the full error with it.
::::

## What's next

- [Manage a feed](/opencti/use/ingest/#manage-a-feed): update, stop, export or delete the collection.
- [Import from a TAXII server](/opencti/use/ingest/taxii-feed/), to poll a collection instead.
- [Share with feeds and streams](/opencti/use/share/feeds/), to publish TAXII collections that others read.
