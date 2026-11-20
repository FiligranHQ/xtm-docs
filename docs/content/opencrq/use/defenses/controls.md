---
title: Set how strong your controls are and read how they held
description: Declare how strong and how widely deployed each of your controls is, per risk scope, and read how OpenAEV's tests and your compliance answers confirm or correct it.
sidebar_position: 1
doc_type: how-to
products: [opencrq, openaev]
personas: [risk-manager, validation-engineer]
owner: docs-opencrq
last_reviewed: 2026-11-04
---

Every attack OpenCRQ simulates meets your controls on its way to your primary assets: your email security, your EDR, your identity platform, your backups. How far it gets depends on what you declare for each control: its **strength** and its **coverage**. This page declares them, for the whole tenant or for one risk scope, and reads how they held when OpenAEV tested them. A control you never declare counts as not deployed, so the price of a scenario is only as good as these declarations. For the model behind the price, read [How OpenCRQ quantifies risk](/opencrq/concepts/how-risk-is-quantified/).

## Before you start

- You are a member of the tenant: every member declares control postures.
- The controls are those of OpenCRQ's catalogue, 51 of them, each covering the ATT&CK techniques it prevents or detects. See the [control catalogue](/reference/opencrq/controls/).
- To read how the controls held, an administrator with **Manage Security** connects OpenAEV. See [Import OpenAEV results](/opencrq/use/sources/openaev/).

## What you declare

Each control has two axes. You answer each with a level, from L0 to L3.

| Axis | The question | Levels |
| --- | --- | --- |
| **Strength** | How is the control implemented? | **Not deployed**, **Basic**, **Intermediate**, **Master**. Each level includes the ones below; **What each strength rung demands** gives the criteria of each level for that control |
| **Coverage** | How widely is it deployed? | **None**, **Partial**, **Broad**, **Full**: the share of the perimeter the control concerns, not how well it works there |

You do not declare whether a control works when an attack comes: OpenAEV's tests answer that, technique by technique, and lower what you declared where they show less (see [Read how a control held](#read-how-a-control-held)).

A control not declared anywhere has no strength and no coverage: the attack run prices it as absent.

The **Effectiveness** of a control sums the two axes in one figure: the strength, times the coverage (**Partial** counts 0.4, **Broad** 0.7, **Full** 1). It reads **Marginal** up to 30%, **Moderate** up to 55%, **Solid** up to 80%, **Strong** above; **Not deployed** at **Not deployed** strength, **Inactive** when the coverage is **None**. It helps compare controls at a glance; the attack run reads the two axes themselves, technique by technique, as audited by the tests.

## Declare a control for a risk scope

The tenant's declaration applies to every risk scope; a scope can declare its own, axis by axis, when its estate differs.

:::steps
1. Open the risk scope in the sidebar, then **Controls**. The list groups the 51 controls by domain (**Endpoints and servers**, **Network**, **Identities**, **Email and SaaS**...) with their **Strength**, **Coverage**, **Effectiveness** and **Last declared** date, and counts the controls not assessed.

   ::screenshot{id=opencrq-controls alt="The controls of Grand Meridian France grouped by domain, 45 of 51 not assessed: EDR Intermediate and Broad at 59% Solid, Patch Management Basic and Broad at 39% Moderate, both declared on January 12, 2026."}

2. Select a control. Its **Overview** gives its definition, the asset families it protects, its **Assessment** and the effectiveness over time.
3. Select **Edit assessment**.
4. For each axis, select a level. Each axis shows the tenant's value beside it; **Revert to the tenant value** drops the scope's own declaration of that axis.
5. Write a **Comment**: what is deployed, where, since when. It stays in the control's **Events**.
6. Check the **Effectiveness** line at the bottom, which shows the value before and after your change, then select **Save**.

   ::screenshot{id=opencrq-controls-assessment alt="The assessment of the identity platform for Grand Meridian France: strength Intermediate, coverage Full, effectiveness Strong at 84%."}
:::

To declare for the whole tenant, open :menu[Defenses > Controls] instead. There, each axis also offers **Auto**, which takes the value from the control's sources: the published compliance assessments and Drata. **Auto** is only available when a source feeds the axis, and shows what it reads today. Switching an axis between **Auto** and a value you choose asks for a **Justification**.

