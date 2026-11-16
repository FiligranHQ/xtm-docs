---
title: Add the assets your simulations target
description: Let OpenAEV agents register your endpoints, describe your other assets by category, import hosts from a CSV file, and gather assets in asset groups by hand or by rule.
doc_type: how-to
products:
  - openaev
personas: [validation-engineer]
owner: docs-openaev
last_reviewed: 2026-11-07
migrated_from:
  - openaev:/usage/assets/
---

An asset is a system an inject acts on: a workstation, a server, a web application, a cloud resource, an AI agent. Hosts where an OpenAEV agent runs register themselves; you describe the others by hand or import them. Asset groups then gather the assets an inject should hit together, such as every front-desk workstation, either by hand or with a rule that keeps the group current. This page builds that inventory. How targets reach injects is explained in [How injects reach assets and people](/openaev/use/targets/).

::diagram{id=openaev-assets-sources}

## Before you start

- Your role has **Access assets** to see :menu[Assets] and :menu[Asset groups], **Manage assets** to create, import and change assets and groups, and **Delete assets** to delete them.
- To register endpoints with an agent, the agent is installed on them: see [Install the OpenAEV agent on your endpoints](/openaev/use/agents/). **Install simulation agents** in the top bar shows the commands to the roles that have **Install agent**.

## Let agents register your endpoints

When an OpenAEV agent starts on an endpoint, the endpoint appears in :menu[Assets] as a **Host**, named after its hostname, with its **Platform**. The agent then keeps its **Hostname**, **IP Addresses**, **MAC Addresses**, **Architecture** and **Seen IP address** up to date. An agent that reports a MAC address OpenAEV already knows updates that asset instead of creating another one. Addresses you add by hand stay; the agent only adds its own.

The **Status** column tells whether the asset can run injects that need an agent:

| Status | Meaning |
| --- | --- |
| **Active** | At least one of its agents contacted OpenAEV in the last hour |
| **Inactive** | Its agents have been silent for more than an hour: check the agent's service on the endpoint |
| **Agentless** | No agent runs on it: injects that run commands through an agent cannot act on it, while injects that work from outside, such as a scan, can |

The same host can run several agents, for example the OpenAEV agent as a service and as a user session, or an EDR used as executor. Its page lists each one under **Agents**: the account it runs as, its executor, its status, then its **Privilege**, **Deployment**, **Version** and **Last Seen**.

OpenAEV tells agents apart by host, executor, account, **Privilege** and **Deployment**. Installing the agent again with the same account and mode updates the existing agent; installing it with another account or mode adds one to the host. Agents of the Caldera executor are the exception for the host: OpenAEV matches them by hostname and IP address instead of MAC address.

::screenshot{id=openaev-assets-list alt="The Assets list, by name: the booking database server booking-db-01, then the front-desk workstations fd-barcelona-03, fd-madrid-02, fd-nice-04 and fd-paris-01, each a host."}

## Describe an asset by hand

Describe the assets no agent registers: a web application to scan, a cloud resource, a network device, an account, an AI agent to test.

:::steps
1. Open :menu[Assets] and click **Create**. The **Create a new asset** panel opens on the **Category** step.
2. Choose the category of the asset. It decides the fields of the next step:

   | Category | For | Its own fields |
   | --- | --- | --- |
   | **Host** | A physical or virtual machine, server or workstation | **Platform** and **Architecture** (required), **Hostname**, **IP Addresses**, **MAC Addresses**, **Internet-facing**, **End of Life** |
   | **Web application** | A website, web API or web service | **URL** (required), **Hostname**, **IP Addresses**, **Internet-facing** |
   | **Cloud resource** | A compute, storage, database, network or serverless resource | **Cloud provider** (required), **Native type**, **Region**, **Account ID**, **Resource ID / ARN** |
   | **Container / Workload** | A container, an image, a Kubernetes object, a serverless function | **Image** |
   | **Network device** | A router, switch, firewall or load balancer | **IP Addresses** (required), **Vendor**, **Model**, **Hostname** |
   | **Mobile device** | A smartphone or a tablet | **Platform** (iOS or Android), **OS version** |
   | **IoT / OT device** | A sensor, camera, PLC or SCADA system | **IP Addresses** (required), **Vendor**, **Model**, **Protocol** |
   | **Identity** | A user, service account, group or role | **Person**, **Identity provider**, **Principal / email** |
   | **SaaS application** | A software-as-a-service application or tenant | **Vendor** |
   | **AI target** | A language model, AI agent, MCP server or RAG pipeline to test | **Provider**, **Modality**, **Model**, **Endpoint URL**, **API token (optional)**, **System prompt (optional)** |
   | **Generic asset** | Anything that fits no other category yet | None |

