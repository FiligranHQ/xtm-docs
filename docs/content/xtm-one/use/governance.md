---
title: Follow your exposure posture against its target
description: Read the Governance & Risk page of XTM One to see how far the exposure posture stands from the target your organization set, which accepted risks expire soon, and set that target.
doc_type: how-to
products: [xtm-one, opencrq]
personas: [executive, risk-manager, admin]
owner: docs-xtm-one
last_reviewed: 2026-11-16
---

The **Governance & Risk** page of XTM One answers two questions a CISO gets from the board: are we where we said we would be, and which risks did we accept? It draws your exposure posture over the last 90 days against the posture target your organization set, and shades the gap. It places the accepted risks of the prioritization policy by the days left before they expire, the expired ones in red. The page reads the posture history and the organization profile only, and never calls a model. The posture itself is explained in [Read the home dashboard](/xtm-one/use/dashboard/#read-the-exposure-posture).

::screenshot{id=xtm-one-governance-posture-target alt="The Governance & Risk page of Hélène Marchetti: the header with the exposure posture and its band, the posture target of 75 and the gap to it, the Posture against target chart on the five band zones with the dashed target line, then the Accepted risks strip with three risks of Grand Meridian Hotels, one expired in red, and the SLA adherence card marked Soon."}

## Before you start

- XTM One runs as the XTM suite's command center, with OpenCTI or OpenAEV connected: the posture comes from them. See [Read the home dashboard](/xtm-one/use/dashboard/#what-you-see-depends-on-the-platform).
- Every user opens the page. Only an administrator sets the posture target and edits the accepted risks.
- The page reads the vertical you chose on the dashboard: **Cyber**, **FIMI** or **Fraud**.

## Open the page

On the home dashboard, the last column of **Exposure lifecycle** is **Governance & Risk**: click it, or its **Open Governance & Risk**. The **Governance & Risk** card, which **Add a widget** offers, has the same button. **Back to Command Center**, at the top of the page, returns to the dashboard.

The header gives two figures:

- **Exposure posture**: the measured score out of 100, and its band, the same as on the dashboard.
- **Posture target**: the score the organization aims for, or `--` while none is set.

Next to **Back to Command Center**, a badge says where the posture stands: "12 points below the target of 75" in red, or "At or above the target of 75" in green. The info button next to the title, **About Governance & Risk**, says what the page shows.

## Read the posture against its target

**Posture against target** draws one point per day over the last 90 days: your exposure posture, as the dashboard computes it for you. One score is kept per day, the last one computed that day. Hover a point to read its day and score.

Behind the line, five zones give the bands of the score:

| Zone | Scores |
| --- | --- |
| **Critical** | Under 40 |
| **At risk** | From 40 to under 60 |
| **Fair** | From 60 to under 75 |
| **Good** | From 75 to under 90 |
| **Strong** | 90 and above |

The target is a dashed line, labelled "Target 75" for a target of 75. Where the posture is under it, the gap between the two is shaded in red: the wider the band, the further you are from the commitment.

Before any day is recorded, the card says "No posture recorded yet: the first day appears after the next sync". With a single day, it shows the point and "Trend after 2 days of history". While no target is set, the card says so instead of the target line: see [Set the posture target](#set-the-posture-target).

## Check the accepted risks

**Accepted risks** reads the accepted risks of the organization's prioritization policy, each with its subject, its decision owner and its expiry date. Each risk is a dot on a strip that runs from **Today** to 90 days:

- red: expired;
- orange: expires within 30 days;
- the accent color: expires later. A risk that expires beyond 90 days sits at the end of the strip.

Hover a dot to read the risk and its days. Under the strip, the five risks that expire first are listed, the expired ones at the top, such as "Expired 7 days ago", "Expires today" or "Expires in 18 days". A risk with no subject is left out.

An expired risk is a decision to review with its owner. To change a risk, an administrator edits **Accepted risks** in the **Prioritization policy** card: see [Set up the organization profile agents read](/xtm-one/administer/organization-profile/#set-the-prioritization-policy).

With no accepted risk, the card says "No accepted risk with an expiry in the prioritization policy". An administrator gets **Open the prioritization policy**, which opens :menu[Administration > Settings > Organization].

## Read the SLA adherence

**SLA adherence** is marked **Soon**: it will measure the remediation service level agreements (SLAs) from OpenCRQ. Without OpenCRQ, the card says "SLA adherence needs OpenCRQ". Once an OpenCRQ platform is registered, it says "OpenCRQ is connected: SLA adherence arrives with its remediation SLAs", with a button that opens OpenCRQ.

## Set the posture target

The posture target is the score the organization commits to, such as 75 for a posture in the **Good** band. An administrator sets it on this page; it is saved in the organization profile.

:::steps
1. Open the **Governance & Risk** page.
2. In **Posture against target**, click **Set a posture target**, or **Change the target** once one is set.
3. In **Posture target**, type a whole number from 0 to 100. Any other value shows "Enter a whole number from 0 to 100", and **Save** stays greyed out.
4. Click **Save**. **Cancel** leaves the target as it was.
:::

"Posture target saved" confirms it. The target line and the gap show at once, and the dashboard's **Governance & Risk** column gives the target too. Each save writes a new version of the organization profile. The target is not part of the organization summary agents receive.

Anyone else reads "No posture target yet: an administrator sets it" until one is set.

:::example[Hélène commits Grand Meridian to a Good posture]
On Sunday 12 April 2026, Hélène Marchetti, CISO of Grand Meridian Hotels, prepares the executive committee of Monday. The committee approved a posture in the **Good** band by the summer.

1. On the home dashboard, she clicks **Open Governance & Risk** in the **Governance & Risk** column.
2. She clicks **Set a posture target**, types `75` in **Posture target**, and clicks **Save**.
3. In **Posture against target**, the dashed "Target 75" line sits on the lower edge of the **Good** zone. The red band under it is the gap she presents to the committee.
4. In **Accepted risks**, she reads three risks:

   | Accepted risk | Decision owner | What the page says |
   | --- | --- | --- |
   | `CVE-2023-4966` on the Madrid VPN gateway | Julien Bernard | "Expired 7 days ago", in red |
   | `CVE-2024-3094` on the legacy build server | Marc Dupuis | "Expires in 18 days", in orange |
   | Front-desk printer firmware 2.1 in Paris | Hélène Marchetti | "Expires in 79 days" |

She asks Julien Bernard to renew the Madrid decision or to patch the gateway before Monday. The committee then sees one commitment, one gap, and every accepted risk with its owner.
:::

## What's next

- [Read the home dashboard](/xtm-one/use/dashboard/), where the **Governance & Risk** column opens this page.
- [Set up the organization profile agents read](/xtm-one/administer/organization-profile/), to keep the accepted risks of the prioritization policy up to date.
- [XTM One in OpenCRQ](/xtm-one/use/in-opencrq/), for the risk side of exposure.
