---
title: Review the findings on your assets
description: See the vulnerabilities and failed tests your integrations report on your assets, read how each one is scored, and set aside the ones you accept or that are not true.
sidebar_position: 2
doc_type: how-to
products: [opencrq, openaev]
personas: [risk-manager, validation-engineer]
owner: docs-opencrq
last_reviewed: 2026-11-04
---

A **finding** is a flaw an integration reports on one of your supporting assets: a known vulnerability (a CVE) on a server, or an attack technique that an OpenAEV simulation ran on a workstation without a security tool preventing or detecting it. OpenCRQ scores each vulnerability from 0 to 100, from how reachable it is, how actively it is exploited and what it would damage, and the attack run uses the open ones as doors into your estate. This page reads the findings, explains their score, and sets aside the ones you accept or that are not true. Read [Build the attack surface from your assets](/opencrq/use/attack-surface/assets/) first if your estate is not declared yet.

## Before you start

- You are a member of the tenant: every member can read findings, attach a CVE and set one aside. A service account cannot change a finding's state.
- At least one integration brings findings in, connected by an administrator with **Manage Security**:
  - [OpenAEV](/opencrq/use/sources/openaev/): the CVEs it found on each asset, and the result of each simulated technique, per security platform.
  - [Microsoft Defender](/opencrq/use/sources/microsoft-defender/): the CVEs on each device.
- For the scores, the [CVE sources](/opencrq/use/sources/vulnerability-sources/) give each CVE its CVSS vector (NIST), its exploitation probability (FIRST EPSS) and whether it is known to be exploited (CISA KEV). OpenCTI adds the threats that cite a CVE.

## Where findings come from

| Kind | **Type** in the list | From | Scored |
| --- | --- | --- | --- |
| A CVE on an asset | **Vulnerability** | OpenAEV, Microsoft Defender, or attached by hand | Yes, from 0 to 100 |
| A technique a simulation ran on an asset, not stopped | **Control Response: Detection**, **Control Response: Prevention** | OpenAEV | No: it measures your controls instead |

A finding is one flaw on one asset. When two integrations report the same CVE on the same asset, it stays one finding with two sources. A technique finding has no score and no history: its verdict counts in the success rate of the controls that should have stopped it. See [Set how strong your controls are and read how they held](/opencrq/use/defenses/controls/).

## Read the findings

:::steps
1. Go to :menu[Attack surface > Findings]. The cards count the **Findings**, the **Impacted assets**, those **New, last 7 days**, and the findings in each band: **Critical**, **High**, **Medium**, **Low**. **Pending vector** counts the vulnerabilities whose CVE has no CVSS vector yet: they have no score and sit in no band.

   ::screenshot{id=opencrq-findings-list alt="The All findings tab of Grand Meridian Hotels: one applicable finding, CVE-2024-3400 on the head-office VPN gateway, counted under Pending vector rather than in a band, its score pending."}

2. Stay on **All findings** to see one line per asset and flaw, sorted by **Score**, the worst first. **Search findings...** keeps the findings whose CVE identifier or title, technique identifier or name, or asset name holds what you type, such as `CVE-2024-3400` or `VPN`; when none does, the list reads **No finding matches this search.** **Filter by state** shows the **Applicable** findings by default; choose **Set aside, no longer reported or not exposed** or **All states** to see the others. **Filter by source** keeps one integration, or **Set by user** for the CVEs attached by hand.
3. Open **Unique findings** to see one line per flaw across the estate: a CVE with its **Worst finding**, its **Affected assets** and its **KEV / EPSS** reading. Its **Score** is the worst asset's score, raised when the CVE reaches other assets at a close level, and capped at 100. **Search findings...** works here too, on the CVE or the technique only. Select a CVE to open its page across the estate, with the tabs **Overview**, **Assets**, **Attack**, **Remediation** and **Events**.
4. Select a finding to open it. A vulnerability opens on **Overview**, with the tabs **Score**, **Attack**, **Controls**, **Remediation**, **Activity** and **Events**.
:::

## See the CVE in OpenCTI and OpenAEV

When the tenant's OpenCTI or OpenAEV connector is set, OpenCRQ shows where the same CVE lives in those products:

- In **Unique findings**, a product mark beside a CVE's identifier, **Open in OpenCTI** or **Open in OpenAEV**, opens the vulnerability there.
- On the CVE's page, an **In OpenCTI** or **In OpenAEV** card, above the tabs, links to the vulnerability there. OpenCTI finds it by its name, the CVE identifier. The **In OpenAEV** card adds the facts OpenAEV returns, such as its **Findings**, **Affected assets**, **Simulations** or **Last simulation**. A card reads **Not found in OpenCTI.** or **Not found in OpenAEV.** when that product does not hold the CVE.

  ::screenshot{id=opencrq-findings-cve-counterparts alt="The page of CVE-2024-3400 across the estate of Grand Meridian Hotels, with the In OpenCTI card above the tabs linking to the Vulnerability CVE-2024-3400 in OpenCTI."}

When a connector is not set, the CVE's page offers **Connect OpenCTI** or **Connect OpenAEV** instead, which opens the connector's settings. Without **Manage Security**, the button is disabled, and its hint says to ask an administrator. An administrator sets the connectors: see [Connect OpenCTI with a Live Stream](/opencrq/use/sources/opencti/) and [Import OpenAEV results](/opencrq/use/sources/openaev/).

## Read how a finding weighs

The **Score** tab breaks the score in two parts, each out of 100, and the score is their geometric mean.