3. Enter the **Name**, and optionally a **Description** and a **Subcategory**, such as **Server** or **Workstation** for a host.
4. Fill in the fields of the category. On a host, **Platform** starts at Linux and **Architecture** at x86_64: set them to the real ones, because injects that run commands pick their command by platform and architecture. Turn on **End of Life** for a host its vendor no longer supports: it changes nothing in how injects run, but a dashboard list widget can show it as a column, to find the hosts to replace among those a simulation got through.
5. Set the **Criticality**: **Very high**, **High**, **Medium**, **Low** or **Unknown**, the default. It ranks the assets in lists and filters, and asset groups can select by it.
6. Add **Tags**, then click **Create**.
:::

The asset appears in the list with its **Category**, its **Criticality** and, once injects have run on it, its **Posture score**: the share of its expectations that succeeded. A host created by hand shows as **Agentless** until an agent registers on it.

To change or delete an asset, open the menu at the end of its line and choose **Update** or **Delete**. Select several lines to delete them together.

## Import hosts from a CSV file

To create many hosts at once, click **Import** above the list and choose a CSV file. Each line becomes a **Host**, with the columns `asset_name`, `asset_description`, `asset_hostname`, `asset_ips`, `endpoint_platform`, `endpoint_arch`, `asset_mac_addresses`, `asset_tags` and `endpoint_is_eol`. IP and MAC addresses are JSON lists, and tags a JSON list of names and colors. The simplest way to get the format right is to export a few hosts first with **Export this list**, and edit that file.

## Gather assets in an asset group

An asset group is what most injects target: one inject on a group runs on each of its assets. A group takes its members in two ways, which add up: the assets you add by hand, and the assets that match its rule.

:::steps
1. Open :menu[Asset groups] and click **Create**. The **Create a new asset group** panel opens.
2. Enter the **Name**, and optionally a **Description** and **Tags**.
3. Under **Rule**, add the filters that select the assets: category, subcategory, criticality, cloud provider, exposure to the internet, platform, architecture, hostname or IP addresses. Leave the rule empty to choose every member by hand.
4. Click **Create**.
5. To add assets by hand, open the menu at the end of the group's line, choose **Manage assets**, and add them.
:::

Open the group to check its members: the list shows both kinds. The rule is evaluated each time the group is read, so a host that registers tomorrow and matches it joins the group, and the scenarios that target the group, without anyone editing them.

A new tenant already holds asset groups by platform and architecture, such as **All Windows x86_64** or **All Linux arm64**, which the default asset rules of OpenCTI scenarios use. See [Set default asset groups by scenario tag](/openaev/configure/asset-rules/).

::screenshot{id=openaev-asset-group-create alt="The Create a new asset group panel, with its name, description and tags, and the rule that adds every endpoint matching a filter."}

## Read an asset

Click an asset to open its page. Its header counts its **Agents**, **Findings**, **Injects played** and **Expectations tested**. The **Overview** tab gives its **Asset Information**, its network details, the **Asset groups** it belongs to, its agents, and the findings, expectations and injects that concern it. The **Statistics** tab draws its **Posture score over time** and **Injects played over time**: the asset that keeps failing is the one to look at.

:::example[Prepare the assets of the Velvet Harbor replay]
At Grand Meridian Hotels, the OpenAEV agent runs on the front-desk workstations of its hotels in France and Spain, from `fd-paris-01` to `fd-barcelona-03`, and on the booking database server `booking-db-01`: they appear as **Host** assets, **Active**. The validation engineer then adds by hand:

| Field | Booking portal | Shared night audit account |
| --- | --- | --- |
| Category | **Web application** | **Identity** |
| **Name** | `Booking portal` | `night.audit` |
| **Subcategory** | Website | User account |
| **URL** | `https://booking.grand-meridian.example` | |
| **Identity provider** | | `Active Directory` |
| **Principal / email** | | `night.audit@grand-meridian.example` |
| **Criticality** | **Very high** | **High** |
| **Internet-facing** | On | |
| **Tags** | `velvet-harbor` | `velvet-harbor` |

Then the asset group **Front desk workstations**, with the **Rule** hostname starts with `fd-`. When the new hotel in Lyon installs the agent on `fd-lyon-01` next month, the workstation joins the group, and the weekly Velvet Harbor simulation covers it.
:::

::screenshot{id=openaev-asset-detail alt="The page of fd-barcelona-03: one agent, no injects played yet, its description, criticality Medium and tag front-desk, its network details, the asset groups it falls in, Front desk workstations among them, and its Active OpenAEV agent."}

## What's next

- [Organize the people your injects target](/openaev/use/targets/people/): the other half of the targets.
- [Set default asset groups by scenario tag](/openaev/configure/asset-rules/): target groups without picking them on each inject.
- [Build a scenario that replays an attack](/openaev/use/scenarios/build/): aim injects at your assets.
