---
title: Track the organizations your company depends on
description: List the providers, customers, partners and subsidiaries XTM One agents should know about, link each one to its organization in OpenCTI, and draft a long list with AI from names or a supplier file.
doc_type: how-to
products: [xtm-one]
personas: [admin]
owner: docs-xtm-one
last_reviewed: 2026-11-16
---

Your organization's profile tells agents who you are. Tracked organizations tell them who you depend on and who you serve: the providers whose breach can reach you, the customers whose breach is your exposure, your partners and your subsidiaries. This page adds them one by one or drafts a whole list with AI, links each one to its organization in OpenCTI, and says where agents read them.

::screenshot{id=xtm-one-administer-tracked-organizations alt="The Tracked organizations card of Settings > Organization, with Draft with AI and Add an organization at the top right, and four organizations of Grand Meridian Hotels: Tidewell Payments, Lanternway Hotel Cloud, Grand Meridian Iberia and Example CERT, each with its relationship, criticality, OpenCTI link and Monitoring off."}

## Before you start

- Your account is an administrator. Only administrators open Settings, and only they add, change or archive a tracked organization.
- To link an organization to OpenCTI, the platform has a company-managed OpenCTI integration, and it gives you a token: the search runs on that OpenCTI, as you. See [Connect agents to your tools](/xtm-one/build/integrations/).
- **Draft with AI** needs the Enterprise Edition and AI switched on. Otherwise, the button shows a lock and opens no draft. Each run spends your agentic quota. See [What each edition of XTM One brings](/xtm-one/administer/editions/).

## Add an organization

:::steps
1. Open :menu[Administration > Settings > Organization]. The **Tracked organizations** card is at the bottom of **Profile**.
2. Click **Add an organization**.
3. In **Name**, type the name agents will use for it in briefs and reports.
4. In **Relationship**, pick what it is to your organization: **Provider** (it supplies a product or a service you depend on), **Customer**, **Partner** or **Subsidiary**. Agents word their briefs from it: a provider's breach can reach you, a customer's breach is your exposure.
5. In **Criticality**, pick what a breach there would do to you, from **Low** (it barely affects you) to **Critical** (it stops your core business). **Medium** is the default.
6. Optionally, in **OpenCTI organization**, search the connected OpenCTI by name or alias and pick its organization. Threats and reports about it are then read from there. Left empty, nothing is read from OpenCTI for it.
7. Click **Create**.
:::

The organization joins the list with **Monitoring off**. Search the list, and filter it by **Relationship**, **Criticality** and **OpenCTI**: **Linked to OpenCTI** or **Not linked to OpenCTI**. **Columns** hides the columns you do not need, or brings back the ones a narrow window set aside. Your browser keeps your choice for your next visit.

Monitoring is off for every tracked organization: nothing is watched in OpenCTI for it, and no run is counted. The platform offers no way to switch it on.

## Draft a list with AI

To add many organizations at once, for example from a supplier export, let a model propose their details. Nothing is created until you accept the proposals.

:::steps
1. On the **Tracked organizations** card, click **Draft with AI**.
2. In **Organization names**, type one name per line. A name already tracked costs no request: it shows as a conflict on the review page.
3. Or, in **Supplier or customer file**, add one CSV or XLSX file of 2 MB at most. Only its first column is read, one name per row. A first row that reads Name, Names, Organization, Organisation, Company, Supplier, Customer or Vendor is skipped as a header; any other first row is read as a name. You can type names and add a file in the same run.
4. In **Relationship when unsure**, pick the relationship for the names the model cannot place. **Provider** is the default.
5. Check the line under the fields: the number of names, and the agentic requests of your quota they take.
6. Click **Draft**. The review page opens.
:::

A run drafts the first 50 names: run it again for the next ones. It takes one agentic request of your quota per 10 names not yet tracked. When your quota is spent, the dialog says "Your agentic quota is spent." and links to :menu[Administration > Settings > Usage and quotas].

For each name, the model reads your organization's profile and proposes a sector, regions, a relationship and a criticality, with how sure it is and why. XTM One also searches the connected OpenCTI for organizations of that name or alias, and picks the match when exactly one has that very name.

## Review the draft

