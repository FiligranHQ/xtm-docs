---
title: Connect OpenCTI with a Live Stream
description: Feed OpenCRQ the threat actors, campaigns and techniques your OpenCTI platform knows, through a Live Stream that keeps them current as analysts change them.
sidebar_position: 1
doc_type: how-to
products: [opencrq, opencti]
personas: [admin, risk-manager, cti-lead]
owner: docs-opencrq
last_reviewed: 2026-11-04
---

OpenCRQ prices a risk scenario from the threat actors that target your sector and your country and pursue the scenario's outcome. The OpenCTI connector subscribes to an OpenCTI Live Stream and keeps those threats current: a threat created, edited, renamed, merged or deleted in OpenCTI changes the same way in OpenCRQ, and the scenarios it reaches are priced again. This page sets up the stream in OpenCTI, binds it to your tenant in OpenCRQ, and checks what arrives. To see where threats weigh in the price, read [How OpenCRQ quantifies risk](/opencrq/concepts/how-risk-is-quantified/).

::diagram{id=opencrq-connect-opencti}

## What the connector reads

The connector reads four kinds of object from the stream:

- **Threats**: intrusion sets, threat actors (groups and individuals) and campaigns. OpenCRQ stores a threat actor group and an individual threat actor as one threat type.
- **`uses` and `targets` relationships** of those threats: the techniques they use, and the sectors, countries and CVEs they target. These are the claims OpenCRQ scores.
- **`attributed-to` relationships** between two threats, such as a campaign attributed to an intrusion set. One toward anything else is ignored.
- **The identities that authored those relationships**, so that each claim is attributed to who made it.

The connector asks OpenCTI for the relationships its rules engine inferred too. Whether a claim counts is decided by the confidence threshold of the tenant, in :menu[Settings > Engine > Threat Relevance]: see [Set how OpenCRQ weighs threats, controls and assets](/opencrq/configure/quantification-engine/).

## Before you start

- In OpenCRQ, your role has the **Manage Security** capability. Configuring the connector, turning it on or off, reconnecting it and resetting it all require it.
- In OpenCTI, you have the **Manage data sharing** capability, to create the Live Stream.
- In OpenCTI, a user for OpenCRQ, with the **Access data sharing** capability and read access to every marking definition applied to the threats, their relationships and the identities that authored them. OpenCTI leaves out of the stream a relationship whose source or target this user cannot read, and OpenCRQ cannot tell that gap from a narrow filter.
- The deployment of OpenCRQ carries the OpenCTI URL and that user's token for your tenant. Until it does, the connector page shows **Unavailable** and says **Add this connector to your deployment configuration to use it.** See [Give OpenCRQ the URL and the token](#give-opencrq-the-url-and-the-token).

## Create the Live Stream in OpenCTI

OpenCRQ requests the stream without the objects each event depends on, so OpenCTI sends what the filter admits and nothing else. The filter must therefore admit the relationships, not only the threats. For each threat type it admits (`Intrusion-Set`, `Threat-Actor-Group`, `Threat-Actor-Individual`, `Campaign`), it admits:

- `uses` from that type to an `Attack-Pattern`,
- `targets` from that type to a `Location`, a `Sector` and a `Vulnerability`,
- and, when it admits two threat types or more, `attributed-to` from a campaign to an intrusion set or a threat actor, and from an intrusion set to a threat actor.

This filter is a group of alternatives, each one a set of conditions that all apply:

```json
{
  "mode": "or",
  "filters": [],
  "filterGroups": [
    {"mode": "and", "filterGroups": [], "filters": [
      {"key": ["entity_type"], "operator": "eq", "mode": "or", "values": ["Intrusion-Set", "Threat-Actor-Group", "Threat-Actor-Individual", "Campaign"]}
    ]},
    {"mode": "and", "filterGroups": [], "filters": [
      {"key": ["entity_type"], "operator": "eq", "mode": "or", "values": ["targets"]},
      {"key": ["fromTypes"], "operator": "eq", "mode": "or", "values": ["Intrusion-Set", "Threat-Actor-Group", "Threat-Actor-Individual", "Campaign"]},
      {"key": ["toTypes"], "operator": "eq", "mode": "or", "values": ["Location", "Sector", "Vulnerability"]}
    ]},
    {"mode": "and", "filterGroups": [], "filters": [
      {"key": ["entity_type"], "operator": "eq", "mode": "or", "values": ["uses"]},
      {"key": ["fromTypes"], "operator": "eq", "mode": "or", "values": ["Intrusion-Set", "Threat-Actor-Group", "Threat-Actor-Individual", "Campaign"]},
      {"key": ["toTypes"], "operator": "eq", "mode": "or", "values": ["Attack-Pattern"]}
    ]},
    {"mode": "and", "filterGroups": [], "filters": [
      {"key": ["entity_type"], "operator": "eq", "mode": "or", "values": ["attributed-to"]},
      {"key": ["fromTypes"], "operator": "eq", "mode": "or", "values": ["Campaign", "Intrusion-Set"]},
      {"key": ["toTypes"], "operator": "eq", "mode": "or", "values": ["Intrusion-Set", "Threat-Actor-Group", "Threat-Actor-Individual"]}
    ]}
  ]
}
```

