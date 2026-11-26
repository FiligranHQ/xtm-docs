---
title: Import from another OpenCTI instance
description: Consume the live stream of another OpenCTI instance, so that its knowledge, its updates and optionally its deletions reach your instance in real time.
sidebar_position: 2
doc_type: how-to
products:
  - opencti
personas: [cti-lead, cti-analyst, admin]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/import/internal-streams/
---

An OpenCTI stream connects your instance to a live stream of another OpenCTI instance: every creation, update and, if you choose, deletion published on the remote stream is replayed on yours, as it happens. Use it to receive intelligence from a partner or a parent organization, or to replicate one instance to another. For how imported data reaches the knowledge base, read [How intelligence gets into OpenCTI](/opencti/use/ingest/).

## Before you start

- Your role has the **Manage ingestion** capability.
- You have the URL of the remote platform and, unless its stream is public, a token from it. The remote user behind the token needs the **Access data sharing** capability and access to the live stream. An administrator of the remote platform shares it in [Share with feeds and streams](/opencti/use/share/feeds/).
- To let the form create a service account, an administrator has set **Default group for Service accounts** in :menu[Settings > Security > Policies]. Otherwise, create the user that will write the data first: see [One account and one author per source](/opencti/use/ingest/#one-account-and-one-author-per-source).

## Create the stream

Open the creation form:

::::only{opencti=">=7.260811"}
Open :menu[Integrations], select the **Available** tab, and click **Create** on the **OpenCTI Stream** card.
::::

::::only{opencti="<7.260811"}
Open :menu[Data > Ingestion > OpenCTI Streams] and click the create button.
::::

Then fill it in:

:::steps

1. Enter a **Name** that says where the data comes from.
2. Enter the **Remote OpenCTI URL**: the address of the remote platform, without any path, such as `https://cti.partner.example`.
3. Enter the **Remote OpenCTI token**. Leave it empty for a public stream.
4. Click **Validate**. The platform connects to the remote one and lists the live streams the token can read.
5. Choose the stream in **Remote OpenCTI stream ID**.
6. Keep **Automatically create a service account** on and set its confidence level, or turn it off and choose an existing account in **Service account responsible for data creation**.
7. Check **Starting synchronization (empty = from start)**. It defaults to the start of today, so only events from today on are replayed. Set an earlier date to replay more, or clear it to import everything the stream holds.
8. Keep **Take deletions into account** on, its default, so that objects deleted on the remote platform are deleted on yours too. Turn it off to keep them.
9. Turn on **Verify SSL certificate** to check the certificate of the remote platform. It is off by default.
10. Click **Verify**. Once the connection is verified, click **Create**.
11. Open the actions menu of the new stream and choose **Start**.

:::

:::example[Receive the hospitality stream of the national CERT]
Example CERT runs its own OpenCTI instance and shares a live stream named "Hospitality sector" with its members. It gave your team a token of a user who can read that stream.

- **Name**: `Example CERT - Hospitality sector`
- **Remote OpenCTI URL**: `https://cti.example-cert.example`, without `/graphql` or any other path
- **Remote OpenCTI token**: the token Example CERT gave you
- **Validate**, then **Remote OpenCTI stream ID**: `Hospitality sector`
- **Automatically create a service account**: on, with a confidence level of 75, as Example CERT is a reliable source
- **Starting synchronization (empty = from start)**: `2026-01-01 12:00 am`, to receive the whole Velvet Harbor campaign without the older history
- **Take deletions into account**: on, so an indicator Example CERT deletes as a false positive disappears from your platform too
- **Verify SSL certificate**: on

Once started, the stream replays the events since 1 January 2026: the intrusion set Velvet Harbor, its malware and indicators, and the report "Velvet Harbor targets hotel booking systems" arrive with their relationships, then each change Example CERT makes reaches your platform within seconds.
:::

::screenshot{id=opencti-ingest-streams-create alt="The Create an OpenCTI Stream form: the remote OpenCTI URL and token with the Validate button, the service account and its confidence level, the synchronization start date, and the Verify and Create buttons."}

## Advanced options

The **Advanced options** section of the form holds two settings meant for specific cases:

- **Avoid dependencies resolution** imports the objects of the stream without the objects they refer to. By default, a malware shared on the stream comes with its relationships and their targets. References such as the author are still resolved.
- **Use perfect synchronization** makes the remote platform the only source of truth: an object it sends overwrites the local one. Use it only to replicate one platform to another.

:::note
**Take deletions into account** does not delete an object that another source also imported on your platform.
:::

::::only{opencti=">=7.261030"}
:::tip[Keep processing statuses]
Imported entities lose their processing status, because the remote status does not exist on your platform. To keep it, give your workflow statuses the same names as the remote ones, then, on your platform, turn on the **Entity status sync** switch in the **Workflow** card of each entity type you receive, in :menu[Settings > Customization > Entity types]. The remote platform needs no setting, but it must run version 7.261030 or later: earlier versions do not send status names in their streams. See [Customize entity types](/opencti/configure/entities/).
:::
::::

## Check that it works

::::only{opencti=">=7.260811"}
Open the stream from the **Deployed** tab of :menu[Integrations]. Its page shows its configuration and the date of the last event it processed. New entities appear with the stream's service account as creator.

:::only{opencti=">=7.261030"}
The **Logs** tab shows each connection to the remote platform.
:::
::::

::::only{opencti="<7.260811"}
In :menu[Data > Ingestion > OpenCTI Streams], the stream shows as running with the date of the last event it processed. New entities appear with the stream's service account as creator.
::::

If nothing arrives, check that the remote user still has access to the stream and that your platform can reach the remote URL. On the remote platform, your connection shows in the **Consumers** column of its live streams, with how far behind it is: its administrator can tell you whether your platform keeps up, as [Share knowledge with your security tools](/opencti/use/share/feeds/#keep-consumers-in-sync) explains.

## What's next

- [Manage a feed](/opencti/use/ingest/#manage-a-feed): update, stop, export or delete the stream.
- [Share with feeds and streams](/opencti/use/share/feeds/), to publish your own live streams.
- [Import from a TAXII server](/opencti/use/ingest/taxii-feed/), to poll a collection of another platform instead of following its stream.