The review page, **Review drafted organizations**, lists one row per name, with its review state, its **Match confidence** (**High confidence**, **Medium confidence** or **Low confidence**), its sector, relationship, criticality and **OpenCTI organization**. Hover a match confidence to read why the model proposed it. A narrow window hides the sector first, then the confidence, relationship and criticality: **Columns** brings them back, or hides the ones you do not need, and your browser keeps your choice. Search the rows by name or sector, and filter them by **Relationship**, **Criticality** and **Match confidence**.

::screenshot{id=xtm-one-administer-tracked-organizations-review alt="The Review drafted organizations page of a supplier export: Corvane Guest Wi-Fi and Saltmarsh CERT with High confidence, Harbourline Linen Services with Low confidence, and Tidewell Payments marked Already tracked, with Discard draft and Create accepted (0) at the top right."}

:::steps
1. Check each row's **OpenCTI organization**. Pick another match, or clear it. A row with **No match** is created without a link.
2. Click the accept or reject icon of each row, or select rows and click **Accept selected** or **Reject selected**.
3. Click **Create accepted**. The button shows how many rows you accepted.
:::

Only the accepted rows are created, each one with **Monitoring off**, and you are back on :menu[Administration > Settings > Organization]. Its sector and regions become its profile. The rows still to review are rejected, and the draft is closed: nothing more can be created from it. **Discard draft** closes it without creating anything.

The review page has its own address, `/settings/organization/tracked/ai-draft/` followed by the draft's id: keep it to come back to a draft you have not decided yet, or to send it to another administrator. A decided draft still opens, read-only, with each row marked **Accepted** or **Rejected** and the line "This draft was already decided; nothing more can be created from it."

Some rows may show **Already tracked**, with the line "See it in Tracked organizations", or **Not drafted**. Neither can be selected. Click an **Already tracked** row to go back to :menu[Administration > Settings > Organization], where the organization is listed. When the model failed on some names, a line says how many, **Details** says why, and **Retry** opens the dialog again with those names, for a new run. When OpenCTI could not be searched, every row shows **No match**: check the OpenCTI integration and your token, then draft again.

## Change or archive an organization

In the list, click the edit icon of a row to change its name, relationship, criticality or OpenCTI organization, then click **Save**.

Click the archive icon to archive it. It leaves the list, and agents no longer read it. Its profile versions are kept.

## How agents read them

On a fleet's **Members** tab, **Organizations** sets the organizations each member reasons about: as a fleet default, or for one member. Your own organization is always in. Add the tracked organizations the fleet works for, or switch on **All monitored organizations**: every tracked organization with monitoring on. Before you apply, a line estimates the runs a month they add to the fleet. See [Run agents as a team with fleets](/xtm-one/build/fleets/).

A member's run then receives the profile of each tracked organization in its scope, after your organization's own.

:::example[Grand Meridian lists the providers behind its hotels]
Hélène Marchetti, CISO of Grand Meridian Hotels, adds the organizations a breach would reach the hotels through. Tidewell Payments takes the card payments of every hotel: **Provider**, **High**. Lanternway Hotel Cloud runs the property management system of every hotel: **Provider**, **Critical**. Grand Meridian Iberia runs the hotels of Madrid and Barcelona: **Subsidiary**, **High**. The national CERT the SOC shares indicators with is a **Partner**, linked to its organization in OpenCTI.

The purchasing team then sends her its supplier export. She clicks **Draft with AI** and types its names in **Organization names**: Corvane Guest Wi-Fi, Saltmarsh CERT, Harbourline Linen Services and Tidewell Payments, with **Provider** in **Relationship when unsure**. The line under the fields reads "4 names, 1 agentic request of your quota". Tidewell Payments is already tracked: it costs no request, and the review page shows it as **Already tracked**.

| Row | Proposal | Her decision |
| --- | --- | --- |
| Corvane Guest Wi-Fi | **Provider**, **High**, **High confidence** | Accept |
| Saltmarsh CERT | **Partner**, **Medium**, linked to its organization in OpenCTI | Accept |
| Harbourline Linen Services | **Provider**, **Low**, **Low confidence**: the name alone does not say which company it is | Leave it to review |

She clicks **Create accepted (2)**. Corvane Guest Wi-Fi and Saltmarsh CERT join the list with **Monitoring off**, and Harbourline Linen Services is rejected with the draft. She adds it by hand once purchasing confirms who it is.
:::

## What's next

- [Run agents as a team with fleets](/xtm-one/build/fleets/)
- [Set up the platform in Settings](/xtm-one/administer/settings/)
- [What each edition of XTM One brings](/xtm-one/administer/editions/)
