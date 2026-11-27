---
title: Track the fixes that close your exposures
description: Follow each fix that closes the open vulnerabilities you see in OpenCTI and OpenAEV as an XTM One case, from its owner and due date to its evidence or accepted risk.
doc_type: how-to
products: [xtm-one, opencti, openaev]
personas: [soc-manager, soc-analyst, executive]
owner: docs-xtm-one
last_reviewed: 2026-11-16
---

In continuous threat exposure management (CTEM), a case is one fix and the open issues it closes. XTM One groups the issues you can see in OpenCTI and OpenAEV into cases. You then follow each fix from **Needs attention** to **In progress**, then to **Resolved** or **Accepted risk**. Cases are shared across your organization, and each one keeps the history of who changed what.

::diagram{id=xtm-one-ctem-cases}

## How cases are built

XTM One reads the open issues with your own access, through the OpenCTI and OpenAEV integrations of your organization:

- From OpenCTI, the vulnerabilities that affect a software of the **Tech stack** in your organization profile. A software matches by its Common Platform Enumeration (CPE) name, or by its product and vendor. Without a tech stack, OpenCTI adds nothing.
- From OpenAEV, the common vulnerabilities and exposures (CVE) findings created in the last 90 days.

It then groups the issues by the fix that closes them:

- A course of action that remediates or mitigates a vulnerability gives a case named after it.
- Otherwise, the affected product gives an upgrade case, such as "Upgrade Example Networks Example Edge Gateway to a fixed version". When the fixed version is known, the name gives it.
- An OpenAEV finding joins the OpenCTI case of the same CVE.
- An issue with no known fix opens no case.

Cases cover cyber exposures only. You see a case when you can see at least one of its issues in OpenCTI or OpenAEV. Its other issues count as restricted, without their details. Anyone who sees a case can change it.

The priority comes from the issues you can see:

- **Critical**: an issue is in the Known Exploited Vulnerabilities (KEV) catalog, or has a Common Vulnerability Scoring System (CVSS) score of 9 or more.
- **High**: a CVSS score of 7 or more, or an Exploit Prediction Scoring System (EPSS) score of 0.5 or more.
- **Medium**: a CVSS score of 4 or more, or an EPSS score of 0.1 or more.
- **Low**: the other cases.

**Recompute**, at the top right of the list, reads the issues again with your access. It adds the new issues and reopens the cases whose issues are open again. It marks an issue closed only when your read shows it closed at the source. It never changes a state, an owner, a due date or a note. If OpenCTI or OpenAEV does not answer, the message names it and no case changes. You can recompute once per minute.

When every issue of a case is closed, **System** resolves the case, unless its risk is accepted. A resolved case whose issues open again returns to **Needs attention**. Each day, the accepted risks that have expired also return to **Needs attention**.

## Before you start

- [Connect OpenCTI or OpenAEV](/xtm-one/build/integrations/) to XTM One, with your own access to it: XTM One reads the issues as you.
- For OpenCTI vulnerabilities, fill in the **Tech stack** of your [organization profile](/xtm-one/administer/organization-profile/). Without a primary organization, the list stays empty.
- Cases call no model: they work with AI switched off, in every edition. Only the agents that update cases need the Enterprise Edition.

## Open the cases