To bring intrusion sets alone, keep only `Intrusion-Set` in the three lists of threat types and drop the `attributed-to` group. A relationship is named by `entity_type` and narrowed by `fromTypes` and `toTypes`: the stream filter has no `relationship_type` key.

The filter editor of the OpenCTI interface does not build groups inside a group, so create the stream with the GraphQL API of OpenCTI, as a user with **Manage data sharing**. The `filters` field takes the filter above as a string; `stream_live: true` creates the stream already started:

```graphql
mutation CreateRiskQuantificationStream($input: StreamCollectionAddInput!) {
  streamCollectionAdd(input: $input) { id name stream_live }
}
```

The stream then shows in **Data > Data sharing > Live streams** of OpenCTI, with its **Stream ID**. Leave out origin filters: OpenCTI does not replay the history of a stream that has them, so the threats already in OpenCTI would never reach OpenCRQ. For the rest of what a stream shares, see [Share knowledge with your security tools](/opencti/use/share/feeds/#create-a-live-stream).

## Give OpenCRQ the URL and the token

The credentials are not stored in OpenCRQ's database: the deployment sets them per tenant, as two environment variables keyed by the tenant's id, and the connector reads that tenant's entry only. The tenant's id is the first segment of the address of OpenCRQ once you are signed in to that tenant.

| Variable | What it holds |
| --- | --- |
| `CONNECTORS__OPENCTI__<tenant-uuid>__URL` | The base URL of the OpenCTI platform |
| `CONNECTORS__OPENCTI__<tenant-uuid>__TOKEN` | The API token of the OpenCTI user for OpenCRQ. Pass it through a secret store |

After a restart with both set, the **OpenCTI** card moves to the **Deployed** tab of :menu[Integrations]. See the [configuration reference](/reference/opencrq/configuration/#connectors) for every connector's variables.

## Bind the stream to your tenant

The connector writes the threats it receives as a service account of the tenant. Create one first in :menu[Settings > Security > Service accounts] with **Create**: see [Let integrations sign in with service accounts](/opencrq/administer/service-accounts/).

:::steps
1. Open :menu[Integrations], stay on the **Deployed** tab, and click the **OpenCTI** card, under **Threat intelligence**.
2. Click **Configure**. The **Configure integration** panel lists the Live Streams the token can read.
3. Choose the stream in **Live Stream**. OpenCRQ judges its filter before you save: **Delivers threat actors** when it admits everything the connector reads. The filter shows below, in a **Readable** and a **Raw** tab.
4. Choose the account in **Service account**.
5. Click **Save**. The connector turns on and reads the stream from its beginning.

   ::screenshot{id=opencrq-opencti-configure alt="The Configure integration panel of the OpenCTI connector: the Enabled switch, the Live Stream Threats for risk quantification with the notice that its filter admits threat actors, and its filter in readable form, one group of conditions per kind of object."}
:::

A judgment other than **Delivers threat actors** is advice, not a block: you can still save.

| Judgment | What it means | What to do |
| --- | --- | --- |
| **Incompatible filter** | The filter admits threats but leaves out at least one relationship the connector reads. The notice names which ones, and what will not arrive | Widen the filter in OpenCTI |
| **No threat actors** | The filter admits no threat type, so nothing would arrive | Choose another stream, or widen this one |
| **Stopped in OpenCTI** | The stream refuses connections | Start it in OpenCTI |
| **Cannot be judged** | The filter uses conditions OpenCRQ does not evaluate | Read the filter shown and decide |

A stream with origin filters also shows **This stream replays no history**: only what OpenCTI publishes from then on arrives.

Once the connector is configured, the panel shows an **Enabled** switch. Turning it off stops reading the stream and keeps its position.

:::example[Feed Grand Meridian the threats Example CERT tracks]
Grand Meridian Hotels prices the risk that Velvet Harbor and Ember Tide put on its hotels. Its OpenCTI platform, at `https://opencti.example.com`, holds what Example CERT shares about both intrusion sets.

In OpenCTI, the administrator creates the stream with the mutation above and these values: **Name** `Threats for risk quantification`, **Description** `Intrusion sets, threat actors and campaigns with what they use and target, read by OpenCRQ for Grand Meridian Hotels.`, not public, started, and the four-group filter of this page. A user `OpenCRQ reader` with **Access data sharing**, cleared for every marking Example CERT uses, holds the token.

The platform engineer sets the tenant's entry, `abd0465d-51cd-5234-85b8-cb1cc781124c` being Grand Meridian's tenant id:

```yaml
services:
  opencrq:
    environment:
      - CONNECTORS__OPENCTI__abd0465d-51cd-5234-85b8-cb1cc781124c__URL=https://opencti.example.com
      - CONNECTORS__OPENCTI__abd0465d-51cd-5234-85b8-cb1cc781124c__TOKEN=${GRAND_MERIDIAN_OPENCTI_TOKEN}
```

In OpenCRQ, the administrator creates the service account `OpenCTI connector`, then in **Configure integration** picks **Live Stream** `Threats for risk quantification`, judged **Delivers threat actors**, and **Service account** `OpenCTI connector`, and saves.

The stream replays its history: Velvet Harbor arrives with its 10 techniques and Ember Tide with its 7, each with the sectors, countries and CVE it targets. OpenCRQ admits Velvet Harbor to the data breach scenarios, and Ember Tide and Velvet Harbor to the ransomware on the booking platform, and prices them. When Example CERT reports Operation Low Season, a campaign of Ember Tide against hotel chains in France and Spain, the campaign reaches OpenCRQ as OpenCTI publishes it, and the ransomware scenario is priced again.
:::

## What comes in, and when

- **The first connection replays the stream's history**, from its first event, then follows it live. While it replays, the **Stream** tab says **This stream is still replaying its history** and shows a **Backfill** bar between the date of the first event and the date the replay will reach. The connector imports what it receives in batches, so OpenCRQ's threats lag behind the stream until the bar is full.
- **Then each change arrives as OpenCTI publishes it.** The **Live tail** field reads **Reached**, and keeps reading it.
- **A dropped connection** is retried, waiting longer between each attempt, and resumes from the last event the connector stored.
- **A rejected token** turns the connector off: the state reads **Failing** and the connection **Authentication failed**. Fix the token in the deployment, then turn **Enabled** back on in **Configure integration**.
- **Saving the configuration again**, even with the same stream, reads the stream from its beginning once more. Nothing already imported is removed.

The threats show in :menu[Threats], where you can filter them by source. Which of them a scenario counts depends on the sectors and countries of its risk scope and on its outcome: see [Follow the threats that target your scopes](/opencrq/use/threats/threat-center/).

## Check that it works

::screenshot{id=opencrq-integrations alt="The Integrations page: the figures of the connector pipeline, with one configured connector, the stream catch-up at 100% and 88 records ingested, above the Deployed connectors grouped by domain."}

1. In :menu[Integrations], the **OpenCTI** card reads **Syncing** while the history replays, then **Healthy**. The **Stream catch-up** figure at the top reaches 100%.
2. On the connector's **Stream** tab, **Connection** reads **Connected** and **Live tail** reads **Reached**. **Read from the stream** counts the objects received, **Held** the objects the connector keeps.

   ::screenshot{id=opencrq-opencti-stream alt="The Stream tab of the Healthy OpenCTI connector: connected, live tail reached, the stream filter judged as delivering threat actors, and the backfill bar full, with 6.4K objects read from the stream and 87 held."}

3. The **Logs** tab shows what the connector did with the stream.
4. :menu[Threats] lists the threats of the stream.

If nothing arrives, open the connector page: a banner names the problem, such as **This stream cannot supply the connector** with the relationships its filter leaves out, **This stream is stopped in OpenCTI**, or **This Live Stream no longer exists in OpenCTI** when the token can no longer see it. When the stream is right and threats are still missing, check the markings the OpenCTI user can read.

## Retire the default threats

A tenant starts with **OpenCRQ Default Threats**, a connector under **Threat intelligence** that loads threat groups and campaigns derived from MITRE ATT&CK, so that scenarios have threats before any feed is connected. It stays on when you connect OpenCTI. Once the stream reaches its live tail, the OpenCTI connector's **Overview** says **Your OpenCTI stream has caught up, and OpenCRQ Default Threats is still enabled.**: the default threats are listed beside yours, and a group both know counts once in a simulation.

When your stream covers the threats you care about:

1. Click **Review Default Threats** on that notice, then **Configure**.
2. Turn the switch off. The confirmation says how many of its threats leave the tenant and how many from your other sources stay.
3. Click **Disable**. Your risk scenarios are simulated again without the default threats; past simulations keep the names of the threats they used.

Turning the switch back on loads the default threats again. The platform reads them from OpenCRQ's dataset repository: on a platform without internet access, `CONNECTORS__DEFAULT_THREATS__DATASET_URL` points it to a mirror of that folder, and an empty value turns the connector off for every tenant. See the [configuration reference](/reference/opencrq/configuration/#connectors).

## Reconnect or start over

Both actions require the **Manage Security** capability.

- **Reconnect**, in the connector's header, opens the stream again at its live edge. During a replay, it asks first: the events between where the replay got to and now are not imported.
- **Reset state**, in the **More actions** menu, reads the stream again from its beginning. Nothing already imported is removed.

## What's next

- [Follow the threats that target your scopes](/opencrq/use/threats/threat-center/)
- [Build and price a risk scenario](/opencrq/use/quantify/risk-scenarios/)
- [Import OpenAEV results](/opencrq/use/sources/openaev/), to measure how your controls hold against these threats
- [OpenCRQ connectors](/reference/opencrq/connectors/) and the [configuration reference](/reference/opencrq/configuration/#connectors)