- **Exploitability** is how likely the flaw gets used. It averages two factors and multiplies the result by the controls the attack meets on the way:
  - **Reachability**: the asset's exposure (internet-facing weighs 1, internal 0.4, as an administrator with **Manage Security** sets them under :menu[Settings > Engine > CRQ Indicators]) times how the CVSS vector says the flaw is reached.
  - **Threat intelligence**: the strongest signal wins. A listing in CISA KEV sets it at 0.9; a threat from OpenCTI that cites the CVE and is active, observed in the last 90 days, sets 0.35; otherwise the EPSS probability, never below 0.05. Two independent signals lift it a third of the way toward 1.
  - **Controls**: the posture of the controls that prevent or detect the techniques exploiting the flaw. The weakest path decides.
- **Impact** is what the flaw would damage: the asset's criticality times the CVSS impact, reduced by the controls that stand on the impact techniques.

The bands follow the score: **Critical** from 90, **High** from 70, **Medium** from 40, **Low** below. They are not the CVSS severity: a critical CVSS on an internal, low-criticality asset behind strong controls scores lower than a high one on the internet-facing booking site.

The finding score helps you choose what to fix first; it is not the price. The attack run does not read the score: it reads each applicable vulnerability as an opening into the asset, with its threat, and walks the paths from there. A finding you set aside is no longer an opening. See [How OpenCRQ quantifies risk](/opencrq/concepts/how-risk-is-quantified/).

The other tabs explain the score's inputs:

- **Attack** lists the **Threats citing this CVE**, with a link to each one in OpenCTI, and the ATT&CK techniques that exploit it. Each threat's badge says how long ago it was last observed: red under 14 days, yellow under 90 days, blue beyond, and **Not dated** when no source dated it. Only a threat observed in the last 90 days counts as active.
- **Controls** lists the **Controls standing on this finding**.
- **Remediation** shows what the vendor published. When the CVE is in the CISA KEV catalog, a banner gives the action CISA requires and its deadline, as CISA published them.
- **Activity** shows the finding's state over time and its sources; **Events** the history of its score.

## Attach a CVE by hand

When you know of a vulnerability no integration reports:

:::steps
1. Open the asset from :menu[Attack surface > Assets], then its **Findings** tab.
2. Select **Create**. The **Attach a CVE** panel opens.
3. In **CVE ID**, type at least three characters of the identifier and pick the CVE from the list. Only CVEs the tenant already knows are listed, from its CVE sources or from the threats OpenCTI streams, and not those already on the asset.
4. Select **Create**.
:::

The finding is marked **Set by user** as its source. Only a finding attached by hand can be deleted, from its page with **Delete**: one an integration reports comes back at its next synchronization.

## Set a finding aside

A finding you accept or that is not true should not count in your risk. Setting it aside takes it out of the counters, the unique findings, the asset's posture and the attack run, and keeps the decision on record.

:::steps
1. Open the finding and select **Set aside**.
2. Choose the **Type**:
   - **Risk accepted**: the flaw is real and you live with it. Its **Reason** is one of **Won't fix**, **Patch unavailable**, **Downtime not permitted**, **Application incompatibility**, **Compensatory control applied**, **Technical debt or end of life**, **Not externally exposed**, **Business exception**, **Vendor dependency** or **Other**.
   - **False positive**: the flaw is not true here, and the integration needs fixing. Its **Reason** is **Detection error**, **Not applicable** or **Other**.
3. Write the **Rationale**, up to 2,000 characters, for whoever reviews the decision next.
4. Select **Set aside**. The finding leaves the default list; choose **All states** to find it again.
:::

Your decision stands over the integrations' reports. To undo it, select **Mark applicable**. When an integration still reports the finding, **Resync with connector** hands the state back to it: its next synchronization decides again.

An integration sets a finding aside on its own when it no longer reports it (**No longer reported**), or when a test finds the technique cannot reach the asset (**Not exposed**). There is no "remediated" state: a fixed vulnerability is one no integration reports any more.

:::example[CVE-2024-3400 on the head-office VPN gateway]
OpenCTI streams Ember Tide as a threat to Grand Meridian: it targets CVE-2024-3400, the flaw it enters hotel chains through. The risk manager of Grand Meridian France attaches it to the Head-office VPN gateway, which is internet-facing and **High** in criticality: on the gateway's **Findings** tab, **Create**, then **CVE ID** CVE-2024-3400.

On 12 April the finding is **Applicable**, with **Set by user** as its source, but it has no score: the tenant's CVE sources have not given CVE-2024-3400 a CVSS vector yet (**CVSS base** reads **Not scored**, **EPSS** **Unknown**, **CISA KEV** **Not listed**), so it counts under **Pending vector**. Its **Attack** tab lists the two **Threats citing this CVE**: Ember Tide, last observed on 8 April through Operation Low Season, in red, and Velvet Harbor, last observed on 9 January, more than 90 days ago, in blue. Ember Tide is active, so its citation will weigh in the threat factor, but only once the finding has a score. Without a vector the attack run has no opening to read for it either, and the scenario prices of 12 April did not move. Once the NIST source scores the CVE, the finding gets its score and becomes a door into the gateway, until the risk manager deletes it once the gateway is patched, or sets it aside with a **Rationale** the auditors can read.
:::

## Check that it works

- In :menu[Attack surface > Findings], the counters and the list show the findings of every integration, with **Set by user** for the ones you attached.
- A finding you set aside shows under **Set aside, no longer reported or not exposed**, and the decision in the **History** of its **Activity** tab.
- The asset's **Events** tab shows the change of its **OpenCRQ Posture**.

## What's next

- [Set how strong your controls are and read how they held](/opencrq/use/defenses/controls/): the technique findings measure them.
- [Follow the threats that target your scopes](/opencrq/use/threats/threat-center/), whose citations raise a finding's score.
- [Import OpenAEV results](/opencrq/use/sources/openaev/).