:::steps
1. On the home page, click **Open Cases** in the **Cases** widget. **View all cases**, on the **Remediation** stage of the [dashboard](/xtm-one/use/dashboard/#follow-the-funnel-and-the-cases), opens the same list.
2. The list opens on **Needs attention**. The other tabs are **In progress**, **Resolved**, **Accepted risk** and **All**, each with its count.
3. Type in **Search fix, product or version** to find a fix, or narrow the rows with the **Priority**, **Owner**, **Due** and **Crown jewels** filters.

::screenshot{id=xtm-one-ctem-cases-list alt="The Cases list of Hélène Marchetti on the Needs attention tab: one Critical case, Upgrade Example Networks Example Edge Gateway to a fixed version, Unassigned; Recompute at the top right."}
:::

Each row gives the **Priority**, the **Fix** and its **Issues**, with those restricted to you. It also gives the **Crown jewels** it touches, whether an issue is in **KEV**, the **Owner**, the **Due** date and the **State**. The rows are sorted by priority, 25 per page, and you can export them as CSV.

In the **Due** filter, **Overdue** keeps the open cases past their due date. **Accepted risk expiring** keeps the accepted risks that expire within 14 days. In the **Owner** filter, **Unassigned** finds the cases nobody holds yet.

When no issue of your stack is open, the list says **No open exposure on your stack**. If OpenCTI or OpenAEV could not be read, the list says so, because open exposures there may be missing.

## Take a case in hand

In the list, select one or more cases, then click an action:

- **Start** moves the cases to **In progress**. You become the owner of those without one; the others keep theirs.
- **Assign owner** gives the cases to a person: type two letters of their name in **Owner**. The owner must see at least one issue of each case.
- **Set due date** sets the **Due date**, 30 days ahead by default.

Each action opens a dialog for the selected cases, confirmed by the button of the same name. XTM One then tells you how many cases it updated. It lists each refused case with the rule it breaks: for example, a case moves to **In progress** only with an owner.

On the case page, **Start** does the same for that case: you become its owner if it has none.

## Resolve or reopen a case

:::steps
1. Open the case from the list.
2. Click **Resolve**.
3. In **Evidence**, give what closed the exposure: a ticket, a change or a validation run.
4. Confirm. The case moves to **Resolved**.
:::

A case is resolved only with evidence. **Reopen**, on a resolved case or an accepted risk, asks for a **Reason** and returns the case to **Needs attention**.

## Accept the risk

When you decide not to fix an exposure for now, accept its risk for a time. Only a person accepts a risk, never an agent.

:::steps
1. On the case page, click **Accept risk**. In the list, select the cases, then click **Accept risk**.
2. Enter the **Reason**, up to 500 characters.
3. In **Expires on**, choose a day at most 90 days away. The case returns to **Needs attention** on that day.
4. In the list, also choose the **Owner**. On the case page, the case keeps its owner, or you become its owner.
5. Confirm. The case moves to **Accepted risk**, and its page shows the reason and the expiry date.
:::

If the dialog answers that the expiry date is at most 90 days away, choose an earlier day.

## Split or merge cases

Split a case when some of its issues need another fix:

:::steps
1. On the case page, select the open issues in the **Issues** table.
2. Click **Split into a new case**.
3. Enter the **Name of the new case**, then confirm.
:::

The new case opens. It starts in **Needs attention** and keeps these issues through every recompute. A closed issue stays with its case.

To merge cases that share one fix, select two cases or more in the list, then click **Merge**. Choose the **Case to keep**: it keeps its state, owner and history, and takes the issues and history of the others. The other cases disappear, and later recomputes add their issues to the case you kept.

## Read a case

The case page shows the fix as its title, with its state, priority and owner. Its cards count the **Open issues** and the **KEV** ones, the **Max CVSS** of the issues you can see, and the **Assets** with their crown jewels.

The **Issues** table gives each issue with its **Source**, **CVSS**, **EPSS**, **KEV**, **Assets** and **State**. Type in **Search CVE** to find one, and filter the **Source** on OpenCTI, OpenAEV or the restricted issues.

The **History** lists every change with its actor: **System** for recomputes, a person, or **Agent**. It records the creation, the issues added and closed, the changes of state, owner and due date, the splits and the merges. It also shows the notes that agents add.

If a link opens **Case not found**, the case does not exist, or none of its issues is visible to you in OpenCTI or OpenAEV.

## Let agents update cases

This requires the Enterprise Edition with AI switched on. The built-in agents **Remediation Coordinator** and **CTEM Mobilization Analyst** can work on cases from the [chat](/xtm-one/use/chat/) or in their runs. They list and read cases, split open issues into a new case, and change the owner, the due date, a note or the state.

An agent acts as the person of the conversation or the run, and sees only the cases that person sees. It never accepts a risk, and it resolves a case only with evidence. The **History** records each of its changes as **Agent**.

:::example[Grand Meridian closes an exploited gateway flaw]
On Monday 13 April 2026, Hélène Marchetti, the CISO of Grand Meridian, opens the cases. **Needs attention** holds "Upgrade Example Networks Example Edge Gateway to a fixed version", **Critical**: its CVE-2024-3400 is in KEV with a CVSS score of 10.

She selects it, assigns it to Julien Bernard of the security operations center (SOC), and sets the due date to 20 April. Julien opens the case and clicks **Start**. On 17 April, he resolves it with the evidence "Change CHG-2026-0414: every Example Edge Gateway upgraded; OpenAEV validation run passed".
:::

## What's next

- [Follow the funnel and the cases](/xtm-one/use/dashboard/#follow-the-funnel-and-the-cases) on the dashboard.
- [Set up the organization profile agents read](/xtm-one/administer/organization-profile/), its tech stack included.
- [Ask the agents](/xtm-one/use/chat/) to prepare the fixes.
- [Review what the agents did](/xtm-one/use/work-history/).