## Read how a control held

Declarations are what you believe. OpenAEV's simulations show what happened when an attack came.

- **Validation**, on a control that a security platform family can test, appears once OpenAEV is connected. Each line is a technique a simulation played: its **Scenario**, the **Platform family** that answered, the **Function** tested, the **Result** (**Prevented** or **Not prevented**, **Detected** or **Not detected**) and the **Tested assets**. Filter the lines by technique, function, result (**Held on every asset** or **Gave way on an asset**) and platform family. **Success rate** and **Proof coverage** follow over time; **Open in OpenAEV** opens the simulation.
- **Prevention** and **Detection** list the techniques the control covers, as a matrix or a table. Each technique shows its **OpenAEV result** and its **Engine efficacy**, the figure the attack run reads: on **Prevention**, the chance the control stops the technique when an attacker uses it; on **Detection**, how strongly it signals it. To read its calculation, select the information icon beside the figure in the table, or the technique in the matrix. Filter the techniques by **Confirmed by the tests**, **Lowered by the tests**, **Not tested** and **Tested, not covered**.
- **Risk Scenarios** lists the scenarios whose techniques the control covers, with the number of **Shared TTP**.
- **Events** records every change of the posture, its author and its comment.

The attack run reads a control technique by technique. On a technique OpenAEV has not tested, the efficacy is the declaration: what your strength and your coverage give on that technique. On a tested technique, the tests audit the declaration, they never raise it:

> Engine efficacy = declaration − tested share × (declaration − rate the tests held)

The tested share is the part of the risk scope's hosts the tests ran on, and the term in brackets counts only when the tests held less often than you declared. A test cannot create coverage nobody declared, so a control declared absent stays absent, and a test that does better than the declaration leaves it as it is. The test results count for as long as the **Validity of a test result** set in the [quantification engine](/opencrq/configure/quantification-engine/) allows.

OpenAEV may test a control on a technique the catalogue does not credit it with. Its **Engine efficacy** reads **Not declared** in the table, and **Tested, not declared** in the matrix; the **Tested, not covered** filter lists it, and the attack run leaves its tests out. If your deployment does act on it, select **Link** at the end of its row, or **Declare on** the control at the bottom of its calculation: the technique joins the control for your tenant, and its tests count. **Link a technique** adds any other technique, **Unlink** takes one out, and **Reset prevention links** or **Reset detection links** discards your changes and goes back to the catalogue. The **Assessment evidence** of a control lists the compliance answers that feed it; see [Measure your compliance maturity against a framework](/opencrq/use/defenses/compliance/).

:::example[MFA for the front-desk accounts]
On 12 January 2026, Grand Meridian France declares its identity platform (IDP) as it is: single sign-on, MFA for the head-office staff only. **Strength** **Basic**, **Coverage** **Partial**, with the **Comment** "Single sign-on; MFA for head-office staff only, not the front-desk accounts".

The treatment plan MFA for the front-desk accounts is completed on 27 February. On 2 March, the risk manager opens **Controls** under Grand Meridian France in the sidebar, selects the identity platform, then **Edit assessment**: **Strength** **Intermediate**, **Coverage** **Full**, and the **Comment** "MFA on every account, the front desks included, since 2026-02-27".

The control now reads 84%, **Strong**. Its **Events** tab shows both declarations, from **Basic** to **Intermediate** and from **Partial** to **Full**, and its **Risk Scenarios** tab the two French scenarios it stands on: Data Breach on Guest Records and Ransomware on the Booking Platform. OpenCRQ prices them again on the new posture, and their annual loss falls.
:::

## Check that it works

- The scope's **Controls** list shows the new levels and a **Last declared** date of today.
- **Overrides on this scope**, in **Filter by assessment status**, lists the controls the scope declares itself.
- The scenarios the control stands on are priced again: their **CRQ** tab dates the run.

## What's next

- [Choose the treatment plan that removes the most loss](/opencrq/use/decide/treatment-plans/): a plan sets target levels for controls and prices the difference.
- [Measure your compliance maturity against a framework](/opencrq/use/defenses/compliance/), whose published answers feed the controls.
- [Import OpenAEV results](/opencrq/use/sources/openaev/).
