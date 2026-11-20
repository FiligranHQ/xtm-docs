---
title: Import assets from Microsoft Defender
description: Fill OpenCRQ's attack surface with the devices, accounts and software vulnerabilities Microsoft Defender sees, rated for criticality and exposure, and kept in step every six hours.
sidebar_position: 3
doc_type: how-to
products: [opencrq]
personas: [admin, platform-engineer, risk-manager]
owner: docs-opencrq
last_reviewed: 2026-11-04
---

The attack run of OpenCRQ only uses the techniques your estate exposes, so the price of a scenario is as good as the inventory behind it. The Microsoft Defender connector imports that inventory from Microsoft Defender for Endpoint, Microsoft Entra ID and Intune, through the Microsoft Graph API: your devices with the CVEs Defender finds on them, the accounts of your directory, and who uses which workstation. It rates each one's criticality and exposure from what Defender knows, and keeps them in step on a schedule. This page registers the application in Microsoft Entra ID, gives its credentials to OpenCRQ and turns the connector on. For how assets weigh in the price, read [How OpenCRQ quantifies risk](/opencrq/concepts/how-risk-is-quantified/).

## What the connector imports

The connector writes **supporting assets** and **findings**. Each run reads everything again, not only what changed.

**Devices.** Every device in Defender's retention window, with or without a vulnerability, since Defender keeps matching new CVEs against the last software inventory of a device that went offline. Each comes with its name, its IP and MAC addresses from its latest snapshot, its operating system release and full version, its vendor, model and subtype (switch, router, smartphone...), which refines its subcategory. A device with no name is kept and shown under a name built from its vendor, model and address.

- **Exposure** follows Defender's internet-facing flag: internet facing, internal, or unknown when Defender does not say.
- **Criticality** is the highest that applies: very high for a domain controller, a certificate services, federation services or Entra Connect server; high for a hypervisor or a backup server; or the asset value set in Defender (very high, high or low). The role names no rule rates are written in the run's log.

**Software vulnerabilities.** One finding per device and CVE, with the CVSS score Defender gives, whether an exploit is available, and the software that carries it. A CVE Defender stops reporting on a device is dismissed as **No longer reported by Microsoft Defender**. CVE details then come from the [public CVE sources](/opencrq/use/sources/vulnerability-sources/).

**Accounts.** The accounts Defender's identity inventory and logon events show over the last 30 days. Their criticality is the highest of:

- their directory roles, active or eligible: very high for the roles that can grant any other, such as Global Administrator, Privileged Role Administrator or User Administrator; high for Global Reader, Security Operator, Security Reader and any other administrator role,
- the privileged groups their logon tokens carry: very high for Domain, Enterprise and Schema Admins, high for Group Policy Creator Owners, medium for Cert Publishers,
- the criticality level Defender gives the account.

Computer accounts, whose name ends in `$`, and service identities such as SYSTEM are not imported. Neither is a disabled account that carries no display name, user principal name, account name or SID: each run removes the ones a previous run left, unless someone added something to them by hand.

**Primary users.** The person each workstation is assigned to, from Intune first, then from the registered owner in Entra ID. Without these sources, the run succeeds and no workstation gets a primary user.

A machine that OpenAEV also reports becomes one asset: see [Import OpenAEV results](/opencrq/use/sources/openaev/).

## Before you start

- In OpenCRQ, your role has the **Manage Security** capability, and a service account exists for the connector: see [Let integrations sign in with service accounts](/opencrq/administer/service-accounts/).
- In Microsoft Entra ID, an app registration with a client secret, and these Microsoft Graph **application** permissions, each with admin consent:

  | Permission | Required | What it reads | Without it |
  | --- | --- | --- | --- |
  | `ThreatHunting.Read.All` | Yes | Advanced Hunting: devices, vulnerabilities, accounts and logon events | The run fails |
  | `Device.Read.All` | No | Entra devices and their registered owners | No workstation gets a primary user from Entra ID |
  | `DeviceManagementManagedDevices.Read.All` | No | Intune managed devices and their primary user | Primary users come from Entra ID alone |

## Give OpenCRQ the credentials

The credentials are not stored in OpenCRQ's database: the deployment sets them per tenant, keyed by the tenant's id, the first segment of the address of OpenCRQ once you are signed in to that tenant. The connector turns available for a tenant once all three are set:

| Variable | What it holds |
| --- | --- |
| `CONNECTORS__MICROSOFT_GRAPH__<tenant-uuid>__AZURE_TENANT_ID` | The ID of your Microsoft Entra tenant |
| `CONNECTORS__MICROSOFT_GRAPH__<tenant-uuid>__CLIENT_ID` | The application (client) ID of the app registration |
| `CONNECTORS__MICROSOFT_GRAPH__<tenant-uuid>__CLIENT_SECRET` | Its client secret. Pass it through a secret store |

Until then, the **Microsoft Defender** card stays on the **Available** tab of :menu[Integrations], and its page reads **Unavailable**:

::screenshot{id=opencrq-defender-connector alt="The Microsoft Defender connector page before its credentials are set: Unavailable, the notice to add it to the deployment configuration, the domain Vulnerability, a scheduled pull whose scope is supporting assets and findings, and a schedule of every 6 hours with no run yet."}

See the [configuration reference](/reference/opencrq/configuration/#connectors).

## Turn the connector on

:::steps
1. Open :menu[Integrations], stay on the **Deployed** tab, and click the **Microsoft Defender** card, under **Vulnerability**. Its state reads **Not configured**.
2. Click **Configure**.
3. Choose the account in **Service account**. The connector writes the assets and findings as this account.
4. Under **Schedule**, keep **Every 6 hours** in **Frequency**, or choose another: from **Every hour** to **Every week**, or **Manual only**. A run reads the whole estate, which takes minutes on a large tenant: run it more often only if the estate changes faster.
5. Click **Save**, then **Sync now** to run it without waiting for the schedule.
:::

A run stops after 30 minutes, and a failed run is tried again, up to 3 attempts. The **Works** tab lists the runs in progress and the completed ones. A primary user source that refuses for lack of consent, or does not answer, such as Intune on a tenant without its license, does not fail the run: the primary users it named stay as they were.

:::example[Import Grand Meridian's hotels from Defender]
Grand Meridian Hotels manages the workstations of its front desks and the servers of its booking platform in Microsoft Defender for Endpoint, and assigns each front-desk workstation in Intune. The risk manager wants the booking servers that face the internet, and the CVEs on them, in the attack surface of Grand Meridian France.

In Microsoft Entra ID, the administrator registers the application `OpenCRQ Defender connector`, creates a client secret, and grants `ThreatHunting.Read.All`, `Device.Read.All` and `DeviceManagementManagedDevices.Read.All` with admin consent. The platform engineer sets the entry of Grand Meridian's tenant, `abd0465d-51cd-5234-85b8-cb1cc781124c`:

```yaml
services:
  opencrq:
    environment:
      - CONNECTORS__MICROSOFT_GRAPH__abd0465d-51cd-5234-85b8-cb1cc781124c__AZURE_TENANT_ID=${GRAND_MERIDIAN_ENTRA_TENANT_ID}
      - CONNECTORS__MICROSOFT_GRAPH__abd0465d-51cd-5234-85b8-cb1cc781124c__CLIENT_ID=${GRAND_MERIDIAN_DEFENDER_CLIENT_ID}
      - CONNECTORS__MICROSOFT_GRAPH__abd0465d-51cd-5234-85b8-cb1cc781124c__CLIENT_SECRET=${GRAND_MERIDIAN_DEFENDER_CLIENT_SECRET}
```

In OpenCRQ, the administrator creates the service account `Microsoft Defender connector`, then in **Configure integration** chooses it, keeps **Every 6 hours**, saves and clicks **Sync now**.

The booking servers arrive as internet facing when Defender flags them so, each with the CVEs Defender finds on it, and the domain controllers of the chain as very high criticality. The front-desk accounts arrive with the workstation each one uses. Every 6 hours, a CVE patched on a server is dismissed and the scenarios that reach it are priced again.
:::

## Check that it works

1. On the connector page, the state reads **Healthy**, and **Last run**, **Status**, **Duration** and **Processed** describe the run.
2. :menu[Assets] lists the devices and accounts, and :menu[Findings] their CVEs.
3. The **Logs** tab gives, at the end of the accounts, how many disabled accounts with no identifying data were skipped and how many assets were removed. It keeps the last runs only.

If the run fails at once, check the three variables and the consent for `ThreatHunting.Read.All`.

## What's next

- [Build the attack surface from your assets](/opencrq/use/attack-surface/assets/)
- [Review the findings on your assets](/opencrq/use/attack-surface/findings/)
- [How CVE sources weigh your vulnerabilities](/opencrq/use/sources/vulnerability-sources/)
- [OpenCRQ connectors](/reference/opencrq/connectors/) and the [configuration reference](/reference/opencrq/configuration/#connectors)
