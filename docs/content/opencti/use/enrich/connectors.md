---
title: Enrich knowledge from external sources
description: Enrich OpenCTI entities and observables with data from external sources, on demand, in bulk, automatically on arrival or from a playbook.
sidebar_position: 1
doc_type: how-to
products:
  - opencti
personas: [soc-analyst, cti-analyst]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/enrichment/
---

To get outside context on an IP address, a domain or a hash in a few seconds, open it in OpenCTI and click **Enrichment**: each enrichment connector your platform runs looks the object up in its source, such as a reputation service, a sandbox or a WHOIS database, and writes what it finds onto the object. The result stays in the knowledge base for the whole team: a score, labels, external references, sightings or new relationships, each traced in the object's history. This page shows how to enrich one object or a whole list, and which way of running enrichment fits which source.

## Before you start

- Your role has the **Ask for knowledge enrichment** capability.
- An administrator has deployed at least one enrichment connector for the type of object you want to enrich. Each connector handles a set of types, its scope, and decides which markings it may send to its source. Deploying one is an administrator's task: see [Import with connectors](/opencti/use/ingest/connectors/) and the [XTM Hub catalog](https://hub.filigran.io/cybersecurity-solutions/open-cti-integrations).

## Enrich one object

:::steps

1. Open the entity or observable and click **Enrichment** in its header.
2. In the **Enrichment connectors** panel, find the connector to use. Its icon says whether it is active or disconnected.
3. Click **Refresh the knowledge using this connector**. OpenCTI sends the object to the connector, which queries its source and sends back what it found.
4. Follow the run in the list under the connector: each request shows its date, its status and the messages of the connector.

   ::screenshot{id=opencti-enrich-connectors-panel alt="The Enrichment connectors panel of an IP address, listing two connectors, each with its refresh button and the status of its last runs."}

:::

The panel says **No connectors for this type of entity** when no deployed connector covers the type. What a connector adds depends on the connector: read its description in the catalog.

:::example[Check the reputation of the Velvet Harbor C2 address]
A proxy alert shows a hotel front-desk workstation talking to `203.0.113.10`. OpenCTI knows the address as the command and control server of Velvet Harbor, and you want to know whether other organizations report it too. Your administrator runs the AbuseIPDB connector on demand only (`CONNECTOR_AUTO=false`), so that the daily quota of the AbuseIPDB plan goes to the addresses analysts ask about, and it never sends addresses marked above `TLP:AMBER`.

1. Open :menu[Observations > Observables] and open `203.0.113.10`.
2. Click **Enrichment**, then **Refresh the knowledge using this connector** on the **AbuseIPDB** line.
3. Wait for the run to show as complete under the connector, then close the panel.

The address now carries the abuse confidence score of AbuseIPDB as its score, a label for each abuse category reported, such as `Brute Force` or `Port Scan`, and a sighting for each country the reports came from. A high score and recent reports from several countries confirm that the address is live infrastructure: block it and open a case for the workstation.
:::

## Enrich many objects

Select the objects in a list, for example in :menu[Observations > Observables], then click the enrichment button in the toolbar and pick the connectors. The selection must hold one type of object. OpenCTI sends each object to the connectors in a [background task](/opencti/use/automate/background-tasks/).

## Choose how enrichment runs

| Mode | How | Good for | Watch out for |
| --- | --- | --- | --- |
| On demand | The **Enrichment** button or the list toolbar | Paid or rate-limited sources, objects under investigation | Nothing runs unless someone asks |
| Automatic | The connector runs with `auto` set to `true` and enriches every new or updated object in its scope. Administrators can narrow it with **Trigger filters** on the connector page | Free, fast sources you want applied everywhere | Quotas run out quickly, and the extra data takes disk space on large platforms |
| From a playbook | The **Enrich through connector** step of a [playbook](/opencti/use/automate/playbooks/) | Enrichment only when conditions hold, for example IP addresses from a given report | The playbook must filter early, as connectors fail on objects outside their scope |

:::note
A connector writes with the confidence level of its user. When its maximum confidence is lower than the confidence of the data in place, its updates are refused. See [Reliability and confidence](/opencti/concepts/reliability-confidence/).
:::

## Check that it works

Open the object again after the run completes. New relationships appear in its **Knowledge** tab, new external references and labels on its **Overview**, and the **History** tab lists the changes the connector made.

## What's next

- [Automate with playbooks](/opencti/use/automate/playbooks/), to enrich only the objects that need it.
- [Import with connectors](/opencti/use/ingest/connectors/), to see the connectors deployed on your platform and their status.
- [How OpenCTI infers new relationships](/opencti/use/enrich/inferences/), to derive new relationships from the ones you have.
